# 盘点Linux Epoll那些致命弱点

## 内容目录

本文来自 Marek’s 博客中 I/O multiplexing part 系列之三和四，原文一共有四篇，主要讲 Linux 上 IO 多路复用的一些问题，本文加入了我的一些个人理解，如有不对之处敬请指出。原文链接如下：

The history of the Select(2) syscall [1]

Select(2) is fundamentally broken [2]

Epoll is fundamentally broken 1/2 [3]

Epoll is fundamentally broken 2/2 [4]

## 2 脉络

系列三和系列四分别讲 epoll(2) 存在的两个不同的问题：

1. 系列三主要讲 epoll 的多线程扩展性的问题
2. 系列四主要讲 epoll 所注册的 fd (file descriptor) 和实际内核中控制的结构 file description 拥有不同的生命周期

我们在此也按照该顺序进行阐述。

### 3 epoll 多线程扩展性

epoll 的多线程扩展性的问题主要体现在做多核之间负载均衡上，有两个典型的场景：

1. 一个 TCP 服务器，对同一个 listen fd 在多个 CPU 上调用 `accept(2)` 系统调用
2. 大量 TCP 连接调用 `read(2)` 系统调用上

### 3.1 特定 TCP listen fd 的 accept(2) 的问题

一个典型的场景是一个需要处理大量短连接的 HTTP 1.0 服务器，由于需要 accept() 大量的 TCP 建连请求，所以希望把这些 accept() 分发到不同的 CPU 上来处理，以充分利用多 CPU 的能力。

这在实际生产环境是存在的， Tom Herbert 报告有应用需要处理每秒 4 万个建连请求；当有这么多请求的时候，很显然，将其分散到不同的 CPU 上是合理的。

然后实际上，事情并没有这么简单，直到 Linux 4.5 内核，都无法通过 epoll(2) 把这些请求水平扩展到其他 CPU 上。下面我们来看看 epoll 的两种模式 LT(level trigger, 水平触发) 和 ET(edge trigger, 边缘触发) 在处理这种情况下的问题。

#### 3.1.1 水平触发的问题：不必要的唤醒

一个愚蠢的做法是是将同一个 epoll fd 放到不同的线程上来 epoll_wait()，这样做显然行不通，同样，将同一个用于 accept 的 fd 加到不同的线程中的 epoll fd 中也行不通。

这是因为 epoll 的水平触发模式和 `select(2)` 一样存在 “惊群效应”，在不加特殊标志的水平触发模式下，当一个新建连接请求过来时，所有的 worker 线程都都会被唤醒，下面是一个这种 case 的例子：

```shell
1. 内核：收到一个新建连接的请求
2. 内核：由于 "惊群效应" ，唤醒两个正在 epoll_wait() 的线程 A 和线程 B
3. 线程A：epoll_wait() 返回
4. 线程B：epoll_wait() 返回
5. 线程A：执行 accept() 并且成功
6. 线程B：执行 accept() 失败，accept() 返回 EAGAIN
```

其中，线程 B 的唤醒完全没有必要，仅仅只是浪费宝贵的 CPU 资源而已，水平触发模式的 epoll 的扩展性很差。

#### 3.1.2 边缘触发的问题：不必要的唤醒以及饥饿

既然水平触发模式不行，那是不是边缘触发模式会更好呢？实际上并没有。我们来看看下面这个例子：

```
1. 内核：收到第一个连接请求。线程 A 和 线程 B 两个线程都在 epoll_wait() 上等待。由于采用边缘触发模式，所以只有一个线程会收到通知。这里假定线程 A 收到通知
2. 线程A：epoll_wait() 返回
3. 线程A：调用 accpet() 并且成功
4. 内核：此时 accept queue 为空，所以将边缘触发的 socket 的状态从可读置成不可读
5. 内核：收到第二个建连请求
6. 内核：此时，由于线程 A 还在执行 accept() 处理，只剩下线程 B 在等待 epoll_wait()，于是唤醒线程 B
7. 线程A：继续执行 accept() 直到返回 EAGAIN
8. 线程B：执行 accept()，并返回 EAGAIN，此时线程 B 可能有点困惑("明明通知我有事件，结果却返回 EAGAIN")
9. 线程A：再次执行 accept()，这次终于返回 EAGAIN
```

可以看到在上面的例子中，线程 B 的唤醒是完全没有必要的。另外，事实上边缘触发模式还存在饥饿的问题，我们来看下面这个例子：

```
1. 内核：接收到两个建连请求。线程 A 和 线程 B 两个线程都在等在 epoll_wait()。由于采用边缘触发模式，只有一个线程会被唤醒，我们这里假定线程 A 先被唤醒
2. 线程A：epoll_wait() 返回
3. 线程A：调用 accpet() 并且成功
4. 内核：收到第三个建连请求。由于线程 A 还没有处理完(没有返回 EAGAIN)，当前 socket 还处于可读的状态，由于是边缘触发模式，所有不会产生新的事件
5. 线程A：继续执行 accept() 希望返回 EAGAIN 再进入 epoll_wait() 等待，然而它又 accept() 成功并处理了一个新连接
6. 内核：又收到了第四个建连请求
7. 线程A：又继续执行 accept()，结果又返回成功
```

在这个例子中个，这个 socket 只有一次从不可读状态变成可读状态，由于 socket 处于边缘触发模式，内核只会唤醒 epoll_wait() 一次。在这个例子中个，所有的建连请求全都会给线程 A，导致这个负载均衡根本没有生效，线程 A 很忙而线程 B 没有活干。

#### 3.1.3 怎样才是正确的做法？

既然水平触发和边缘触发都不行，那怎样才是正确的做法呢？有两种 workaround 的方式:

1. 最好的也是唯一支持可扩展的方式是使用从 Linux 4.5+ 开始出现的水平触发模式新增的 `EPOLLEXCLUSIVE` 标志，这个标志会保证一个事件只有一个 epoll_wait() 会被唤醒，避免了 “惊群效应”，并且可以在多个 CPU 之间很好的水平扩展。
2. 当内核不支持`EPOLLEXCLUSIVE` 时，可以通过 ET 模式下的 `EPOLLONESHOT` 来模拟 LT + `EPOLLEXCLUSIVE` 的效果，当然这样是有代价的，需要在每个事件处理完之后额外多调用一次 epoll_ctl(EPOLL_CTL_MOD) 重置这个 fd。这样做可以将负载均分到不同的 CPU 上，但是同一时刻，只能有一个 worker 调用 accept(2)。显然，这样又限制了处理 accept(2) 的吞吐。下面是这样做的例子：
3. `内核：接收到两个建连请求。线程 A 和 线程 B 两个线程都在等在 epoll_wait()。由于采用边缘触发模式，只有一个线程会被唤醒，我们这里假定线程 A 先被唤醒`
4. `线程A：epoll_wait() 返回`
5. `线程A：调用 accpet() 并且成功`
6. `线程A：调用 epoll_ctl(EPOLL_CTL_MOD)，这样会重置 EPOLLONESHOT 状态并将这个 socket fd 重新准备好 “`

#### 3.1.4 其他方案

当然，如果不依赖于 epoll() 的话，也还有其他方案。一种方案是使用 `SO_REUSEPORT` 这个 socket option，创建多个 listen socket 共用一个端口号，不过这种方案其实也存在问题: 当一个 listen socket fd 被关了，已经被分到这个 listen socket fd 的 accept 队列上的请求会被丢掉，具体可以参考 https://engineeringblog.yelp.com/2015/04/true-zero-downtime-haproxy-reloads.html 和 LWN 上的 comment[5]

从 Linux 4.5 开始引入了 `SO_ATTACH_REUSEPORT_CBPF` 和 `SO_ATTACH_REUSEPORT_EBPF` 这两个 BPF 相关的 socket option。通过巧妙的设计，应该可以避免掉建连请求被丢掉的情况。

### 3.2 大量 TCP 连接的 read(2) 的问题

除了 3.1 中说的 accept(2) 的问题之外， 普通的 read(2) 在多核系统上也会有扩展性的问题。设想以下场景：一个 HTTP 服务器，需要跟大量的 HTTP client 通信，你希望尽快的处理每个客户端的请求。而每个客户端连接的请求的处理时间可能并不一样，有些快有些慢，并且不可预测，因此简单的将这些连接切分到不同的 CPU 上，可能导致平均响应时间变长。一种更好的排队策略可能是：用一个 epoll fd 来管理这些连接并设置 `EPOLLEXCLUSIVE`，然后多个 worker 线程来 epoll_wait()，取出就绪的连接并处理[注1]。油管上有个视频介绍这种称之为 “combined queue” 的模型。

下面我们来看看 epoll 处理这种模型下的问题：

#### 3.2.1 水平触发的问题：数据乱序

实际上，由于水平触发存在的 “惊群效应”，我们并不想用该模型。另外，即使加上 `EPOLLEXCLUSIVE` 标志，仍然存在数据竞争的情况，我们来看看下面这个例子：

```
1. 内核：收到 2047 字节的数据
2. 内核：线程 A 和线程 B 两个线程都在 epoll_wait()，由于设置了 EPOLLEXCLUSIVE，内核只会唤醒一个线程，假设这里先唤醒线程 A
3. 线程A：epoll_wait() 返回
4. 内核：内核又收到 2 个字节的数据
5. 内核：线程 A 还在干活，当前只有线程 B 在 epoll_wait()，内核唤醒线程 B
6. 线程A：调用 read(2048) 并读走 2048 字节数据
7. 线程B：调用 read(2048) 并读走剩下的 1 字节数据
```

这上述场景中，数据会被分片到两个不同的线程，如果没有锁保护的话，数据可能会存在乱序。

#### 3.2.2 边缘触发的问题：数据乱序

既然水平触发模型不行，那么边缘触发呢？实际上也存在相同的竞争，我们看看下面这个例子：

```
1. 内核：收到 2048 字节的数据
2. 内核：线程 A 和线程 B 两个线程都在 epoll_wait()，由于设置了 EPOLLEXCLUSIVE，内核只会唤醒一个线程，假设这里先唤醒线程 A
3. 线程A：epoll_wait() 返回
4. 线程A：调用 read(2048) 并返回 2048 字节数据
5. 内核：缓冲区数据全部已经读完，又重新将该 fd 挂到 epoll 队列上
6. 内核：收到 1 字节的数据
7. 内核：线程 A 还在干活，当前只有线程 B 在 epoll_wait()，内核唤醒线程 B
8. 线程B：epoll_wait() 返回
9. 线程B：调用 read(2048) 并且只读到了 1 字节数据
10. 线程A：再次调用 read(2048)，此时由于内核缓冲区已经没有数据，返回 EAGAIN
```

#### 3.2.3 怎样才是正确的做法？

实际上，要保证同一个连接的数据始终落到同一个线程上，在上述 epoll 模型下，唯一的方法就是 epoll_ctl 的时候加上 `EPOLLONESHOT` 标志，然后在每次处理完重新把这个 socket fd 加到 epoll 里面去。

### 3.3 epoll load balance 总结

要正确的用好 epoll(2) 并不容易，要用 epoll 实现负载均衡并且避免数据竞争，必须掌握好 `EPOLLONESHOT` 和 `EPOLLEXCLUSIVE` 这两个标志。而 `EPOLLEXCLUSIVE` 又是个 epoll 后来新加的标志，所以我们可以说 epoll 最初设计时，并没有想着支持这种多线程负载均衡的场景。

### 4. epoll 之 file descriptor 与 file description

这一章我们主要讨论 epoll 的另一个大问题：file descriptor 与 file description 生命周期不一致的问题。

Foom 在 LWN[6] 上说道：

```
显然 epoll 存在巨大的设计缺陷，任何懂得 file descriptor 的人应该都能看得出来。事实上当你回望 epoll 的历史，你会发现当时实现 epoll 的人们显然并不怎么了解 file descriptor 和 file description 的区别。:(
```

实际上，epoll() 的这个问题主要在于它混淆了用户态的 file descriptor (我们平常说的数字 fd) 和内核态中真正用于实现的 file description。当进程调用 close(2) 关闭一个 fd 时，这个问题就会体现出来。

`epoll_ctl(EPOLL_CTL_ADD)` 实际上并不是注册一个 file descriptor (fd)，而是将 fd 和 一个指向内核 file description 的指针的对 (tuple) 一块注册给了 epoll，导致问题的根源在于，epoll 里管理的 fd 的生命周期，并不是 fd 本身的，而是内核中相应的 file description 的。

当使用 close(2) 这个系统调用关掉一个 fd 时，如果这个 fd 是内核中 file description 的唯一引用时，内核中的 file description 也会跟着一并被删除，这样是 OK 的；但是当内核中的 file description 还有其他引用时，close 并不会删除这个 file descrption。这样会导致当这个 fd 还没有从 epoll 中挪出就被直接 close 时，epoll() 还会在这个已经 close() 掉了的 fd 上上报事件。

这里以 dup(2) 系统调用为例来展示这个问题：

```
 1 rfd, wfd = pipe()
 2 write(wfd, "a")             # Make the "rfd" readable
 3
 4 epfd = epoll_create()
 5 epoll_ctl(efpd, EPOLL_CTL_ADD, rfd, (EPOLLIN, rfd))
 6
 7 rfd2 = dup(rfd)
 8 close(rfd)
 9
10 r = epoll_wait(epfd, -1ms)  # What will happen?
```

由于 close(rfd) 关掉了这个 rfd，你可能会认为这个 epoll_wait() 会一直阻塞不返回，而实际上并不是这样。由于调用了 dup()，内核中相应的 file description 仍然还有一个引用计数而没有被删除，所以这个 file descption 的事件仍然会上报给 epoll。因此 `epoll_wait()` 会给一个已经不存在的 fd 上报事件。更糟糕的是，一旦你 close() 了这个 fd，再也没有机会把这个死掉的 fd 从 epoll 上摘除了，下面的做法都不行：

```
1 epoll_ctl(efpd, EPOLL_CTL_DEL, rfd)
2 epoll_ctl(efpd, EPOLL_CTL_DEL, rfd2)
```

Marc Lehmann 也提到这个问题：

```
因此，存在 close 掉了一个 fd，却还一直从这个 fd 上收到 epoll 事件的可能性。并且这种情况一旦发生，不管你做什么都无法恢复了。
```

因此，并不能依赖于 `close()` 来做清理工作，一旦调用了 close()，而正好内核里面的 file description 还有引用，这个 epoll fd 就再也修不好了，唯一的做法是把的 epoll fd 给干掉，然后创建一个新的并将之前那些 fd 全部再加到这个新的 epoll fd 上。所以记住这条忠告：

```shell
永远记着先在调用 close() 之前，显示的调用 epoll_ctl(EPOLL_CTL_DEL)
```

### 4.1 总结

显式的将 fd 从 epoll 上面删掉在调用 `close()` 的话可以工作的很好，前提是你对所有的代码都有掌控力。然后在一些场景里并不一直是这样，譬如当写一个封装 epoll 的库，有时你并不能禁止用户调用 close(2) 系统调用。因此，要写一个基于 epoll 的轻量级的抽象层并不是一个轻松的事情。

另外，Illumos 也实现了一套 epoll() 机制，在他们的手册上，明确提到 Linux 上这个 epoll()/close() 的奇怪语义，并且拒绝支持。

希望本所提到的问题对于使用 Linux 上这个糟糕的 epoll() 设计的人有所帮助。

------

注1：笔者认为该场景下或许直接用一个 master 线程来做分发，多个 worker 线程做处理 或者采用每个 worker 线程一个自己独立的 epoll fd 可能是更好的方案。

### 5 引用

[1]https://idea.popcount.org/2016-11-01-a-brief-history-of-select2/

[2]https://idea.popcount.org/2017-01-06-select-is-fundamentally-broken/

[3]https://idea.popcount.org/2017-02-20-epoll-is-fundamentally-broken-12/

[4]https://idea.popcount.org/2017-03-20-epoll-is-fundamentally-broken-22/

[5]https://lwn.net/Articles/542866/

[6]https://lwn.net/Articles/542866/

[7]https://kernel.taobao.org/2019/12/epoll-is-fundamentally-broken/

[8]https://zh.wikipedia.org/wiki/Epoll

[9]https://stackoverflow.com/questions/4058368/what-does-eagain-mean