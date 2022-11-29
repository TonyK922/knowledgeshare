# 详解 NAT : 换个马甲就能上网

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnliaFHiaUxFCBAiaKoZ8AeBDzfSZeiah1ibtloZbJicBsNy3AFPHq1I3xAYBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上帝视角

### 初识 NAT

**IP 地址**分为**公网地址**和**私有地址**。**公网地址**有 IANA 统一分配，用于连接互联网；**私有地址**可以自由分配，用于私有网络内部通信。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbng7tyQMAcCrniaeibZDKba6gayVH0UcFmiakJwFxSa8V4c7RorFqc1l4pA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

私网和公网

随着互联网用户的快速增长，2019 年 11 月 25 日全球的**公网 IPv4 地址已耗尽**。在 IPv4 地址耗尽前，使用 **NAT**（ Network Address Translation ）技术解决 IPv4 地址不够用的问题，并持续至今。

**NAT 技术**的是将私有地址转换成公网地址，使私有网络中的主机可以通过少量公网地址访问互联网。

但 NAT只是一种过渡技术，从根本上解决问题，是采用支持更大地址空间的下一代 IP 技术，即 **IPv6 协议**，它提供了几乎用不完的地址空间。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnS80qibicS8v3KujTUvn1tfCbJt8ohM7Z07G6LCLj29GNIVqTy5iaTIseg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)NAT技术

### NAT 技术

IP 地址中预留了 3 个**私有地址**网段，在私有网络内，可以任意使用。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnSuhQ7BPzcOUCwmPulZLx1ChHMk8YbnARic7XWJB1QICR4zJHtQ1gH3w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

私有地址范围

其余的 IP 地址可以在互联网上使用，由 IANA 统一管理，称为**公网地址**。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbn3po2J4pbwr3aZqp1nQDdF2jYgpg0uzblyH83ahaLREFib0IB33kCEfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

公网地址范围

NAT 解决了 IPv4 地址不够用的问题，另外 NAT 屏蔽了私网用户真实地址，提高了私网用户的安全性。

典型的 **NAT 组网**模型，网络通常是被划分为私网和公网两部分，各自使用独立的地址空间。私网使用私有地址 10.0.0.0/24 ，而公网使用公网地址。为了让主机 A 和 B 访问互联网上的服务器 Server ，需要在网络边界部署一台 NAT 设备用于执行地址转换。NAT 设备通常是**路由器**或**防火墙**。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbn7xhkib5Oic48LJCzgqiaibahC9NvP9pWZG95IRn1s7unibcaWjHtkEW5kVQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

典型NAT组网

#### 基本 NAT

**基本 NAT** 是最简单的一种地址转换方式，它只对数据包的 IP 层参数进行转换，它可分为**静态 NAT** 和**动态 NAT** 。

**静态 NAT** 是公网 IP 地址和私有 IP 地址有一对一的关系，一个公网 IP 地址对应一个私有 IP 地址，建立和维护一张静态地址映射表。

**动态 NAT** 是公网 IP 地址和私有 IP 地址有一对多的关系，同一个公网 IP 地址分配给不同的私网用户使用，使用时间必须错开。它包含一个公有 IP 地址池和一张动态地址映射表。

**举个动态 NAT 栗子**

私网主机 **A**（ 10.0.0.1 ）需要访问公网的服务器 **Server**（ 61.144.249.229 ），在路由器 RT 上配置 **NAT** ，地址池为 219.134.180.11 ~ 219.134.180.20 ，地址转换过程如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnexwa2lziasUSRPngaA9klJAUicoZhwUElf1sPR45WzoSRlNfrB8ibGX6w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

基本NAT

1. **A** 向 Server 发送报文，网关是 10.0.0.254 ，源地址是 10.0.0.1 ，目的地址是 61.144.249.229 。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnVibicCSicNnhMF8dsTDYlPjQqicTc6UzicbGB6PcoffaibNXyd8odo9nJjng/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   A发包

2. **RT** 收到 IP 报文后，查找路由表，将 IP 报文转发至出接口，由于出接口上配置了 NAT ，因此 RT 需要将源地址 10.0.0.1 转换为公网地址。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbn7241sYT9mZJIiciackpVY3cRXEfWicF3RbIUOPicQDnpbEzhesdtqbgSow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)RT收包

3. **RT** 从地址池中查找第一个可用的公网地址 219.134.180.11 ，用这个地址替换数据包的源地址，转换后的数据包源地址为 219.134.180.11 ，目的地址不变。同时 RT 在自己的 NAT 表中添加一个表项，记录私有地址 10.0.0.1 到 公网地址 219.134.180.11 的映射。RT 再将报文转发给目的地址 61.144.249.229 。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnbH7FuaZGGLq7GVf5aSadNbr3rKUbWL7Oric2lFwa8xIpARKvTiabcyEQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)NAT转换

4. **Server** 收到报文后做相应处理。

5. **Server** 发送回应报文，报文的源地址是 61.144.249.229 ，目的地址是 219.134.180.11 。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnyM1GAfRdJsjZKR5FaGYZM1AMviapsic3XNWzQSzBicv2lQqaDPAEIYpaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Server发回应包

6. **RT** 收到报文，发现报文的目的地址 219.134.180.11 在 NAT 地址池内，于是检查 NAT 表，找到对应表项后，使用私有地址 10.0.0.1 替换公网地址 219.134.180.11，转换后的报文源地址不变，目的地址为 10.0.0.1 。RT 在将报文转发给 A 。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnZ4zVWMD5jHuPibFJ9BW1GtfMqTMUEosD5AmuSaNUGWFRr7EKVFjhJibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   NAT转换

7. **A** 收到报文，地址转换过程结束。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnUldDSw54DNL3HVj1yZTiaicpDpTSJNy2WgAiaAyr3JwhDK1ZuCMiaiafkLQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   A收包

如果 **B** 也要访问 Server ，则 RT 会从地址池中分配另一个可用公网地址 219.134.180.12 ，并在 NAT 表中添加一个相应的表项，记录 B 的私有地址 10.0.0.2 到公网地址 219.134.180.12 的映射关系。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnn3ppUPcjmAMvhzEv49MACJuU1QYwNHeXTEB6V5QastKwSj4KVYsKSw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

B的NAT转换

#### NAPT

在基础 NAT 中，私有地址和公网地址存在一对一地址转换的对应关系，即一个公网地址同时只能分配给一个私有地址。它只解决了公网和私网的通信问题，并没有解决公网地址不足的问题。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnjWPUtKfKBpJCVOnjH3K5C7utqMupEOibgd5hUXzFvn07OQxfdpp57IA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

NAT表

**NAPT**（ Network Address Port Translation ）对数据包的 IP 地址、协议类型、传输层端口号同时进行转换，可以明显提高公网 IP 地址的利用率。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnYnC8XvwaoqGfKzr76aKwmIpVjGdwv5icTibuicYRwZmPVwGZAu0zx7H2g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

NAPT的NAT表

**举个栗子**

私网主机 **A**（ 10.0.0.1 ）需要访问公网的服务器 **Server** 的 WWW 服务（ 61.144.249.229 ），在路由器 RT 上配置 NAPT ，地址池为 219.134.180.11 ~ 219.134.180.20 ，地址转换过程如下：

1. **A** 向 Server 发送报文，网关是 RT（ 10.0.0.254 ），源地址和端口是 10.0.0.1:1024 ，目的地址和端口是 61.144.249.229:80 。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnK3qicOfdyH3Q3hibR882EE7Z9MaGnBliaR7c9pOGOdUwoxlWjLSUGcJ1A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   A发包

2. **RT** 收到 IP 报文后，查找路由表，将 IP 报文转发至出接口，由于出接口上配置了 NAPT ，因此 RT 需要将源地址 10.0.0.1:1024 转换为公网地址和端口。

3. **RT** 从地址池中查找第一个可用的公网地址 219.134.180.11 ，用这个地址替换数据包的源地址，并查找这个公网地址的一个可用端口，例如 2001 ，用这个端口替换源端口。转换后的数据包源地址为 219.134.180.11:2001 ，目的地址和端口不变。同时 RT 在自己的 NAT 表中添加一个表项，记录私有地址 10.0.0.1:1024 到 公网地址 219.134.180.11:2001 的映射。RT 再将报文转发给目的地址 61.144.249.229 。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnOyo6reGewgZyuibmEvJV5EUK7aibicdx5iaNEic99o9htLRUdy0QdRr2qog/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   NAPT转换

4. **Server** 收到报文后做相应处理。

5. **Server** 发送回应报文，报文的源地址是 61.144.249.229:80 ，目的地址是 219.134.180.11:2001 。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnMpj1B2FAtEfNnxq9SwSC1JAiaI40EJSu37NIcLJjL44D8pPayKnepUg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   Server发回应包

6. **RT** 收到报文，发现报文的目的地址在 NAT 地址池内，于是检查 NAT 表，找到对应表项后，使用私有地址和端口 10.0.0.1:1024 替换公网地址 219.134.180.11:2001，转换后的报文源地址和端口不变，目的地址和端口为 10.0.0.1:1024 。RT 再将报文转发给 A 。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnRM1r54pSoqeZIAia48XT1ib5vuCtMUMOr7OiarnxzRgJwoic0kicKtTPTUQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   NAPT转换

7. **A** 收到报文，地址转换过程结束。

如果 **B** 也要访问 Server ，则 RT 会从地址池中分配同一个公网地址 219.134.180.11 ，但分配另一个端口 3001 ，并在 NAT 表中添加一个相应的表项，记录 B 的私有地址 10.0.0.2:1024 到公网地址 219.134.180.12:3001 的映射关系。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnTsX7KBiajVPb32Z8waz3m1C30ZePhGPbh8HJkBAlT3rT9e28N6PrVug/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

B的NAPT转换

#### Easy IP

在标准的 NAPT 配置中需要创建公网地址池，也就是必须先知道公网 IP 地址的范围。而在拨号接入的上网方式中，公网 IP 地址是有运营商动态分配的，无法事先确定，标准的 NAPT 无法做地址转换。要解决这个问题，就要使用 **Easy IP** 。

**Easy IP** 又称为基于接口的地址转换。在地址转换时，Easy IP 的工作原理与 NAPT 相同，对数据包的 IP 地址、协议类型、传输层端口号同时进行转换。但 Easy IP 直接使用公网接口的 IP 地址作为转换后的源地址。Easy IP 适用于**拨号接入**互联网，动态获取公网 IP 地址的场合。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnyzInu8TuKTBxfHxWfNibP8HXem52OVhe9xyFzfyu64obJQ10E5zzvHA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)Easy IP

Easy IP 无需配置地址池，只需要配置一个 **ACL**（访问控制列表），用来指定需要进行 NAT 转换的私有 IP 地址范围。

#### NAT Server

从基本 NAT 和 NAPT 的工作原理可知，NAT 表项由私网主机主动向公网主机发起访问而生成，公网主机无法主动向私网主机发起连接。因此 NAT 隐藏了内部网络结构，具有屏蔽主机的作用。但是在实际应用中，内网网络可能需要对外提供服务，例如 Web 服务，常规的 NAT 就无法满足需求了。

为了满足公网用户访问私网内部服务器的需求，需要使用 **NAT Server** 功能，将私网地址和端口静态映射成公网地址和端口，供公网用户访问。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbntcy0z3bhsOTahkhWmRxibFEsPyMicvCo0qVMbddpodBw0wVVSXoc6RIA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)NAT Server

**举个栗子**

**A** 的私网地址为 10.0.0.1 ，端口 8080 提供 Web 服务，在对公网提供 Web 服务时，要求端口号为 80 。在 NAT 设备上启动 NAT Server 功能，将私网 IP 地址和端口 10.0.0.1:8080 映射成公网 IP 地址和端口 219.134.180.11:80 ，这样**公网主机 C** 就可以通过 219.134.180.11:80 访问 A 的 Web 服务。

#### NAT ALG

基本 NAT 和 NAPT 只能识别并修改 IP 报文中的 IP 地址和端口号信息，无法修改报文内携带的信息，因此对于一些 IP 报文内携带网络信息的协议，例如 FTP 、DNS 、SIP 、H.323 等，是无法正确转换的。

**ALG** 能够识别应用层协议内的网络信息，在转换 IP 地址和端口号时，也会对应用层数据中的网络信息进行正确的转换。

**举个栗子：ALG 处理 FTP 的 Active 模式**

**FTP** 是一种基于 TCP 的协议，用于在客户端和服务器间传输文件。FTP 协议工作时建立 2 个通道：**Control 通道**和 **Data 通道**。Control 用于传输 FTP 控制信息，Data 通道用于传输文件数据。

私网 **A**（ 10.0.0.1 ）访问公网 **Server**（ 61.144.249.229 ）的 FTP 服务，在 RT 上配置 NAPT，地址池为 219.134.180.11 ~ 219.134.180.20 ，地址转换过程如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnrV0iah9Zk9BCTaNicJY8OqzmykJyjC0uJsllTpB4wFcuOHrtiaBtMtCgA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)NAT ALG

1. **A** 发送到 Server 的 FTP Control 通道建立请求，报文源地址和端口为 10.0.0.1:1024 ，目的地址和端口为 61.144.249.229:21 ，携带数据是 “ IP = 10.0.0.1 port=5001 ”，即告诉 Server 自己使用 TCP 端口 5001 传输 Data。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnTr0Q2rwANaw8qbTlr0HQgMgia7euPbdeuicAe9iam6uoWIcMibSXKsppiaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   A发包

2. **RT** 收到报文，建立 10.0.0.1:1024 到 219.134.180.11:2001 的映射关系，转换源 IP 地址和 TCP 端口。根据目的端口 21 ，RT 识别出这是一个 FTP 报文，因此还要检查应用层数据，发现原始数据为 “ IP = 10.0.0.1 port=5001 ”，于是为 Data 通道 10.0.0.1:5001 建立第二个映射关系：10.0.0.1:5001 到 219.134.180.11:2002 ，转换后的报文源地址和端口为 219.134.180.11:2001 ，目的地址和端口不变，携带数据为 “ IP = 219.134.180.11 port=2002 ”。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnsJ8Mbw0ZCdEx3FSllWINcoKkZmic6OmSZrwVoUiag21eVxeyaursFxeA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   NAT ALG转换

3. **Server** 收到报文，向 A 回应 command okay 报文，FTP Control 通道建立成功。同时 Server 根据应用层数据确定 A 的 Data 通道网络参数为 219.134.180.11:2002 。

4. **A** 需要从 FTP 服务器下载文件，于是发起文件请求报文。Server 收到请求后，发起 Data 通道建立请求，IP 报文的源地址和端口为 61.144.249.229:20 ，目的地址和端口为 219.134.180.11:2002，并携带 FTP 数据。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbn9mgRiccX7aHV8O4iasGtqEXpTic0PibdIhlFbfxuGywkHBTicicnj5vdhS1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

   Data通道

### NAT 实战

#### 基本 NAT 实验

##### 实验拓扑图

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnQZ2lzROVQwxLDsZbc7gTsZ5Lq0GLMBjx1lhXia2icWTLTHE4vibNjmqow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)拓扑图

##### 实验要求

- ENSP 模拟器
- PC 通过公网地址访问互联网

##### 实验步骤

1. 根据接口 **IP 地址表**，配置各个设备的接口地址。

   ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnHAN1tlDxIV4QeZvuD5GRMrbGvSbCfMTKMZInC8kmdTYkdn2h5ryG0A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

   IP地址表

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnR47X7701ekYlz3hHUudFpgPp3kdnc0By8C8uofTcl9NuuNtVqXpGxg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

PC配置

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnb7MyGPMkWKgS5ibHlmlgQYGyYgtYWF2PibRGCr88NP1aVAlcxemMBu5A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)RT配置

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnZIeasziaElfLz1CIgtxOt0apkCBTeLicSM8mGhhDibhSIv9vwHor7mUdQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)ISP配置

1. 在 RT 上配置 **NAT 配置**。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnJokJGH4u74453eibovBTWrbsYD4rCKM2lkyuDTaltf7chxkNvAvx2wA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)NAT配置

配置**基本 NAT** 只需要一条命令：把私有 IP 地址转换成公网 IP 地址，在接口视图下配置 **nat static global** *global-address* **inside** *host-address* 命令。**默认路由**是网关路由器上的常见配置。使用 **display nat static** 命令查看 RT 上的静态 NAT 配置。

1. 在 PC 上**验证**联网功能。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnQdzgQAeFico9H4Qp2I97NLDW2JVr7GI8tMlibAibHVSVCgSiadrmibesZPA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

PC验证结果

1. **抓包**查看 NAT 转换效果。分别抓包 RT 的内网口 G0/0/0 和外网口 G0/0/1 的报文，看出发送的 Echo Request 报文和接收的 Echo Reply 报文都有进行 NAT 转换。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnTmKejdiciaqtgibdu887snx69EeCknRuwHIxSBnpmic6VK9LlrE5PcbGLQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)RT内网口抓包

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnqQRaBGpv46JrxTMqzCkBxYOB64gBluxzicBflzoz0wbWCg7A82KjW1g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)RT外网口抓包

#### NAPT 实验

##### 实验拓扑图

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnLfCVh2IU5pHOkuticS2iaqx3mkuOrl0bwMBPUC2kGkmez7mjvXic0uNicw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)拓扑图

##### 实验要求

- RT 使用 NAPT 功能
- ISP 分配 4 个可用的公网地址：202.0.0.3 ~ 202.0.0.6
- VLAN 10 的用户使用两个公网地址
- VLAN 20 的用户使用另外两个公网地址

##### 实验步骤

1. 根据接口 **IP 地址表**，配置各个设备的接口地址。配置命令可参考上一个实验步骤 1 。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnSaniaIeIVAWVXTNpulRoTyibx1jicrsSz6Fkiaw1LROiaNXxtxYHFrzgeOg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

IP地址表

1. 在 RT 上配置 **NAPT** 配置。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnteFk501WF0wPDT5bCQUzMwX1wYgQk1SFVtSCXdEPd8vlX5cPy3HibBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)NAPT配置

在 NAPT 的配置中，使用**基本 ACL** 来指定私有 IP 地址范围。ACL 2010 指定 VLAN 10 的 IP 地址空间，ACL 2020 指定 VLAN 20 的 IP 地址空间。使用 **nat address-group** *group-index start-address end-address* 命令指定公网 IP 地址范围，分别指定了两个 NAT 地址组，编号分别选择了 1 和 2 。在外网接口上，使用 **nat outbound** *acl-number* **address-group** *group-index* ，绑定 NAT 转换关系。

使用 **display nat address-group** 命令查看 RT 上的 NAT 地址组配置。命令 **display nat outbound** 查看出方向 NAT 的转换关系。

1. 分别在 PC10 和 PC 20 上**验证**上网功能。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbn3uyC8YFNdibFD3fA3UH27fooiccL8Hia50Jd3QQAibjjQc5Jg6qYz2aO2A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

PC10验证结果

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnoTKaQc2ExOxIrZysj9Ufg6yAnasf5qfbA8bQbqCNyU6lQZ1jJBU3kA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

PC20验证结果

1. **抓包**查看 NAT 转换效果。分别抓包 RT 的内网口 G0/0/1 和外网口 G0/0/0 的报文，查看 VLAN 10 的用户出发送的 Echo Request 报文和接收的 Echo Reply 报文都有进行 NAT 转换。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnmibHVNjfIFicOuVMrxs41d1iaaNhWeKTLNjvIYw1WJiabEtcBlaynXXJZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)RT内网口VLAN10抓包

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/NQOMbFoOWXdxHQIYmCPFXIABXzNDpibbnu3Fd52oqSY2AL8UhiaiaCtyT2W5pnrD3jFwMpa6qFE3coDT0t8mqouGA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)RT外网口抓包

#### 其它常用 NAT 命令

NAT Server 是在接口视图下配置，命令格式为：**nat server protocol** { **tcp** | **udp** } **global** *global-address global-port* **inside** *host-address host-port* 。

检查 NAT Server 配置信息命令：**display nat server** 。

检查 NAT 会话命令：**display nat session all** 。

启动 NAT ALG 功能命令：**nat alg** *all* **enable** 。

查看 NAT ALG 功能命令：**display nat alg** 。

------

**参考资料：**
高级网络技术 - 田果
华为防火墙技术漫谈 - 徐慧洋
HCNA网络技术学习指南 - 华为技术有限公司
路由交换技术 - 杭州华三通信技术有限公司