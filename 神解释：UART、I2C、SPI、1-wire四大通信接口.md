## 1、裘千丈轻功水上漂之UART

射雕英雄传中的裘千丈说，UART就是我的轻功水上漂过河。想从河上过（通信），提前布暗桩，行走时步伐按桩距固定（波特率提前确定），步幅太大或太小都会落水。

为了不被二弟裘千仞识破，可以安排侍卫在对岸监视通知，没风险才开始表演（流控）。为了保证踩点准确，隔一段距离定个特殊标记的粗木桩。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4W1T4tmuLNx2R03a65ibnv6w7rnNiamt7nOOPpoh8xmBkVbNe1rYnCMSSUzJk7lOMCOl2P6F68sjYLplJfeyf0Rg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

UART通用异步收发传输器（Universal Asynchronous Receiver/Transmitter），通信双方接三根线，RX、TX和GND。其中，TX用于发送数据，RX用于接受数据，双方收发交叉对接，支持全双工方式。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4W1T4tmuLNx2R03a65ibnv6w7rnNiamt7nU5mJKsTTUGcAAzHuVAM2Z6X86GbPiadgyOMdK5awEGoU6K0yX6V3NPg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

因为没有时钟控制，什么时机开始发数据，且保证对方正确接收？

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4W1T4tmuLNx2R03a65ibnv6w7rnNiamt7n2ztgeBb4bwAd9QibibCiaajEE2cxNpt6Uf88QoibZrtj0icbRerMfGB6MMA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

如A发数据到B，平时空闲时A.TX和B.RX.保持1，当A.TX先发0作为起始位，告诉B请注意，我要发数据了。然后就开始发数据，数据位可配置，通常是5位，6位，7位，8位，一帧数据发完后，A.TX给个高电平告诉B.RX我发完了一帧。如果开启校验位，在发停止位之前发送个校验位，一般都不需要校验位了，短距离有线传输出错的概率非常小。如果还有数据，则重复前面的操作。

一般软件配置串口，有波特率，数据位、停止位、校验位、流控。分别表示传输速度，一帧数据的长度，以及发完告知停止，发完是否校验，是否进行发送控制。看起来参数很多，针对个人经验，一般都是固定8位数据位，1位停止位、无校验、无流控，只是配置波特率。

UART没有时钟控制数据捕获时机，依靠通信前就定义波特率，双方按定义的频率读写数据位，正如裘千丈的水上漂，一旦暗桩安装固定，就得按固定的步长行走，否则就会出错落水。

UART在水上漂项目可以，但是传输效率有限，一般高到921600，如果再高可能出现误码，继续加高，就是高空飞行，最后裘千丈就是期望在高空也行走自如，想攀上黄蓉乘坐的大雕逃命，不慎坠落，死于飞行事故。

## 2、叫你一声你敢答应吗之I2C

作为太上老君看银炉的童子，银角大王最懂I2C，万千人中我叫你一声，你答应了就倒霉（从机地址正确才能通信）。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4W1T4tmuLNx2R03a65ibnv6w7rnNiamt7nUs9XZGOLWdk3M0IRfdOyJXm1pvaYCReMentiagYjiaYLYiciadwdt3WEWA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

IIC（Inter Integrated Circuit）两根线，一条时钟线SCL和一条数据线SDA，所以是半双工通信，主从模式，支持一对多，一个银角大王可以对付一群猴子，每个猴子名字不同（从设备的I2C地址不同），点名叫到谁，谁就被紫金葫芦带走。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4W1T4tmuLNx2R03a65ibnv6w7rnNiamt7nPupmicuNz5u63OKxEWUyqZCg3G4udSEOsCic8vOJibd5JZrsLL39L7A7Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

假设主机A给从机B发数据（A.SCL接B.SCL，A.SDA接B.SDA），根据应用，A可以同时接B,C,D。空闲时，SDA和SCL上的电平都为高电平。

**起始和停止**起始条件S：当SCL高电平时，SDA由高电平向低电平转换；停止条件P：当SCL高电平时，SDA由低电平向高电平转换。起始和停止条件一般由主机产生，总线在起始条件后处于busy的状态，在停止条件的某段时间后，总线才再次处于空闲状态。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/MLfSTncC3tOvrRG7SQVPklPHNCERBC6pl4dTeDHcafFYaiaRztDIgf3U82F7EYtMT8eaVDCJNxibUcmevibp2Gqzw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

空闲时SDA和SCL上的电平都为高电平。A先把SDA拉低，等SDA变为低电平后再把SCL拉低（以上两个动作构成了I2C的起始位），此时SDA就可以发送数据了，与此同时，SCL发送一定周期的脉冲，SDA发送数据和SCL发送脉冲的要符合的关系是：SDA必须在SCL是高电平时保持有效，在SCL是低电平时发送下一位（SCL会在上升沿对SDA进行采样）。

**传输与响应**一次传8位数据，8位数据传输结束后A释放SDA，SCL再发一个脉冲（这是第九个脉冲），触发B将SDA置为低电平表示确认（该低电平称为ACK）。最后SCL先变为高电平，SDA再变为高电平（以上两个动作称为结束标志），如果B没有将SDA置为0，则A停止发送下一帧数据。

**整体时序**I2C总线上的每个设备都有唯一地址，数据包传输时先发送地址位，接着才是数据。一个地址字节由7个地址位（可以挂128个设备）和1个指示位组成（7位寻址模式），0表示写，1表示读。一般芯片手册I2C地址都是7位地址，有些与某个引脚的电平相关，主机控制最后读写位。

实际项目一般都是采用I2C库，有的库要求传入的是8位的写的地址，有的是7位，由接口函数再区分读写补位。当然，最愚蠢的办法是从0到255定时循环读某个寄存器地址，读到正确值时的地址就是正确的从机地址。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4W1T4tmuLNx2R03a65ibnv6w7rnNiamt7nOJAzVaJPZhic9cfJQe9UkINicBic8HukEqqDMYnYHsn3YO9CQhh1BuBrQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

一般情况下使用I2C库，除了配置从机地址，其他的起始、结束等时序等其实不太关注，只需要配置时钟频率，一般看从机最大支持多少，以及主机的系统时钟，太高会偶尔出现错误，再没有时间要求的情况下，时钟越低越稳定。

## 3、慕容复斗转星移之SPI

天龙八部的慕容复：虽然我不如乔峰可以使出降龙十八掌，但是他对我出手，我也以彼之道还施彼身，对方输出时也会被反噬，互相伤害，他停止时钟我也无可奈何。正如SPI，主机开启了时钟发数据，从机也在同时输出，时钟停，大家都收手。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4W1T4tmuLNx2R03a65ibnv6w7rnNiamt7nYrmIqknDII0ofNvqU9IUmKyfiaHY3Z5YTnTibwC42VIb2b0dUYa71t8g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

SPI串行外设接口（Serial Peripheral Interface）主从模式，一种高速的，全双工同步的通信总线。标准SPI是4条线。SDI（数据输入）、SDO（数据输出）、SCLK（时钟）、CS（片选，有些也称为SS）。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/MLfSTncC3tOvrRG7SQVPklPHNCERBC6pTvicia0aF8bGCXiazwJB1FS64BEQjGViaRJk08sEYGJgb2ggiaUd6uTYUSw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**SDO/MOSI：**主设备数据输出，从设备数据输入，master output slave input；

**SDI/MISO：**主设备数据输入，从设备数据输出，master input slave output；

**SCLK：**时钟信号，由主设备产生；

**CS/SS：**从设备使能信号，由主设备控制。当有多个从设备的时候，主设备通过片选引脚选择其中一个从设备进行通信。

（I2C是通过软件协议实现多选一，SPI是通过硬件实现。）

![图片](https://mmbiz.qpic.cn/mmbiz_png/4W1T4tmuLNx2R03a65ibnv6w7rnNiamt7nsrrC9eaGfqe3WsbaNzUKOXUVATxF0UoUcKJgb7y7EZqQoFSRXkLkiaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

当主机控制CS，开启时钟闸门，主从双方就可以开始放数据位或者取数据位进行交互了，但在什么时机开始，就有标准了。根据外设工作要求，其输出串行同步时钟极性和相位可以进行配置。

**CPOL：**时钟极性选择，为0时SPI总线空闲为低电平，为1时SPI总线空闲为高电平。

**CPHA：**时钟相位选择，为0时在SCK第一个跳变沿采样，为1时在SCK第二个跳变沿采样。

| mode | CPOL | CPHA |
| ---- | ---- | ---- |
| 0    | 0    | 0    |
| 1    | 0    | 1    |
| 2    | 1    | 0    |
| 3    | 1    | 1    |

这样就有四种模式。以模式1为例，空闲时为低，第一次时钟跳变采样，也就是上升沿读数采样，对着下降沿放数据。如果实在分不清，还有愚蠢的办法，四种模式全部尝试一次，就可知道正确模式。

SPI传输数据没有位数限制，只要定义收发高位在前还是低位在前，可以持续高速传输。

正如前面，若是乔峰收手，慕容复就没法使出降龙十八掌的效果，但是他可以当面骂乔峰是契丹狗，乔峰一怒之下就发功，慕容复就奸计得逞。这契丹狗三字翻译为软件术语就是触发中断，从机发中断告知主机我有事来找我；主机定时查询也可实现，只是使用情况更少。

## 4、裘千尺的吐枣核绝技与1-wire

裘千丈的三妹裘千尺被囚地下，她以口喷射枣核钉打在枣树，树的摇晃就会掉下枣子充饥。这枣核钉是单向操作，用力过猛，枣核透过枣树，用力太轻或者射偏了，枣树没有反应，这样枣核用完了就悲剧了。可见这绝技，看起来简便，实则背后隐藏了精确控制，对时机、位置控制要完美，如1-wire通信，单线控制，时钟精准。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/MLfSTncC3tOvrRG7SQVPklPHNCERBC6phMib7LGUMLqMexELIAva8LsF3HIIecRQY5bwApOHQpc3evDFxoib0nug/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

1-wire总线接口简单，一根线就可以，一般内部开漏输出，外部硬件上拉。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/MLfSTncC3tOvrRG7SQVPklPHNCERBC6papK4SNWrB11rKacxvyZ2y4lu93DgPIdIDbXsEQrQxvpcI0GmQVicJKg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

1-wire使用一条线来传送的四种信令组成，包括复位脉冲和在线应答脉冲的复位序列、写0时隙、写1时隙、读时隙。除在线应答脉冲以外，所有其它信号都由总线主机发出，并且发送的所有数据和命令都是字节的低位在前。主机与从机的数据通信是通过时隙完成的，在每个时隙只能传送一位数据。通过写时隙可把数据从主机传送给从机，通过读时隙可把数据由从器件传送给主机，将完成一位传输的时间称为一个时隙。

一般操作流程参考外设芯片手册，主要是不同平台的延时处理，需要软件实现1us延时的接口，否则数据通信异常。

## 5、秘籍功法

四种接口，每个都有合适的应用场景，对硬件端口的占用、对软件的控制要求、通信效率也不相同。尤其前3种属于常用协议，一般都支持硬件接口，厂家也一般提供hal库，对软件开发人员的要求逐渐降低。这也导致代码应用很溜，实际底层原理略微欠缺，一旦通信异常或者有特殊需求就无从下手。如使用GPIO模拟出UART，使用SPI实现AT功能。

武林人士一般都追求失传的武林秘籍，正如软件开发人员，有问题总是寄希望与其他人的经验总结，或者厂家的技术支持或源码，而不是创造新的功法。笑傲江湖的岳不群本是华山派掌门，精通紫霞神功，武功属于一流，但是没继续专研自家内功，为了辟邪剑谱自宫了，软件开发人员想重蹈覆辙么？

不论剑宗、气宗，先把功能跑通再反推代码原理和实现流程，还是先理清时序和原理再编码实现功能，短期内剑宗效率高，加工资快；气宗则可能被淘汰，尤其在势利的小公司，不注重新人培养。如果合二为一，项目紧急则拿来就用，空闲时专研总结，取长补短，则是完美开发人员的素质。

软件开发没有秘笈功法，全靠个人学习总结。