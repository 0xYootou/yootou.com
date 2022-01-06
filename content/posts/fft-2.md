
+++
resources = []
title = "Arduino LED 频谱仪 #2 音频传感器选择"
mainContent = ""
pubdate = "2022-01-06"
featuredImage = "/fft-2/IMG_6475.JPG"
+++


## 特别注意

今天跟大家分享下关于音频采集和频域分析的过程，也比较简单，不过相比于网上很多示例来说，至少是真的 run 起来了，之前我在网上看到很多文章和示例代码，实际上都非常浅，也基本上很难跑起来，所以也趟了不少坑。

另外我在这个环节趟过的最大的坑是电气隔离的坑，因为麦克风采集的电信号非常微弱，所以它很容易被电路干扰，实际上玩 hifi 的同学应该对相关的经验比较了解，按理说我也是个 hifi 玩家，应该比较早想到这个问题的，但是我的确忽略了，因为调了很久电路，一直达不到自己想要的结果，而且差异很大，后来突然想起来这回事，把电路做了隔离才ok的。

另外就是关于元器件的运行电压，例如今天要讲的麦克风传感器，官方规格描述是要低于3.3v电压，否则容易损坏，所以不能直接接入usb的5v电压。

## 音频采集
说到音频采集，最重要的当然是麦克风，新接触的同学可能会直接买mic头，这就麻烦了，mic 只是最末端的传感器，他只能产生模拟信号，你需要自己搭建周边电路和放大电路、编码电路，才能真正获得数字音频信号，而且需要把信号编码传给 MCU，过于复杂了，实际上我们要的是集成的传感器，直接输出数字信号。

去淘宝搜索了一圈，发现了这些玩意：

![](/fft-2/IMG_6487.jpeg)

注意下，先别急着买，实际上我们使用的不是这里的任意一款传感器。

### 第一种：声音传感器
首先，第一和第二种，叫声音传感器，而不是麦克风放大器，这些模块不是用来录音的，而是用来探测分贝的，也就是他们可以设置一个阈值，当声音的分贝大于阈值的时候就会输出高电平，否则会输出低电平，所以它的输出只有两个状态，所以他们的使用场景是类似于楼道里的声控开关。

### 第二种：麦克风放大传感器
继续看其他的模块，这些模块是可以探测声音的，其实可以用于今天的小作品，不过这些传感器的端口都是普通的 io 口，他们的输出信号非常直白，就是电压值，然后通过 arduino 的 adc 口去探测值，分辨率很低，采样率也很低。

这种传感器的接口通常只有3个或4个，除去 VCC 和 GND ，就是一根普通的采集输出。

Arduino uno 的 A0-A5 可以做 adc 采样，采样分辨率1024，也就是采样一次，值的范围可以有 1024 级。代码非常简单：

```
void setup() {
   Serial.begin(9600);         //使用9600速率进行串口通讯
}
 
void loop() {
  int n = analogRead(A0);    //读取A0口的电压值
  double vol = n * (5 / 1024.0*100);   //使用双精度浮点数存储数据，数据由电压值换算得到
  delay(2000);                           //等待2秒，控制刷新速度
}

```

途中还有一个 MAX9814，也是一个非常流行的传感器，提供了一些高级功能，另外性能也比较好，可以控制增益调节，低功耗待机等。

官方参数：

> MAX9814 IC包含低噪声放大器、输出放大器、麦克风偏置电压发生器和自动增益控制(AGC)等内部电路。麦克风放大器的总增益可选择40dB、50dB或60dB而无压缩。MAX9814利用压缩/限幅电路将麦克风输出限制为设定电压。  
* MAX9814评估板工作于2.7V至5.5V电源范围
*  7V至5.5V单电源工作
* 20dB动态增益压缩
* 可选的增益控制
* 可编程动作时间
* 可选择动作/释放比
* 低功耗关断模式

![](/fft-2/max1.jpg)

### 第三种： MEMS 全向麦克风

我最终选用的是高性能全向麦克风，具体型号是 INMP 441，体积非常小巧。由一个 MEMS传感器，信号组成调节，模数转换器，抗混叠滤波器，电源管理和行业标准的24位I²S接口。I²S接口允许INMP441直接连接到数字处理器，如DSP和微控制器，无需使用用于系统中的音频编解码器。INMP441具有高信噪比，是款选择近场应用。 INMP441具有扁平宽带频率响应，导致自然声音高清晰度。

* 具有高精度24位數據的數字I²S接口
* 高信噪比為61 dBA
* 高靈敏度-26 dBFS
* 從60 Hz到15 kHz的穩定頻率響應

我们为啥选 INMP 441 呢，最主要的原因其实是他的 i2s 接口，另外他的分辨率是很强的，而且信号是内部处理好的，同时还具备一定的 buffer ，因为我们的 arduino 芯片还需要处理傅里叶变换，控制，时间显示，联网等，所以使用 i2s 接口的传感器，mcu 基本不需要付出成本去接入麦克风传感器，而只需要定期从 i2s 接口的缓冲区获取结果数据即可。

不过，上篇文章我们提到过，不是所有的 arduino 芯片都有 i2s 接口，而且大部分 arduino 其实都没有这个接口，所以一定要配套，我们这里使用的是 esp32 芯片来配套，esp32 内部有两套 i2s 通信 io。

通过 esp32 和 INMP441 ，做频谱显示其实是大材小用，其实完全可以用来做录音笔，保存成 wav 文件，然后存储到 sd 卡，甚至是通过 websocket 传输到服务器，这些用 esp32 芯片都完全可以搞定。

下图里的这两个圆形的芯片就是 INMP441，虽然体积很小，但是功能很强大。

![](/fft-2/IMG_6475.JPG)

一枚 INMP441 淘宝的价格在9块钱左右，不贵，多买几个，另外焊接的时候要小心，不要搞反了，像我图中，方形凸起的主体要对着外部，焊脚在另一面，我最开始焊的一个就搞错了。。另外建议买焊接好的成品，自己焊接容易搞坏掉，外围的信号条被破坏了应该会有问题，很考验焊接技术，关键是在你调通电路之前，你自己也不知道坏没坏，不太好检测。

另外注意电压问题，不要给此芯片接入 5v 电源，具体其他限制可以看芯片的官方参数文档。

## 接线
![](/fft-2/IMG_6488.jpg)

INMP 441 共有六个 io 口：
* 其中 VCC 和 GND 是电源引脚，VCC 电压 1.8至3.3 V
* SCK 是 i2s 接口的串行数据时钟引脚。
* SD 是 i2s 接口的串行数据输出引脚。该引脚在未有效驱动相应的输出通道时处于三态。 SD走线应与一个100 kΩ下拉电阻相连，以便在总线上的所有麦克风的输出处于 三态期间，对通道进行放电。
* WS 是 i2s 数据串行数据字选择引脚
* L/R 是左/右通道选择引脚。置于低电平状态时，麦克风通过I 2 S接口左通道输出信号；置 于高电平状态时，麦克风通过右通道输出信号


今天这篇文章就到这里把，关于具体的接线，以及收音的代码，和频谱分析，内容比较多，放在下篇文章展开。

有关麦克风芯片的问题，欢迎探讨。