# Overview

自己接触过许多家的高速serdes，无奈数学基础太差，只能像乌鱼爬一样，一点一点的积累知识和经验，这里把平日里学到的点点滴滴记录下来，希望对自己和同事有所帮助。

# 基础知识 (摘录自李闻界老师的讲课内容)

## Channel的特性

channel的特性。这里第一张图给出了三种它的传输函数，或者说它的s21。在不同的长度，loss区别很大。我们看到第一个蓝色的，它的loss就会比较小，在2.5Gbps时loss大概不到5db;而红色的这种比较长，它的loss在2.5Gbps的时候就有10个dB多一些。这个绿色的一根线，它不光loss多一点，还在10Gbps时有一个很差的一个点的loss，到-60多dB。
一般在设计的时候，会拿一个channel的model一般是Sparameter或者是LGC这种model，然后在前仿真/后仿真去验证我们的design。
![1](/Users/onlyone/Downloads/serdes_md/1.jpeg)从它的冲击响应可以看到红色的在4ns前面一点开始上升，比如说在4ns前一点去sample它，那么可以得到一个数据。但是在后面，比如说4ns多一点的时候去sample它，也可能得到1个1，本来应该是得到了1个0，这就会产生一个错误的数据。
单位冲激响应叠加之后，就会产生一个眼图。或者在PRBS的输入下，折叠输出之后会得到一个眼图。我们看到这个眼图在不同的channel上它的opening，就是高度、宽度差别非常大的

## SerDes的基本结构

SerDes的基本结构图。前面就是把并行数据转换成串行数据（一般也叫MUX），然后再给TX发送出去。因为channel本征阻抗是50Ω，所以TX这边加一个termination，50Ω的特征阻抗来保证阻抗匹配。Rx这边同样加一个50Ω特征阻抗来保证匹配，这样才没有反射。然后RX和TX这边分别有负责时序的，比如说PLL和Timing Recovery，这些模块专门为这些电路提供clock。
![2](/Users/onlyone/Downloads/serdes_md/2.jpeg)
把它可以分成两个主要部分，第一部分就是timing，为发送和接收数据提供clock；第二部分就是signaling，就是信号处理。TX边主要是把数据发送出去保证一定的眼高，然后Rx这边把数据恢复出来。因为channel loss之后，signal可能是看不到的。这就说RX主要做两件事，既要恢复数据、又要恢复时序。恢复数据就是要知道是0还是，对于这种PAM 2也就是NRZ类数据来说。并且要找到怎么去、在哪个时间点上去sample这个data，也就是恢复出一个最好的timing。Rx clock的上升沿采在这个数据它的正中间，这样一般是最好的。
![3](/Users/onlyone/Downloads/serdes_md/3.jpeg)

## Jitter

data上面的jitter其实不是那么在乎，只要保证setuptime、hold time，而clock上的jitter就至关重要。

## 发送端的VCO

RING VCO，一般来说可能在5GHz的clock能做到1.5ps的RMS Jitter。有可能会做的更好，但问题是功耗在继续增加。
对于PAM4来说，它的eye opening会更小，因为它的上升沿有不同transition，这样PAM4的eye opening会小30%。这里的30%具体指的是EYE的宽度，另外EYE的高度比较明显的会小1/3，也就是9.5dB。
对于14GHz的clock或者28Gbps datarate来说，用RING产生时钟比较困难，并且对这种data rate，jitter要求会更高。比如说RMS jitter 0.2ps，这时候大概占UI的11%。在高频的时候，数据恢复这边（signaling）也会更困难，所以要尽量保证这边的Jitter margin多留一点。

## 接收端的CDR

另外一部分就是在RX的timing recovery。CDR的主要目的就是找到一个最好的sample点，使得sample的数据尽量是对准数据的中心店，也就是说BER尽量的小。
CDR的主要性能就是jitter tolerance，它可以track PLL的一部分jitter。还有就是CDR的bandwidth和它的功耗，bandwidth越高功耗就要更高。

## 发送和接收的均衡（EQ）

数据处理主要是均衡器，TX这边叫emphasis，或者叫FFE。RX这边的均衡器会比较多一些，比如说CTLE。有有源的、无源的，可能还会有些其它的滤波器。然后最重要的还有DEF，它有各种结构。
![4](/Users/onlyone/Downloads/serdes_md/4.jpeg)

## TX FFE

如果channel的insertion loss特别小(<15dB)，TX那边直接把信号给发送出来，通过一个50Ω的阻抗匹配。然后RX这边有个50Ω的阻抗匹配，最后直接一个比较器就把它收过来了，这样也是可以的。
但是，当channel的insertion loss比较大(>25dB)的时候，这个问题就十分重要了，TX FFE可以使得频率响应曲线变得平坦，减少ISI。TX输出阻抗和RX端口阻抗要保证50Ω来匹配channel的阻抗，保证没有反射。
在下面的三张图表示均衡器的作用。比如说，Channel一开始时随着频率升高loss越来越大的，但是如果在5GHz的点给一个gain，而在低频时候没有gain。这样就把5GHz这个频点给boost上去，看到第三张图绿色的这个就显得比较平坦，这样的ISI就会小。
![5](/Users/onlyone/Downloads/serdes_md/5.jpeg)
TX端的Equalization是TX的equalization。一般用4-tap FFE，除了一个主的以外，前面有一个，后面还有个POST1、POST2，然后都加在一起后经过50Ω输出。
看一下TX FFE的作用，它是如何消除ISI的。如果没有FFE就是没有post-emphasis。那么data出来之后高频的信号就变得小。而如果有第一个post的emphasis，可以看到第二张图红色的输出的响应，高频的信号明显变大了，低频信号稍微小一点
![6](/Users/onlyone/Downloads/serdes_md/6.jpeg)

## RX CTLE

Rx的均衡技术。第一个就是CTLE，连续时间线性均衡器。它主要来补偿channel对高频信号的insertion loss。它是一种无源的，只有RC或者有L。无源的就意味着没有Gain。它只能把低频的减弱，把高平的相对来说是增高能量。并且无源design有个好处就是它不会产生noise，除了R会产生的noise，但是一般来说R会比较小。而晶体管的noise相对多一些。
CTLE一般是RX的第一级。
在奈奎斯特频率，CTLE一般都会有一些Gain。这样后边的noise就不那么敏感了，比如后面VGA、DFE、sampler。
CTLE一般都是采用可编程的CS、RS，因为channel都不一样，甚至有时还会有自适应算法来自动配置RS、CS(有待商榷??)。
`下图中的fp1就是Nyquist频率，对于10Gbps的baud rate而言，fp1=5GHz。`
![7](/Users/onlyone/Downloads/serdes_md/7.jpeg)

## RX DFE

![8](/Users/onlyone/Downloads/serdes_md/8.jpeg)

DFE技术（decisionfeedback equalization）

1. decision，决定输入的数据是1还是0
2. feedback。也就是说它把这个收到的数据，通过一条链路返回到输入，然后再输入上直接减掉一些信息。那么减掉的量的多少，也就是它输入的数据会乘以一个系数。它减掉多少电压，就是这个信号的加权（W）是多少。
3. 非线性，channel就是非线性的，很多时候非线性还非常大

所以总结下来，DFE包含3点，一个就是decision，第二个就是feedback，然后第三个就是它非线性。
![9](/Users/onlyone/Downloads/serdes_md/9.jpeg)

这里有两个channel，这两个channel的insertion loss不一样，那么接收端的DFE就要自适应这两个channel。并且一般的自适应电路是always on，用它来去cover一些电路或者是那些channel的温飘效应得到。自适应的话就需要额外的比较器，并且一直工作。
![10](/Users/onlyone/Downloads/serdes_md/10.jpeg)

# 知识点进阶

## 不同介质的冲击响应曲线

- PCB Backplane的冲击响应曲线类似像泊松分布
- Fiber Optic的冲击响应曲线类似高斯正态分布
- DAC的冲击响应曲线则介于上面两者之间

## Nyquist频率

一般对于10Gbps信号我们用@5GHz Nyquist频率来看它的Insertion Loss点，25.78125Gbps速率的信号用12.89GHz
以25.78125Gbps速率信号为例，serdes内部的恢复时钟不会是25.78125GHz，因为这样时钟信号太高了，只会恢复出12.89GHz的时钟，然后通过移相90度，对于25.78125Gbps这么高速信号，一般只会sample一个点
对于10Gbps信号，会用2个不同相位的时钟(0相位的5GHz，和移相90度的5GHz)采样4个点。
另外，可以把25.78125Gbps速率的信号接入到频率分析仪的输入端，可以看到它的能量最大的频点是12.89GHz，所以这也可以说明内部恢复出来的时钟不是25.78125GHz,而是12.89GHz.

## CDR的类型

Muller-Muller
Bang-Bang

- CDR其实反而喜欢信号有ISI，如果没有任何ISI，那么CDR肯定会失锁CDR锁定的点是在H(-1)和H(1)之间。
- Muller-Muller CDR两边的采样点是在一半Peak的电平处，Bang-Bang会高一些。
- CDR带宽的需求, CDR支持的带宽主要取决于CDR能够跟踪抖动的速度，速率太高的jitter，CDR无法跟踪，这是由于CDR的反馈是从FFE或者DFE之后过来的，Latency比较大，所以一般CDR只能follow 4MHz的Jitter，有的厂商的实现比较特别，为CDR特别设计了3个dedicated tap的FFE(pre, main , post)，不和data共用，所以latency特别小，可以做到10MHz的带宽
- CDR的RMS jitter是通过phase noise的各个点做积分，计算每个点连接的折线下面的面积。

## ISI

- FFE一般可以configurable,分配多少个Tap给Pre,多少个Tap给Post，1个给Main, 比如16个tap, H0~H3分配给Pre, H4给Main, H5~H15分配给Post
- H0~H3的规律，如果Main是正的, H3(-), H2(+), H1(-), H0(+)，实际上H1和H0往往是0，如果H3(-)的绝对值> 30%*Main, 说明对端发送的Pre需要加强一些
- H5~H15的规律, 如果Main是正的, H5~H15不像pre那样会bounce，一般都是正的或者都是负的，同样的如果H5的绝对值 > 30%*Main，那么说明对端发送的Post需要加强一些
- CTLE, 一般靠手工配置，根据信道的loss来配置相应的值，信道的插损越大，CTLE需要配置的peaking值越大，以保证CTLE的输出满足VGA的输入要求。CTLE主要是压低低频信号，等效于放大高频信号，第一个主要极点是放大Nyquist频率点。有的设计还把CTLE细分成LFP, MFP, HFP

## 接收端眼图的测量

- 接收端眼图的眼图的测量一般位于FFE或者DFE之后，接收端眼图的外眼表征的是低频信号的幅度，内眼表征的是高频信号的幅度。一般只能sample外眼的眼高和眼宽，如果外眼的眼高已经很小，那么内眼的幅度就更小了。所以可以通过外眼的眼高间接的推断出内眼(高频分量)的幅度
- 有的时候，尽管接收的眼图幅度很大，但是宽度很小，这是由于FFE和DFE补偿过头了，这时光看眼高不足以判断接收信号质量的好坏，还必须通过眼宽的数值来综合判断。然而，往往在56G速率以上，眼宽应该比较难以采样。

## FFE和DFE比较

- FFE是非因果关系，对于发送端而言，它可以预先知道下面一个或者几个要发送的bit信号是“0”还是"1"，这样才可以消除Pre的ISI。然后对于接收端而言，它无法知道下面要接收的一个或者多个bit信号是"0"还是“1”，所有在接收端实现FFE的serdes都玩了一个trick，就是把ADC之后的串行码流延迟几个cycle，比如延迟4个cycle，得到bit0, bit1, ... bit[N], 然后把"现在"的bit时间点指向bit4，这样就知道"未来"要接收的bit0..3是"0"还是"1"了。FFE可以出了不会有错误传递之外，还能够补偿pre的ISI干扰
- DFE是因果关系，对于post的ISI干扰补偿非常有效，但是对于pre的ISI干扰无能为力，同时级数多了，会导致错误传递，即多个bit的burst错误。所以56G及以后的serdes，多采用FFE之后只有1个tap的DFE设计

## `如何调节TX FFE参数`

- 根据scope测量的眼图来调节，对于Insertion Loss比较大的信道，要相应增加pre-emphsis和post-emphasis。实际上是做de-emphasis，即变相压低低频信号的幅度，来实现增强高频信号的幅度
- 根据接收端的眼图来调整:
  - 如果发现最靠近main Cursor的第一级pre-cursor，称为H(-1)的绝对值 > 1/3*Main绝对值，那么说明发送端的pre-cursor需要适当增加，如果适当增加了发送端的pre-cursor之后，发现接收端的FFE的H(-1)的绝对值降低下来了，说明调整有效
  - 如果发现最靠近main Cursor的第一级post-cursor，称为H(+1)的绝对值 > 1/3*Main绝对值，那么说明发送端的post-cursor需要适当增加，如果适当增加了发送端的post-cursor之后，发现接收端的FFE/DFE的H(+1)的绝对值降低下来了，说明调整有效
  - 如果发现接收端的VGA的peaking值(放大倍数)已经达到了极限值，那么说明接收信号的幅度过小，接收端需要拼命加大VGA的放大倍数，把信号调整到适合ADC的输入端(典型的400mv~500mv)，这是需要相应调整发送端的Main或者Amplitude，如果发现接收端的VGA的peaking值相应降低了，说明调整有效
- 根据接收端的BER来做相应的调整

## `如何调整RX参数`

- 接收端虽然有很多模块用于EQ，但是一般只有CTLE是手工配置的，VGA, FFE和DFE都是靠算法自动调节的。CTLE需要事先根据信道的Insertion Loss配置相应的值，CTLE的最主要作用是压低低频信号，等效于放大高频信号(实际无法放大，因为CTLE一般是无源的)

## 关于re-timer或者repeater的作用

高速串行信号经过一段距离传输之后，往往在接收端每个bit信号的信号宽度会发生变化，比如发送端发送 1111_0_1111, 这样在接收端的"0"这个数据bit的信号宽度就会特别窄，很可能时钟无法对准信号的中间采样，导致误码。经过re-timer或者repeater的信号重新整形之后，把过窄的信号宽度重新整形成1个UI，然后再发送出去，这样后续的接收器就不容易出错

# 眼图预加重: Pre-cursor and Post-cursor1,2,3

post-cursor和pre-cursor名称中的post和pre的由来:

- “post”是指数据"0" -> "1"或者"1" -> "0"`跳变之后`的预加重
- “pre”是指数据"0" -> "1"或者"1" -> "0"`跳变之前`的预加重

![11](/Users/onlyone/Downloads/serdes_md/11.jpeg)