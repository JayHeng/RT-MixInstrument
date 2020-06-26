# RT-MixInstrument
A Multifunction Instrument based on i.MXRT | 一款基于i.MXRT的多功能仪器（固件），支持示波器/频谱仪/信号发生器模式

### 一、项目背景
　　2020鼠年春节对全国人民来说应该是最清闲的一个春节了，大部分时间都只能躺在家里，关心着2019-nCoV疫情的发展。微信朋友圈充斥着各种疫情的消息，不过我的好友硬禾学堂苏老师却是一股清流，在这假期里，他依旧一心科研，发起的硬禾马拉松活动之基于树莓派的双通道示波器设计吸引了我的兴趣，这让我回想起了读研时曾经凭兴趣做过的简易音频分析仪小项目，前前后后在四块不同开发板上（MSP430最小系统板，原子/超牛/红牛STM32开发板）都实现了原型，于是我赶紧从旧硬盘里翻找当时的项目资料，运气不错资料还在，当时甚至都还拍了照片：  

<img src="http://henjay724.com/image/github/RT-MixInstrument-dso0123.jpg" style="zoom:100%" />  

　　看着自己曾经写的项目设计文档真的感慨万千，但也突然灵机一动，最近我一直在跟超强性能MCU - i.MXRT打交道，何不在i.MXRT上也实现一版看看效果呢，于是一冲动之下便有了这个github项目。  
　　回想起学生时代做这个小项目时，经常看阿莫论坛上的魏坤开源示波器帖子去了解示波器和FFT原理，虽然我最终做出来的效果和完善度跟魏坤比相差太远，但总归也是完成了最初步的原型（当时主要有实验室项目缠身，这个分析仪完全是个人兴趣挤时间做着玩的）。  

**魏坤手持示波仪**
> * [2009 OURDSO第一版（ADS830E+ATmega8/32+LCD320x240）](https://www.amobbs.com/thread-2228838-1-1.html)  
> * [2009 OURDSO第二版（ADC08200+Altera EP2C8Q208+LCD480x272）](https://www.amobbs.com/thread-3613209-1-1.html)  
> * [2011 OURDSO最新版（AD9288+Altera EP3C10E144+LCD480x320）](https://www.amobbs.com/thread-4807710-1-1.html)  

　　现在计划用i.MXRT重新做音频分析仪，肯定希望比当年学生时代做的小玩意更进一步。魏坤的开源示波器项目虽然已经很强大完善了，但后期毕竟切到FPGA平台了，对于MCU做主控的项目来说参考价值不太大了，还好安富莱硬汉哥也搞了一个开源示波器项目，而且完成度比魏坤更高，所以我们就参考硬汉哥的作品来做吧。  

**安富莱开源示波器**
> * [2014 一代（安富莱STM32-V5开发板(内部ADC)+LCD800x480）](http://www.armbbs.cn/forum.php?mod=viewthread&tid=3886&extra=page%3D1)  
> * [2017 二代（安富莱STM32-V6开发板(内部ADC)+LCD800x480）](http://www.armbbs.cn/forum.php?mod=viewthread&tid=45785&extra=page%3D1)  
> * [2018 二代网络版（安富莱STM32-V6开发板(内部ADC)+LCD800x480）](http://www.armbbs.cn/forum.php?mod=viewthread&tid=89526)  

### 二、项目设计
#### 2.1 总体设计
　　本项目主要是为了展示i.MXRT这颗MCU的强大性能，因此直接选用官方配套EVK作为硬件平台，第一阶段计划在i.MXRT1060上实现了示波器原型。  
　　简单地说，示波器主要由三部分组成：信号采集、信号处理、波形显示。咱们一步步说，首先是信号采集，因为输入的是模拟信号，所以我们需要进行模数转换，RT1060内部集成了两个12bit精度1MS/s采样速度的ADC模块，每个ADC最大支持16通道输入，这对于本项目来说够用了。信号处理部分则完全由600MHz CM7内核搞定。最后是波形显示，RT1060-EVK搭配了4.3寸TFT LCD屏，分辨率为480x272，这样的LCD对于一个初级示波器显示来说已经够用了，并且本项目还计划设计一个配套上位机 [RT-MixInstrument-UI](https://github.com/JayHeng/RT-MixInstrument-UI)，i.MXRT将采集转换后的信号数据通过UART/USB发送给上位机显示，那么波形显示上可以不受LCD分辨率限制。  

<img src="http://henjay724.com/image/github/RT-MixInstrument_BlockDiagram_v0.1.JPG" style="zoom:100%" />  

#### 2.2 阶段实现

> * 阶段一：FW实现ADC单通道低速采集并通过串口发送给上位机，UI负责实时波形和频谱显示  
> * 阶段二：FW实现ADC单通道低速采集，并能进行FFT处理，在LCD上实时波形和频谱显示  
> * 阶段三：FW实现ADC双通道高速采集并通过USB发送给上位机，UI负责实时波形和频谱显示  
> * 阶段四：FW实现ADC双通道高速采集，并能进行FFT处理，在LCD上实时波形和频谱显示  


