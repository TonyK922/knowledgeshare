测量C / C ++程序或其某一部分执行时间可能比想象中难，因为许多方法通常无法移植到其他平台。为了选择合适的方法方法，很大程度上取决于操作系统，编译器版本以及“时间”的含义。

本文包括一个综合列表，其中列出了一些当前可用的最佳选项，以及每个选项的局限性。希望在这里您可以找到一个或多个可用于程序的选项。

墙壁时间（Wall time）与CPU时间

首先，重要的是定义和区分在测量执行时间时经常使用的这两个术语。

挂钟时间（也称为时钟时间或挂钟时间）只是测量过程中经过的总时间。这个时间可以使用秒表进行测量了，前提是您能够精确地在所需的执行点启动和停止它。

另一方面，CPU时间是指CPU忙于处理程序指令的时间。等待其他事情完成（例如I / O操作）所花费的时间不包括在CPU时间中。

您应该使用这些定义中的哪一个取决于您为什么首先要测量程序的执行时间。在下面的列表中，大多数方法只计算这些类型的时间中的一种，而只有少数几种能够计算两种时间。同样重要的是，某些同时适用于Linux和Windows用户，而另一些则仅限于特定的操作系统。为了使读者更轻松，我在每个部分的开头明确列出了每种方法的测量时间以及在哪些系统上可用。

## 有关代码示例的注释

下面包括的代码示例基于为求无穷和1/2 + 1 /2¹+ 1 /2²+ 1 /2³+…= 2的近似值的程序。虽然循环的100次迭代足以得出精确的结果，为了获得足够测量的市场我执行10亿次迭代。

在这些程序中，CPU几乎100％的时间都处于忙碌状态，因此挂墙时间和CPU时间之间几乎没有任何差异。如果您想在实验期间让CPU空闲一段时间，可以使用函数`sleep()`（在<unistd.h>中提供）轻松实现。

现在，让我们开始我们的列表。

## 1.使用“ time” Linux命令

***适用于：**仅Linux。（实际上，它可以用于可从终端执行的任何程序。）*
***测量：**挂墙时间和CPU时间。*

好的，这不是真正的C / C ++代码。但是，由于对于许多运行Linux的人来说可能就足够了，因此我决定在更复杂的选项之前添加此选项。如果您只想测量*整个*程序的CPU和/或挂墙时间，则实际上不需要更改代码。在终端命令行运行程序之前加上time，然后，当您的程序执行完毕后，测量的时间将显示在屏幕上。像这样：

```bash
$ time ./MyProgram
Result: 2.00000000000000000000 
 
real 0m5.931s 
user 0m5.926s 
sys 0m0.005s
```

在输出中，“real”表示墙壁时间，“user”表示CPU时间，因此无需更改任何代码行就可以对整个程序进行两种测量。但是，如果要测量程序的*各个部分*所花费的时间，则将需要以下其他选项之一。

**注意：**在编写此代码之前，我一直以为Windows具有自己的命令版本`time`的命令提示符，因此当我发现Windows尚未提供该命令时，我实际上感到很惊讶。如果有兴趣，您可以在网上找到[一些替代方法](https://link.zhihu.com/?target=https%3A//superuser.com/questions/228056/windows-equivalent-to-unix-time-command)，但我想将时间测量直接嵌入到C / C ++代码中应该更容易移植。

## 2.使用<chrono>

**适用于：** *Linux和Windows，**但需要C ++ 11或更高版本。***

*测量**：**挂墙时间。*

这可能是当今测量墙壁时间的最佳，最便捷的方法，但仅在C ++ 11和更高版本上可用。如果您的项目/编译器不支持C ++ 11，则需要本文列出的其他选项之一。

该`<chrono>`库可以访问您机器中的几个不同的时钟，每个时钟都有不同的用途和特性。如果需要，您可以在[此处](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/cpp/chrono)获取有关每种时钟的更多详细信息。但是，除非您确实需要其他时钟，否则我建议只使用`high_resolution_clock`。该时钟使用的是最高分辨率的时钟，因此对大多数人来说可能就足够了。使用方法如下：

```cpp
#include <stdio.h>
#include <chrono>

int main () {
    double sum = 0;
    double add = 1;

    // Start measuring time
    auto begin = std::chrono::high_resolution_clock::now();
    
    int iterations = 1000*1000*1000;
    for (int i=0; i<iterations; i++) {
        sum += add;
        add /= 2.0;
    }
    
    // Stop measuring time and calculate the elapsed time
    auto end = std::chrono::high_resolution_clock::now();
    auto elapsed = std::chrono::duration_cast<std::chrono::nanoseconds>(end - begin);
    
    printf("Result: %.20f\n", sum);
    
    printf("Time measured: %.3f seconds.\n", elapsed.count() * 1e-9);
    
    return 0;
}
```

正如您在第19行看到的那样，我们选择将测得的时间转换为纳秒（尽管稍后将其转换为秒）。如果你愿意，你可以修改代码以使用您选择的其他单元，使用`chrono::hours`，`chrono::minutes`，`chrono::seconds`，`chrono::milliseconds`，或`chrono::microseconds`。

**注意：**我见过有人提到，与其他C / C ++方法相比，使用此库衡量执行时间可能会增加可观的开销，尤其是在循环中多次使用时。老实说，我自己还没有测试或经历过，所以我对此不多说。如果您发现这对您来说是个问题，也许您应该考虑使用以下其他选项之一。

## 3.使用<sys / time.h>和gettimeofday（）

**适用于：** *Linux和Windows。*
*测量**：**挂墙时间。*

该函数`gettimeofday()`返回自1970年1月1日UTC时间00:00:00起经过的时间。棘手的是，该函数在单独的`long int`变量中同时返回秒数和微秒数，因此要获得包括微秒数在内的总时间，您需要将两者进行总计。方法如下：

```cpp
#include <stdio.h>
#include <sys/time.h>

int main () {
    double sum = 0;
    double add = 1;

    // Start measuring time
    struct timeval begin, end;
    gettimeofday(&begin, 0);
    
    int iterations = 1000*1000*1000;
    for (int i=0; i<iterations; i++) {
        sum += add;
        add /= 2.0;
    }
    
    // Stop measuring time and calculate the elapsed time
    gettimeofday(&end, 0);
    long seconds = end.tv_sec - begin.tv_sec;
    long microseconds = end.tv_usec - begin.tv_usec;
    double elapsed = seconds + microseconds*1e-6;
    
    printf("Result: %.20f\n", sum);
    
    printf("Time measured: %.3f seconds.\n", elapsed);
    
    return 0;
}
```

**注1：**如果您不关心秒的分数，则可以通过计算直接获得经过时间`end.tv_sec - begin.tv_sec`。
**注2：**第二个参数`gettimeofday()`用于指定当前时区。由于我们正在计算经过的时间，因此时区无关紧要，前提是`begin`和都使用相同的值`end`。因此，我们对两个调用都使用零。

## 4.使用<time.h>和time（）

**适用于：** *Linux和Windows。*
*测量**：**挂墙时间，**但仅度量整秒。***

该函数`time()`类似于`gettimeofday()`，因为它返回自纪元时间以来经过的时间。但是，有两个主要区别：首先，您无法指定时区，因此始终为UTC。其次，也是最重要的一点，它只返回*整秒*。因此，只有在测量间隔长于几秒钟的情况下，使用这种方法测量时间才有意义。如果毫秒，微秒，纳秒或微秒对您的测量很重要，则应使用其他方法之一。这是使用方法：

```cpp
#include <stdio.h>
#include <time.h>

int main () {
    double sum = 0;
    double add = 1;

    // Start measuring time
    time_t begin, end;
    time(&begin);
    
    int iterations = 1000*1000*1000;
    for (int i=0; i<iterations; i++) {
        sum += add;
        add /= 2.0;
    }
    
    // Stop measuring time and calculate the elapsed time
    time(&end);
    time_t elapsed = end - begin;
    
    printf("Result: %.20f\n", sum);
    
    printf("Time measured: %ld seconds.\n", elapsed);
    
    return 0;
}
```

**注意：** `time_t`实际上与相同`long int`，因此您可以直接使用`printf()`或进行打印，也可以`cout`轻松地将其转换为您选择的其他数字类型。

## 5.使用<time.h>和clock（）

**适用于：** *Linux和Windows。*
*测量**：*** *Linux上的CPU时间和Windows上的挂墙时间。*

该函数`clock()`返回自程序开始执行以来的时钟滴答数。如果将其除以常数`CLOCKS_PER_SEC`，将获得程序已运行多长时间（以秒为单位）。但是，这将取决于操作系统而具有不同的含义：**在Linux上，您将获得CPU时间，而在Windows上，您将获得挂墙时间。**因此，使用此工具时必须非常小心。这是代码：

```cpp
#include <stdio.h>
#include <time.h>

int main () {
    double sum = 0;
    double add = 1;

    // Start measuring time
    clock_t start = clock();
    
    int iterations = 1000*1000*1000;
    for (int i=0; i<iterations; i++) {
        sum += add;
        add /= 2.0;
    }

    // Stop measuring time and calculate the elapsed time
    clock_t end = clock();
    double elapsed = double(end - start)/CLOCKS_PER_SEC;
    
    printf("Result: %.20f\n", sum);
    
    printf("Time measured: %.3f seconds.\n", elapsed);
    
    return 0;
}
```

**注意：** `clock_t`也是`long int`，因此您需要先将其转换为浮点类型，然后再除以`CLOCKS_PER_SEC`，否则您将获得整数除法。

## 6.使用<time.h>和clock_gettime（）

**适用于：仅Linux。**

*测量**：**挂墙时间和CPU时间。*

关于这一点的好处是，您可以使用它来测量墙壁时间和CPU时间。但是，仅在Unix系统中可用。以下措施墙时间的例子，但可以修改它用来衡量CPU时间，只需更换`CLOCK_REALTIME`为`CLOCK_PROCESS_CPUTIME_ID`。

```cpp
#include <time.h>

int main () {
    double sum = 0;
    double add = 1;

    // Start measuring time
    struct timespec begin, end; 
    clock_gettime(CLOCK_REALTIME, &begin);
    
    int iterations = 1000*1000*1000;
    for (int i=0; i<iterations; i++) {
        sum += add;
        add /= 2.0;
    }
    
    // Stop measuring time and calculate the elapsed time
    clock_gettime(CLOCK_REALTIME, &end);
    long seconds = end.tv_sec - begin.tv_sec;
    long nanoseconds = end.tv_nsec - begin.tv_nsec;
    double elapsed = seconds + nanoseconds*1e-9;
    
    printf("Result: %.20f\n", sum);
    
    printf("Time measured: %.3f seconds.\n", elapsed);
    
    return 0;
}
```

**注1：**除了`CLOCK_REALTIME`和`CLOCK_PROCESS_CPUTIME_ID`，您还可以使用此功能的其他时钟。您可以检查[此页面](https://link.zhihu.com/?target=https%3A//linux.die.net/man/2/clock_gettime)以获得更完整的列表。
**注2：**此函数`timespec`使用的结构与`gettimeofday()`（上面方法3）使用的结构非常相似。但是，它包含纳秒而不是微秒，因此在转换单位时要小心。

## 7.使用<sysinfoapi.h>和GetTickCount64（）

**适用于：仅Windows。**

*测量**：**挂墙时间。*

该函数`GetTickCount64()`返回自系统启动以来的毫秒数。也有一个32位版本（`GetTickCount()`），但限制为49.71天，因此使用64位版本会更安全一些。使用方法如下：

```cpp
#include <stdio.h>
#include <sysinfoapi.h>

int main () {
    double sum = 0;
    double add = 1;

    // Start measuring time
    long long int begin = GetTickCount64();

    int iterations = 1000*1000*1000;
    for (int i=0; i<iterations; i++) {
        sum += add;
        add /= 2.0;
    }

    // Stop measuring time and calculate the elapsed time
    long long int end = GetTickCount64();
    double elapsed = (end - begin)*1e-3;

    printf("Result: %.20f\n", sum);

    printf("Time measured: %.3f seconds.\n", elapsed);

    return 0;
}
```

## 8.使用<processthreadsapi.h>和GetProcessTimes（）

**适用于：仅Windows。**

*测量**：*** *CPU时间。*

这是迄今为止列表中最复杂的方法，但它是其中唯一可用于测量Windows CPU时间的方法。我不会详细介绍它的工作原理，因为它过于复杂并且我自己从未使用过，但是您可以查看[官方文档](https://link.zhihu.com/?target=https%3A//docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-getprocesstimes)以了解更多详细信息。这是代码：

```cpp
#include <stdio.h>
#include <processthreadsapi.h>

double get_cpu_time(){
    FILETIME a,b,c,d;
    if (GetProcessTimes(GetCurrentProcess(),&a,&b,&c,&d) != 0){
        //  Returns total user time.
        //  Can be tweaked to include kernel times as well.
        return
            (double)(d.dwLowDateTime |
            ((unsigned long long)d.dwHighDateTime << 32)) * 0.0000001;
    }else{
        //  Handle error
        return 0;
    }
}

int main () {
    double sum = 0;
    double add = 1;

    // Start measuring time
    double begin = get_cpu_time();

    int iterations = 1000*1000*1000;
    for (int i=0; i<iterations; i++) {
        sum += add;
        add /= 2.0;
    }

    // Stop measuring time and calculate the elapsed time
    double end = get_cpu_time();
    double elapsed = (end - begin);

    printf("Result: %.20f\n", sum);

    printf("Time measured: %.3f seconds.\n", elapsed);

    return 0;
}
```

**注意：**我从[Stack Overflow答案中](https://link.zhihu.com/?target=https%3A//stackoverflow.com/a/17440673)改编了这段代码，因此我想将所有功劳归功于在此处发布答案的用户Alexander Yee。实际上，他在这里介绍了一种很好的可移植的方法，可以使用`#ifdef`宏在Linux和Windows计算机上同时计算墙和CPU时间，因此您可能还希望在此处检查完整的答案。

## 最后的想法

好了，您已经拥有了：多种方法来衡量C / C ++的执行时间。正如你所看到的，没有一个放之四海而皆准的解决办法：以上所有方法都有局限性，其中没有一个是能够计算*两个*挂钟时间和CPU时间*和*可*既*用于Linux和Windows。不过，我希望这些方法中至少有一种适用于您的代码以及您的目标。感谢您的阅读。

## 资源资源

如果需要其他信息，下面将为本文列出的每种方法提供更详细的文档：

1. [‘Time’ Linux 命令](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man1/time.1.html)
2. [chrono](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/cpp/chrono)
3. [gettimeofday()](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man2/gettimeofday.2.html)
4. [time()](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man2/time.2.html)
5. [clock()](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man3/clock.3.html)
6. [clock_gettime()](https://link.zhihu.com/?target=https%3A//linux.die.net/man/2/clock_gettime)
7. [GetTickCount64()](https://link.zhihu.com/?target=https%3A//docs.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-gettickcount64)
8. [GetProcessTimes()](https://link.zhihu.com/?target=https%3A//docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-getprocesstimes)