+++
resources = []
title = "Arduino LED 频谱仪 #1 arduino 芯片选择"
mainContent = ""
pubdate = "2022-01-05"
featuredImage = "/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6479.JPG"
+++

本系列文章将带着大家一起简单了解如何制作一个音频频谱显示的 arduino 小作品。

最终效果：
{{< bilibili BV1CY411a7aP >}}

此小产品涉及到以下功能：

1. 音频采集
2. 音频傅立叶变化
3. fastled 驱动 led 板
4. 时间显示及其他动效
5. 亮度和灵敏度调节
6. 亚克力盒子制作

主要涉及以下芯片：

1. arduino 芯片，我们选择 esp32。
2. 音频传感器，
3. WS2812B 8\*32 芯片
4. 2 位数码管
5. INMP441 全向麦克风
6. 旋转编码器和旋钮盖
7. 面包板和杜邦线、插线柱等等若干
8. 电烙铁和焊锡
9. 亚克力板 3mm 以上厚度若干，立柱若干、半透亚克力板 1mm 若干

稍后我会一一介绍为什么选择这些元件以及如何购买比较划算。

以下是半成品

![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6473.JPG)
![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6474.JPG)
![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6475.JPG)
![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6476.JPG)
![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6477.JPG)
![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6478.JPG)
![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6479.JPG)

## 第一部分 arduino 芯片选择

Arduino 芯片有很多类型，一般入门用的都是 uno，功能简单，但是便宜，不过我一般不使用 arduino uno，主要是 uno 主频很低，处理稍微复杂一些的场景就性能不足，另外就是也没有联网之类的功能，另外也缺少一些端口，我之前做物联网应用的时候，用的是 esp8266 芯片，其技术参数见：

https://www.espressif.com/sites/default/files/documentation/esp8266-technical_reference_cn.pdf

- 单核处理器，运行频率为 80MHz
- 802.11 b/g/n
- Wi-Fi Direct (P2P)、soft-AP
- 内置 TCP/IP 协议栈
- 内置 TR 开关、balun、LNA、功率放大器和匹配网络
- 内置 PLL、稳压器和电源管理组件
- 802.11b 模式下+19.5dBm 的输出功率
- 内置温度传感器
- 支持天线分集
- 断电泄露电流小于 10uA
- 内置低功率 32 位 CPU
- SDIO 2.0、 SPI、UART
- STBC、1x1 MIMO、2x1 MIMO
- A-MPDU 、A-MSDU 的聚合和 0.4s 的保护间隔
- 2ms 之内唤醒、连接并传递数据包
- 待机状态消耗功率小于 1.0mW (DTIM3)

主要是因为 esp8266 兼容 arduino，然后 cpu 频率比较高，另外就是特别便宜，同时还支持 wifi。

在购物网站上只需要七八块钱就能买到一个可以直接用的板子，的确是非常划算了。

不过本次实验用的不是 8266，而是他的升级版 esp 32，其实这也是一个应用非常多的芯片，他的优势主要是 cpu 主频高，接口方面更丰富。

本次实验涉及到音频采样，如果要做 44100 hz 的采样的话，需要输入的频率是 =2× 采样频率 × 采样位数 = 2 * 44100 *32 = 2.8 mhz，这个是这个实验对 arduino
板子的最低要求，而且我们这里还会涉及到一个非常复杂的操作，也就是快速傅里叶变换，一般的 arduino cpu 频率很难支撑，而且我担心 esp 8266 也心有力而余不足。

另外，从麦克风传感器中获取数据，有两种方式，adc 方式，和 i2s 方式，adc 是利用 arduino io 口的输入能力定时采样，io 口的采样频率非常有限，很难完整完整的采样，所以我们需要 i2s 接口，他的采样是异步的，带缓冲，而且和 io 口是独立的，不会影响 io 口的其他用途。

具备 i2s 的 arduino 芯片是比较少的，如 arduino due，官方价格 300 多一片，而且仿品很少。不过 esp32 这个芯片就厉害了，他支持 i2s 通信协议，同时频率也非常高，主频可达 240Mhz，而且部分型号是双核 MCU。

![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/esp32.png)

esp32 官方中文文档：

https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_cn.pdf

esp32 开发板的价格通常在 20 元左右，可以多买几片，因为这种成本低的芯片，有一定概率是坏片，不过通常概率也不会很高。

![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6482.JPG)

这是我用过的一些 arduino 芯片，带 wifi 的基本都是 esp8266，然后还有 arduino uno 入门款。

另外在闲鱼上淘了一篇 due ，贵的东西做工就是不一样，金光闪闪的，不过这个芯片还从来没用上过。

![](/Arduino%20%E9%9F%B3%E9%A2%91%E7%9B%92%E5%AD%90%E5%88%B6%E4%BD%9C%20%231%20arduino%20%E8%8A%AF%E7%89%87%E9%80%89%E6%8B%A9/IMG_6484.JPG)

## 交流

关于 arduino 各种型号的芯片选择，大家有场景可以留言，我可以给出自己的意见，因为 arduino 型号太多了，没法一篇文章说清楚。
