在menuconfig中配置：

详细介绍内核配置选项及删改情况
第一部分：全部删除
Code maturity level options ---> 代码成熟等级选项
[]Prompt for development and/or incomplete code/drivers 默认情况下是选择的，这将会在设置界面中显示还在开发或者还没有完成的代码与驱动.不选。
第二部分 ：除以下选项，其它全部删除
General setup—〉
System V IPC (IPC:Inter Process Communication)是组系统调用及函数库，它能让程序彼此间同步进行交换信息。某些程序以及DOS模拟环境都需要它。为进程提供通信机制，这将使系统中各进程间有交换信息与保持同步的能力。有些程序只有在选Y的情况下才能运行，所以不用考虑，这里一定要选。
第三部分：除以下选项，其它全部删除
Loadable module support ---> 可引导模块支持 建议作为模块加入内核
[] Enable loadable module support 这个选项可以让你的内核支持模块，模块是什么呢？模块是一小段代码，编译后可在系统内核运行时动态的加入内核，从而为内核增加一些特性或是对某种硬件进行支持。一般一些不常用到的驱动或特性可以编译为模块以减少内核的体积。在运行时可以使用modprobe命令来加载它到内核中去(在不需要时还可以移除它)。一些特性是否编译为模块的原则是，不常使用的，特别是在系统启动时不需要的驱动可以将其编译为模块，如果是一些在系统启动时就要用到的驱动比如说文件系统，系统总线的支持就不要编为模块了，否在无法启动系统。
[]Automatic kernel module loading 一般情况下，如果我们的内核在某些任务中要使用一些被编译为模块的驱动或特性时，我们要先使用modprobe命令来加载它，内核才能使用。不过，如果你选择了这个选项，在内核需要一些模块时它可以自动调用modprobe命令来加载需要的模块，这是个很棒的特性，当然要选Y喽。
第四部分：全部删除
Block layer-----〉块设备
第五部分：除以下选项，其它全部删除
Processor type and features ---> 处理器类型
Subarchitecture Type (PC-compatible) ---> 这选项的主要的目的，是使Linux可以支持多种PC标准，一般我们使用的PC机是遵循所谓IBM兼容结构(pc/at)。这个选项可以让你选择一些其它架构。我们一般选择PC-compatible就可以了。
Processor family（386） : 它会对每种CPU做最佳化，让它跑的好又快，一般来说，你是什么型号的就选什么型号的就好。我选的是386，这样内核会省下不少空间
第六部分：除以下选项，其它全部删除
Power management options (ACPI, APM) ---> 电源管理选项
[ ] Power Management Debug Support 电源管理的调试信息支持，如果不是要调试内核有关电源管理部份，请不要选择这项。
ACPI Support ---〉高级电源接口配置支持，如果BIOS支持，建议选上这项
[]Button 这个选项用于注册基于电源按钮的事件，比如power, sleep等，当你按下按钮时事件将发生，一个守护程序将读取/proc/acpi/event，并执行用户在这些事件上定义的动作比如让系统关机。可以不选择，根据自己的需求。
第七部分：除以下选项，其它全部删除
Bus options (PCI, PCMCIA, EISA, MCA, ISA) ---> 总线选项
[]PCI support
PCI access mode (Any) ---> PCI外围设备配置，强列建议选Any，系统将优先使用MMConfig，然后使用BIOS，最后使用Direct检测PCI设备。
第八部分：除以下选项，其它全部删除
Executable file formats --->
Kernel support for ELF binaries ELF是开放平台下最常用的二进制文件，它支持不同的硬件平台。一定要选。
第九部分：除以下选项，其它全部删除
Networking
Networking options --->
[]Unix domain sockets
[]TCP/IP networking
第十部分：除以下选项，其它全部删除
Device Drivers --->设备驱动
Block devices-------〉
[]Compaq SMART2 support
[] Compaq Smart Array 5xxx support
[]Loopback device support 大部分的人这一个选项都选N，因为没有必要。但是如果你要mount iso文件的话，你得选上Y。这个选项的意思是说，可以将一个文件挂成一个文件系统。如果要烧光盘片的，那么您很有可能在把一个文件烧进去之前，看看这个文件是否符合IS09660的文件系统的内容，是否符合您的需求。而且，可以对这个文件系统加以保护。不过，如果您想做到这点的话，您必须有最新的mount程序，版本是在2.5X版以上的。而且如果您希望对这个文件系统加上保护，则您必须有des.1.tar.gz 这个程序。注意：此处与网络无关。建议编译成模块
[] RAM disk support
SCSI device support ---> 里面有关于USB支持的，要选择
[]SCSI device support USB要用，必须选择
[]legacy /proc/scsi/ support USB要用，必须选择
[]SCSI disk support USB要用，必须选择
SCSI Low-level drivers
[]Serial ATA(SATA) support
[]Intel PIIX/ICH SATA support 这个必须选择，否则无法产生引导文件
[]Via SATA support
Networking device support ---> 这个下面是选网卡驱动，一定要选
Ethernet(1000mbit)-我的电脑是千兆网卡所以就选这个
[]broadcom Tigon3support
Input device support ---> 这个里面要设置你的鼠标键盘什么的
[]Provide legacy /dev/psaux device
Graphics support --->
[]Support for frame buffer devices 支持Frame buffer的，一定要选择
USB support --->
[]USB device filesystem 这个好象是用U盘必须的
[]EHCI HCD (USB 2.0) support 有usb2.0就选上把，编译成模块
[]OHCI HCD support 必须选择，编译成模块
[]UHCI HCD (most Intel and VIA) support 必须选择，编译成模块
[]USB Mass Storage support 用U盘必须选择
USB Human Interface Device (full HID) support 里面选择usb鼠标和usb键盘，如果你有一定选上这个必需选
HID input layer support 应该选择
/dev/hiddev raw HID device support如果这里有USB键盘和鼠标选项，一定要选择

第十一部分：除以下选项，其它全部删除
file systems --->文件系统
<*> Second extended fs support
[*] Ext2 extended attributes
[*] Ext2 POSIX Access Control Lists
[*] Ext2 Security Labels
<M> Ext3 journalling file system support
[*] Ext3 extended attributes
[*] Ext3 POSIX Access Control Lists
[*] Ext3 Security Labels 以上这些肯定是要选择的，linux的标准文件系统
<M> Kernel automounter support 内核自动挂载的，当然要选
<M> Kernel automounter version 4 support (also supports v3) 当然要选
DOS/FAT/NT Filesystems --->
<M> DOS FAT fs support
<M> MSDOS fs support
<M> VFAT (Windows-95) fs support
<M> NTFS file system support
Native language support语言支持，这里就支持英语和汉语就行了，不多说了
[]NLS ISO 8859-1 必须选择，这个是关于U盘挂载的。
CD-ROM/DVD Filesystems ---> 这个是关于挂载ISO文件的，用的话就选。
<*> ISO 9660 CDROM file system support
第十二部分: 全部删除
Instrumentation support
第十三部分：全部删除
Kernel hacking --->破解核心？可不是当骸客啦，不选
第十四部分：全部删除
Security options --->
第十五部分：全部删除
Cryptographic options --->这是核心支持加密的选项
第十六部分：全部删除
Library routines --->

附：
内核配置
　　内核配置的方法很多，make config、make xconfig、make menuconfig、make oldconfig等等，它们的功能都是一样的，区别应该从名字上就能看出来，只有make oldconfig是指用系统当前的设置（./.config）作为缺省值。这里用的是make menuconfig。
　　需要牢记:不必要的驱动越多，内核就越大，不仅运行速度慢、占用内存多，在少数情况下、还会引发其他问题。具体步骤如下：
首先确定shell是bash。
然后
$make menuconfig
有一些默认的符号其含义如下：
y：加载
n：不加载
m：作为模块加载

可以配置的选项有以下一些：
1）code maturity level option 代码成熟度
prompt for development and/or incomplete code/drivers [N/y/?]
如果有兴趣测试一下内核中尚未最终完成的某些模块,就选y，否则选N，想知道更详细的信息选？会看到联机帮助（以下？的含义相同）,N大写表示缺省值。

2）processor type and features 处理器类型及特性
Processor family（386，486/Cx486，586/K5/5x86/6x86，Pentium/K6/TSC， PPro/6x86MX）[PPro/6x86MX]
[]内的是缺省值，我们可以根据前面介绍的uname 命令执行的结果选择。此项如果高于386，那么生成的内核在386机器上将不能启动。
Math emulation（CONFIG_MATH_EMULATION）[N/y/?]
需要进行协处理器模拟吗？一般的机器都回n。如果机器已经有硬件的协处理器，那么内核仍将使用硬件，而忽略软件的math-emulation，这将使内核变大变慢。
MTRR（Memory Type Range Register）support（CONFIG_MTRR）[N/y/？]
在Pentium、Pro/Pentium II类的系统中可以提高图像写入速度。
Symmetric multi-processing support（CONFIG_SMP）[Y/n/？]
如果您的机器有多个处理器，就选y。此时要选中下面的Enhanced Real Time Clock Support

3）loadable model support 可加载模块支持
Enable loadable module support（CONFIG_MODULES）[Y/n/？]
最好选y，不然许多仅供动态加载的模块就不能用了。
Set version information on all symbols for modules（CONFIG_MODVERSIONS）[N/y/？]
选N
Kernel module loader（CONFIG_KMOD）[N/y/?]

4）general setup 一般设置
Networking support（CONFIG_NET）[Y/n/?]
选y吧，现在还有几台计算机不用上网呢？
PCI support （CONFIG_PCI）[Y/n/?]
PCI 总线和设备总该有吧。
PCI access mode（BIOS，Direct，Any）[Any]
缺省值比较保险，但如果您对您的主板很有信心，就选BIOS。
PCI quirks （CONFIG_PCI_QUIRKS）[Y/n/?]
用于修补BIOS中对PCI有影响的BUG，同样，如果您对主板很有信心，就选n。
Backward-compatible /proc/pci〉（CONFIG_PCI_OLD_PROC）[Y/n/?]
以前的内核使用/proc/pci，新版内核使用/proc/bus/pci，要保持兼容性就选y。
MCA support（CONFIG_MCA）[N/y/?]
查看帮助吧。
SGI Visual Workstation support（CONFIG_VISWS）[N/y/?]
您的机器是SGI的吗？是就选y。
System V IPC（CONFIG_SYSVIPC）[Y/n/?]
进程间通信函数和系统调用。Linux内核的五大组成部分之一，一定要选。
BSD Process Accounting（CONFIG_BSD_PROCESS_ACCT）[N/y/?]
用于启动由内核将进程信息写入文件的用户级系统调用。就看您想不想用它了。
Sysctl support（CONFIG_SYSCTL）[Y/n/?]
在内核正在运行的时候修改内核。用8KB空间换取某种方便。别选吧，除非你真的想试试。
Kernel support for a.out binaries（CONFIG_BINFMT_AOUT）[Y/m/n/?]
为了能使用以前编译的程序，选y。
Kernel support for ELF binaries（CONFIG_BINFMT_ELF）[Y/m/n/?]
为了能使用现在编译的程序，选y。
Kernel support for MISC binaries（CONFIG_BINFMT_MISC）[Y/m/n/?]
一般选y，用于支持java等代码的自动执行。
Parallel port support（CONFIG_PARPORT）[N/y/m/?]
并口设备，如打印机。

5）plug and play support 即插即用设备支持
Plug and Play support （CONFIG_PNP）[N/y/?]
选y吧。

6）block devices 块设备
Normal PC floppy disk support（CONFIG_BLK_DEV_FD）[Y/m/n/?]
一般的软驱。选y。
Enhanced IDE/MFM/RLL disk/cdrom/tape/floppy support（CONFIG_BLK_DEV_IDE）[Y/m/n/?]
这几种接口的硬盘、光驱、磁带、软驱。选y。
Include IDE/ATAPI CDROM support（CONFIG_BLK_DEV_IDECD）[Y/m/n/?]
CDROM。选y。

7）networking options 网络选项
Packet socket （CONFIG_PACHET）[Y/m/n/?]
按照目前网络发展的状况，选y比较好。当然也可以选其它的。
Kernel/User netlink socke（CONFIG_NETLINK）[N/y/?]
内核与用户进程双向通信。选y。
Network firewalls（CONFIG_FIREWALL）[N/Y/?]
如果真的需要用防火墙，就选y。
UNIX domain sockets（confgi_unix）[Y/m/n/?]
socket 的用处太多了。选y。
TCP/IP networking（CONFIG_INET）[Y/n/?]
选y，理由如上一条。
The IPX protocol （CONFIG_IPX）[N/y/m/?]
其实并没有那么多人真的需要使用或者学习IPX，所以一般选N。
Appletalk DDP（CONFIG_ATALK）[N/y/m/?]
选N，理由同上。

8）SCSI support SCSI支持，SCSI low-level drives SCSI低级驱动
根据系统中SCSI设备的实际情况选择。

9）Networking device support 网络设备支持
如果用LAN上网，就选择网卡；
如果用MODEM拨号上网，就要看ISP提供那种服务了，一般都是PPP。

10）Amateur Radio support 业余收音机支持
这是什么我不太清楚，所以选N。

11）ISDN subsystem ISDN子系统
好像已经有支持ISDN的MODEM了，所以最好先看看自己的MODEM是不是这种，再做选择。

12）Old CD-ROM dfivers （not SCSI， not IDE） 老式光驱驱动
一般选N，因为这种设备实在很少见。

13）Character devices 字符设备
Virtual terminal（CONFIG_VT）[Y/n/?]
Linux上一般可以用Alt+F1/F2/F3/F4来切换不同的任务终端，即使在一台计算机上也可以充分使用Linux的多任务能力，一些需要以命令行方式安装合适用的软件如果有虚拟终端的支持就会更方便，因此选y。
Support for console on virtual terminal（CONFIG_VT_CONSOLE）[Y/n/?]
选y将支持一个虚拟终端作为控制台。一般为Alt+F1。
Support for console on serial port（CONFIG_SERIAL）[Y/m/n/?]
除非真的需要一个串口控制台，否则选n。
Extended dumb serial driver options（CONFIG_SERIAL_EXTENDED）[N/y/?]
如果希望使用"dumb"的非标准特性（如HUB6支持），选y，一般选N。
Non-standard serial port support（CONFIG_SERIAL_NONSTANDARD）[N/y/?]
非标准串口。一般选N。
UNIX98 PTY support（CONFIG_UNIX98_PTYS）[Y/n/?]
PTY指伪终端，一般用户就选n。但如果想用telnet或者xterms作为终端访问主机,并且已经安装了glibc2.1，就可以选y。
Maximum number of UNIX98 PTYs in use（0-2048）（CONFIG_UNIX98_PTY_COUNT）[256]
缺省值就可以了。
Mouse Support（not serial mice）（CONFIG_MOUSE）[Y/n/?]
PS/2等非串口鼠标选y，否则选N。

14）Mice 鼠标
根据自己的鼠标类型选择。

15）Video for Linux Linux视频
根据系统中的音/视频捕捉设备选择。

16）Joystick support 操纵杆
根据系统中的游戏杆设备选择

17）Ftape，the floopy tape device driver Ftape设备驱动
Ftape （QIC-80/Travan）support（CONFIG_FTAPE）[N/y/m/?]
如果系统中有磁带机，选y。

18）Filesystems 文件系统
文件系统的选择要比较仔细，因为其中的一些给某些系统功能提供支持。而且除了proc、ext2等文件系统之外，其它的文件系统（包括下面的网络文件系统）都可以选择为m方式，从而减小内核启动时的体积。
Quota support（CONFIG_QUOTA）[N/y/?]
用于给用户划分定量的磁盘空间。如不用此功能就选N。
DOS FAT fs support（CONFIG_FAT_FS）[N/y/m/?]
为内核提供FAT支持，多数用户有可能从Linux访问同一系统中的WINDOWS硬盘空间，因此最好选y。
ISO 9660 CDROM filesystem support（CONFIG_ISO9660_FS）[Y/m/n/?]
有标准光驱的系统应该选Y。
Minix fs support（CONFIG_MINIX_FS）[N/y/m/?]
用于创建启动盘的文件系统，多数应该选y或者m。
/proc filesystem support（CONFIG_PROC_FS）[Y/n/?]
虚拟文件系统，必须选Y。
Second extended fs support（CONFIG_EXT2_FS）[Y/m/n/?]
Linux标准文件系统，都应该选Y。

19）Network file systems 网络文件系统
Coda filesystem support （advanced network fs）（CONFIG_CODA_FS）[N/y/m/?]
先看帮助再选。
NFS filesystem support（CONFIG_NFS_FS）[Y/m/n/?]
选Y或n，能够访问远程NFS文件系统。
SMB filesystem support（to mount WfW shares etc.）（CONFIG_SMB_FS）[N/y/m/?]
要访问WINDOWS系统中的共享资源选y。
NCP filesystem support（to mout NetWare volumes）（CONFIG_NCP_FS）[N/y/m/?]
如果真的需要访问NetWare文件系统，就选y或者m。

20）Partion Types 分区类型
一般用不上；要用请参看帮助。

21）Console drivers 控制台驱动
VGA text console（CONFIG_VGA_CONSOLE）[Y/n/?]
用VGA模式下用文本方式操作Linux，一般选y。
Video mode selection support（CONFIG_VIDEO_SELECT）[N/y/?]
大多数系统都不需要这项功能。

22）Sound 声音
Sound card support（CONFIG_SOUND）[N/y/m/?]
如果系统中安装了声卡，就选y（或者m），然后查看帮助。

23）Kernel　hacking 内核监视
kernel hacking往往会生成非常大或者非常慢（甚至又大又慢）的内核，甚至会引起内核工作不稳定。如果一定要选，那么也最好不要选其中的"development"、"experimental"、"debugging"项