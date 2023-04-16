# popcount 算法分析

population count，简称 [popcount](http://en.wikipedia.org/wiki/Hamming_weight) 或叫 sideways sum. 是计算一个数的二进制表示有多少位是1，在一些场合下很有用，比如计算0-1稀疏矩阵(sparse [matrix])或位数组(bit array)中非零元素个数、比如计算两个字符串的汉明距离(Hamming distance)。然而 Intel 直到2008年11月 Nehalem 架构的处理器Core i7才引入了 SSE4.2 指令集，其中就有 CRC32 和 POPCNT 指令，POPCNT可以处理16,32,64位整数。

## iterated_popcnt

**代码分析：**

```c
int iterated_popcnt(unsigned int n)
{
    int count=0;
    for(;n;n>>=1)
        count+=n&1u;
    return count;
}
```

iterated_popcnt 通过迭代查找1累加，多少位就循环多少次，简单明了也因此最慢。



## sparse_popcnt

```c
int sparse_popcnt(unsigned int n)
{
    int count=0;
    while(n)
    {
        ++count;
        n&=(n-1);
    }
    return count;
}
```

sparse_popcnt 是对 iterated_popcnt 的改进，每次迭代总是将最右边的非零位置零。减法的妙用。
试想一下，一个仅最高位为1的整数，用此方法的话仅需一次迭代；而 iterated_popcnt 还是会“乖乖的”迭代64次，对64位整形来讲。



## dense_popcnt

```c
int dense_popcnt(unsigned int n)
{
    int count = CHAR_BIT*sizeof(unsigned int);
    n^=(unsigned int)-1;
    while(n)
    {
        --count;
        n&=(n-1);
    }
    return count;
}
```

dense_popcnt 只是 sparse_popcnt 的一个变种。
如果我们有先验条件，知道数n的bit位有很多很多的1(取名 dense 的由来)，可以先取反求 popcount，最后用int位数一减就得到结果。如果数很随机，即数的1、0位数差不多(not too sparse, not too dense)，将失去优势，与 sparse_popcnt 表现一样，如果忽略掉开始的一次取反操作。
n&(n-1) 式子还有一个用途，就是判断一个整数 x 是否是 2 的 n 次幂。条件是 **x!=0 && (x&(x-1))==0** 。



## lookup_popcnt

```c
int lookup_popcnt(unsigned int n)
{
#if 0 /* generate the table algorithmically, and you should put it outside. */
    static unsigned char TABLE[256]={0};
    for(unsigned int i=1;i<256;++i)
        TABLE[i]=TABLE[i>>1]+(i&1u);
    
    unsigned char *p=(unsigned char*)&n;
    return TABLE[p[0]]+TABLE[p[1]]+TABLE[p[2]]+TABLE[p[3]];
    
#else

# define BIT2(n) n,            n+1,            n+1,            n+2
# define BIT4(n) BIT2(n), BIT2(n+1), BIT2(n+1), BIT2(n+2)
# define BIT6(n) BIT4(n), BIT4(n+1), BIT4(n+1), BIT4(n+2)
# define BIT8(n) BIT6(n), BIT6(n+1), BIT6(n+1), BIT6(n+2)

    static const unsigned char TABLE[256]={BIT8(0)};

    return TABLE[n       & 0xff]+
           TABLE[(n>>8)  & 0xff]+
           TABLE[(n>>16) & 0xff]+
           TABLE[(n>>24) & 0xff];
           
#endif          
}
```

lookup_popcnt 通过查表法，用空间换时间，所以来得快。
这里没有制作整个 uint 区间的（需太大空间）表，而是一个 uchar 表，分四次求值然后相加。你可以尝试 ushort 表，然后查表两次(上面用了四次)，看会不会更快，毕竟直接索取结果应该比任何计算都要来得快，按理说。 
上面的代码列出了两种方法构造 TABLE：

1. 利用 f(n)=f(n/2)+n%2, f(0)=0。直观而且方便移植。不要求char一定为8位，即(CHAR_BIT==8)

2. 利用宏扩展递归展开，很漂亮的写法。**上面假定了char为8位**。有些旧的机器一个字节7位，就需要作一些修改。

CHAR_BIT 和 UCHAR_MAX 作为宏定义在 limit.h 文件中，方便移植。

```c
int lookup_popcnt(unsigned int n)
{
#define TBL_LEN (1<<CHAR_BIT)

#define BIT2(n)      n,       n+1,       n+1,       n+2
#define BIT4(n) BIT2(n), BIT2(n+1), BIT2(n+1), BIT2(n+2)
#define BIT6(n) BIT4(n), BIT4(n+1), BIT4(n+1), BIT4(n+2)
#define BIT8(n) BIT6(n), BIT6(n+1), BIT6(n+1), BIT6(n+2)

    static const unsigned char TABLE[TBL_LEN]=
        {
#if (CHAR_BIT==8)
            BIT8(0)
#elif (CHAR_BIT==7)
            BIT6(0), BIT6(1)
#else
# error "BITX to be added here."
#endif
        };

    return TABLE[n                 & UCHAR_MAX]+
           TABLE[(n>>CHAR_BIT)     & UCHAR_MAX]+
           TABLE[(n>>(CHAR_BIT*2)) & UCHAR_MAX]+
           TABLE[(n>>(CHAR_BIT*3)) & UCHAR_MAX];

#undef TBL_LEN
}
```

## parallel_popcnt

```c
#define POW2(c) (1u<<(c))
#define MASK(c) (((unsigned int)(-1))/(POW2(POW2(c))+1u))
#define COUNT(x,c) ((x)&MASK(c)) + (((x)>>(POW2(c)))&MASK(c))

int parallel_popcnt(unsigned int n)
{
    n=COUNT(n,0);
    n=COUNT(n,1);
    n=COUNT(n,2);
    n=COUNT(n,3);
    n=COUNT(n,4);
/*  n=COUNT(n,5);  for 64-bit integers */
    return n ;
}
```

想象一下已算好的每k位一组的 bitcount，向`2*k`位挺进。
`num=a*2k+b`, a为高k位，b为低k位，于是结果为 a+b。
用关于num的表达式，用mask分别屏蔽高低位取出a和b再加。
如此，32位整数需5次迭代，64位整数需6次迭代。

关于第n次迭代MASK的值
    [0]. 01010101 01010101 01010101 01010101 = 55555555 = FFFFFFFF/(0x2+1)
    [1]. 00110011 00110011 00110011 00110011 = 33333333 = FFFFFFFF/(0x4+1)
    [2]. 00001111 00001111 00001111 00001111 = 0F0F0F0F = FFFFFFFF/(0x10+1)
    [3]. 00000000 11111111 00000000 11111111 = 00FF00FF = FFFFFFFF/(0x100+1)
    [4]. 00000000 00000000 11111111 11111111 = 0000FFFF = FFFFFFFF/(0x10000+1)
上面除了左边第一列是二进制外，右边的都是十六进制数。
第n次的数的规律是 `2^n` 个0，接着` 2^n` 个1，然后循环。`2^2^n`对应高`2^n`位，1对应低`2^n`位。
所以右边的除数是 `2^2^n+1`，此处^为乘方，右结合。其实就是费马数(Fermat number)。

## nifty_popcnt

```c
#define MASK_01010101 (((unsigned int)(-1))/3)
#define MASK_00110011 (((unsigned int)(-1))/5)
#define MASK_00001111 (((unsigned int)(-1))/17)

int nifty_popcnt(unsigned int n)
{
    n = (n & MASK_01010101) + ((n>>1) & MASK_01010101) ;
    n = (n & MASK_00110011) + ((n>>2) & MASK_00110011) ;
    n = (n & MASK_00001111) + ((n>>4) & MASK_00001111) ;        
    return n%255 ;
}
```

## hacker_popcnt

```
int hacker_popcnt(unsigned int n)
{
    n -= (n>>1) & 0x55555555;
    n  = (n & 0x33333333) + ((n>>2) & 0x33333333);
    n  = ((n>>4) + n) & 0x0F0F0F0F;
    n += n>>8;
    n += n>>16;
    return n&0x0000003F;
}
```

取名 hacker_popcnt 是因为来自于 [*Hacker's Delight*](http://www.hackersdelight.org/) 这本书。网上有电子版下载，强烈推荐翻阅。
书中第5章专门讲 Counting Bits。上面的那段代码来自第一节的 `Figure 5-2 Counting 1-bits in a word.`该代码也被 [Android](http://en.wikipedia.org/wiki/Android_%28operating_system%29) 开源项目所用，路径` <AOSP>libcore/luni/src/main/java/java/lang/FFmpeg` 中也有这段代码，路径 libavutil/common.h 可以 grep 一下关键字 popcount。其中的 Integer 和 Long 类的方法 bitCount，分别对应32位和64位的 popcount
第一行的减法的由来：二进制二位数 2a+b 左移一位得 a，于是 (2a+b)-a=a+b
方法原理同上面的 nifty_popcnt 一样，这里不多描述。

## hakmem_popcount

```c
/* HAKMEM Popcount

  Consider a 3 bit number as being
        4a+2b+c
  if we shift it right 1 bit, we have
        2a+b
  subtracting this from the original gives
        2a+b+c
  if we shift the original 2 bits right we get
        a
  and so with another subtraction we have
        a+b+c
  which is the number of bits in the original number.

  Suitable masking  allows the sums of  the octal digits  in a 32 bit  number to
  appear in  each octal digit.  This  isn't much help  unless we can get  all of
  them summed together.   This can be done by modulo  arithmetic (sum the digits
  in a number by  molulo the base of the number minus  one) the old "casting out
  nines" trick  they taught  in school before  calculators were  invented.  Now,
  using mod 7 wont help us, because our number will very likely have more than 7
  bits set.   So add  the octal digits  together to  get base64 digits,  and use
  modulo 63.   (Those of you  with 64  bit machines need  to add 3  octal digits
  together to get base512 digits, and use mod 511.)
 
  This is HACKMEM 169, as used in X11 sources.
  Source: MIT AI Lab memo, late 1970's.
*/

int hakmem_popcount(unsigned int n)
{
    unsigned int tmp;
    tmp = n - ((n>>1) & 033333333333) - ((n>>2) & 011111111111);
    return ((tmp + (tmp>>3)) & 030707070707) % 63;
}
```

## assembly_popcnt
```c
int assembly_popcnt(unsigned int n)
{
#ifdef _MSC_VER /* Intel style assembly */
    __asm popcnt eax,n;
#elif __GNUC__ /* AT&T style assembly */
    __asm__("popcnt %0,%%eax"::"r"(n));
}
```

最终来到了汇编指令 POPCNT，汇编语言有两种style：Intel 和 AT&T。语法一样，只是写法不同。之间的区别自己 Google。
Visual C++ 用的是 Intel style；类UNIX 使用 AT&T style。大学学的是前面一种。个人觉得 Intel 的简单易懂，上面的一行指令足已证明。C/C++函数的返回值都是存入 EAX 寄存器的。函数很简短，所以被调用的时候可以直接 inline 到其他函数。

关于汇编指令的用法，请查阅 Intel 官方手册，地址在[这里](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)。你的机器不一定支持（编译通过但运行出错），所以最好先检测一下。 Linux 平台通过 cat /proc/cpuinfo | grep popcnt 命令就可以看到是否支持。*Before an application attempts to use the POPCNT instruction, it must check that the processor supports SSE4.2 (if CPUID.01H:ECX.SSE4_2[bit 20] = 1) and POPCNT (if CPUID.01H:ECX.POPCNT[bit 23] = 1).*

asm 本来是 C++ 的关键字的，[MSDN](http://msdn.microsoft.com/en-us/library/2e6a4at9%28v=vs.80%29.aspx)上说 *The __asm keyword replaces C++ asm syntax. asm is reserved for compatibility with other C++ implementations, but not implemented. Use__asm.* 如果用 GCC 编译器的话，可以使用asm，但编译的时候要用 gcc popcnt.c -o bitcnt -std=c99 -fasm，-fasm是让编译器认"asm", "inline" or "typeof"为关键字。
[GNU编译器也内置了很多函数](http://gcc.gnu.org/onlinedocs/gcc-4.3.0/gcc/X86-Built_002din-Functions.html)，也包括 int __builtin_popcount (unsigned int x);，如果你的机器架构支持的话，会直接译成一条CPU指令。