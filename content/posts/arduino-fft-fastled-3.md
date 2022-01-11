+++
resources = []
title = "Arduino LED 频谱仪 #3 LED选择和FastLED"
mainContent = ""
pubdate = "2022-01-11"
featuredImage = "/fft-3/2.png"
+++
## 关于本系列

本系列文章将带着大家一起简单了解如何制作一个音频频谱显示的 arduino 小作品。

本节将介绍 LED 的选择及其驱动和注意事项。

前续文章：
* [#1 arduino 芯片选择](../arduino-fft-fastled/)
* [#2 音频传感器选择](../fft-2/)

## 视频

元旦跨年家庭小 Party 的时候，这个小玩意发挥了很大的作用，一群朋友听着音乐跳舞跨年，我则打开 LED 频谱仪，给大家来了一个现场 DJ。

{{< bilibili BV1QY411875p >}}

以下是成品视频，这系列文章会比较长，感兴趣的小朋友可以收藏我的博客，然后慢慢追更。

{{< bilibili BV1sT4y1m7ji >}}

## LED 序列的选择

在 LED 选择这块，我也趟过坑，例如想要自己用最原始的 LED 小灯泡自己组线，但是这样的话，会有三个问题：
1. 涉及到模拟电路，这块需要一些电路知识，基本忘光了，例如需要搭配电阻和电容。
2. 彩色 LED 涉及到编码驱动，特别是 LED 多了之后，太麻烦了。
3. LED 阵列的灯泡数量很多，例如一个 8*32 的阵列，需要 256 个灯泡，每个灯泡需要正负极，串联或者并联，连线会非常复杂。

所以，一般做电路搭建实验，不会使用原始器件，例如麦克风也一样，不会直接使用 咪头，因为他们需要搭配周边电路，还需要访大和滤波等，成本和复杂度增高。

后来我又研究了一下，首先发现 arduino 社区常用的 驱动 RGB LED 的库，发现了一个名叫 FastLed 的库。

https://fastled.io/

这个库看起来功能很强大，于是顺着这个库去了解常见的 RGB LED 的驱动芯片是什么。

查看其 github 描述，有以下一段话：

> This is a library for easily & efficiently controlling a wide variety of LED chipsets, like the ones sold by adafruit (Neopixel, DotStar, LPD8806), Sparkfun (WS2801), and aliexpress

于是我把这些芯片都去搜了一遍，后来基本锁定 ws2801，因为我发现淘宝上卖的 led 成品基本都是 ws2801 驱动的。

确定驱动之后，就开始采购用于本制作的 LED 阵列了。

我的需求如下：

1. 需要显示频谱，所以阵列宽度至少8位，否则很难表现振幅的变化。
2. 需要显示时间（时分秒），总共 6个数字+2个冒号，每个数字至少需要 3*5 才能区分开，加上每个数字之间的空位，也就是宽度至少 6*4+2*2 = 28。
3. 如果时间再加上边框，可以发现 8*32 阵列基本够用了。

8*32 其实是比较常见的阵列，就是4个8*8拼在了一起，这里我们最好采购一体的阵列，而不是买四块自己去拼，这样会多出一些连线，还有缝隙，效果不完美。

![](/fft-3/1.jpg) 

淘宝上常见的是这款，我用的就是这款，因为是普通 led 灯珠，这款阵列在亮度很高的时候，颜色区分可能没你想象的那么明显，其他方面都没有太大问题，不过这款阵列的价格一直在涨，现在大概60元左右一张。

![](/fft-3/2.png)

## 亚克力遮光板

实际上大家从我视频里看到的 led 效果会稍微好一些，光线比较柔和，其实我在led阵列前面盖了一块半透的遮光亚克力，可以把光线变得柔和（2mm磨砂亚克力，30cm*40cm，大概10块钱一张）

亚克力是用亚克力切割刀切割的，另外胶水使用无痕胶水（ergo8500无影uv胶，30元左右一瓶，送紫外灯）。

后续会专门开一篇讲亚克力加工。

## ws2801 阵列连线

购买 8*32 led 阵列后，连线其实很简单，阵列总共有3根输入线，其中一个正极一个负极，真正的信号线只有一根，只需要直接连接到 arduino 芯片的一个输出口即可，需要其他周边电路。

但是请注意，led阵列的耗电量是比较大的，对电流的要求也比较高，所以它也会产生比较严重的电路干扰，所以需要将 **led阵列的电源和arduino芯片及其周边芯片的电路隔离** ，否则可能会产生意外的后果，例如音频采集无法工作，甚至esp32 无法正常工作等。

## fastled 及 驱动代码

fastled 是一个通用的库，封装了很多向 led 阵列输出某个位置的某个颜色的方法，并且封装了很多工具类，例如使用html color code 设置颜色，向阵列写入颜色，批量写入和清空等等。

这里不详细介绍其功能，只介绍一下基本的使用和注意要点。

### 基本使用

```c++
#include <FastLED.h>

// 只需要一个 io 口即可驱动
#define LED_PIN 23

#define LED_TYPE WS2812B
#define COLOR_ORDER GRB

// 定义 led 灯珠数量
#define kMatrixWidth 32
#define kMatrixHeight 8 
#define NUM_LEDS (kMatrixWidth * kMatrixHeight)

// 定义 led 灯珠数组

CRGB leds[256] = {};

// 初始化阵列
void initLed()
{
    LEDS.addLeds<LED_TYPE, LED_PIN, COLOR_ORDER>(leds, NUM_LEDS);
    LEDS.setBrightness(BRIGHTNESS);
    pinMode(LED_PIN, OUTPUT);
    Serial.println("led ready");
    Serial.println(sizeof(leds));
}
// 点亮阵列
void ledShow()
{
    LEDS.show();
}

void setup()
{
  initLed();
  setAllBlack();
  ledShow();
}

int index = 0;
void loop()
{
  index++;
  if(index > NUM_LEDS) index = 0;
  // fastled 的工具方法，填充所有灯珠为黑色
  fill_solid(leds, NUM_LEDS, CRGB::Black);
  // 设置当前灯珠为 #C151FB 
  leds[index].setColorCode(0xC151FB);
  // 输送到芯片
  ledShow();
  delay(1000);
}

```

### 排列和阵列定位

以上代码的效果是从第一个灯珠开始，分别点亮阵列中的每个灯珠，这里你需要做个记录，观察一下你的 led 阵列的排列方式，这很重要，会决定你之后如何在一个二维阵列中定位你要点亮的那个灯珠是第几个灯珠，因为 led 阵列有很多种排列。

例如：

```
0 >  1 >  2 >  3 >  4
                    |
.----<----<----<----'
|
5 >  6 >  7 >  8 >  9
                    |
.----<----<----<----'
|
10 > 11 > 12 > 13 > 14
                    |
.----<----<----<----'
|
15 > 16 > 17 > 18 > 19
```
或者

```
0 >  1 >  2 >  3 >  4
                    |
                    |
9 <  8 <  7 <  6 <  5
|
|
10 > 11 > 12 > 13 > 14
                   |
                   |
19 < 18 < 17 < 16 < 15
```

注意分辨，这是两种完全不一样的排列方式，后续你需要写一个函数，输入二维阵列中的位置，根据你观察到的板子的排列方式，计算出其对应的一维的 index。

```c++
// 如我买的阵列的转换方法
uint16_t XY(uint8_t x, uint8_t y)
{
    int xStart = ((x + 1) / 2) * 16 + (x + 1) % 2;
    int realIndex = 0;
    if ((x + 1) % 2 == 0)
    {
        realIndex = xStart - y;
    }
    else
    {
        realIndex = xStart + y;
    }
    return realIndex - 1;
}
```

这样，传入行和列的x、y即可定位到一个具体的灯珠，因为我们平常操作阵列都是按照二维数组的方式操作的，经常涉及到某个整行或者某个整列，所以需要定义这个方法。

定义之后，设置某个位置的灯珠的位置代码：

```c++
leds[XY(x, y)].setColorCode(code);
```

## 彩虹色

看开头的视频，我们的频谱和时间跑马灯都是彩虹序列的，这里我们定义一个彩虹色的数组，供后续类似的功能使用，具体跑马灯等代码等之后的章节介绍。

```c++
uint32_t LED_COLORS[32] = {
     0x4AF7A2,
     0x57FDD8,
     0x5CEEF9,
     0x43BCFA,
     0x4992F8,
     0x4567EE,
     0x4C44F1,
     0x854EFB,
     0xC151FB,
     0xDE47F1,
     0xEA4CD4,
     0xE33998,
     0xF53876,
     0xFE4A59,
     0xF7694B,
     0xFA9A52,
     0xFDC54E,
     0xFDC650,
     0xFF9D53,
     0xFE6E4E,
     0xFF4855,
     0xFA3E4E,
     0xF42E70,
     0xF644DA,
     0xF54DDF,
     0xBE50FF,
     0x884CFA,
     0x3D3BED,
     0x2D53F0,
     0x4A62F9,
     0x498DF5,
     0x47BAF8};
```

这个彩虹色还是比较好看的，主要是蓝色绿色红色的渐变连续色。

后续点亮频谱和时间显示的代码，之后介绍到相应的地方我们再展示，今天对 fastled 的介绍就先到这里啦，大家有问题可以在文章下方评论，希望多多探讨。




