+++
title = "Web 体验技术发展前沿速览"
pubdate = "2022-01-21"
featuredImage = "/modern/cover.jpeg"
categories = ["web"]
+++

我最近在听的音乐推荐

<iframe allow="autoplay *; encrypted-media *; fullscreen *" frameborder="0" height="300" style="width:100%;max-width:660px;overflow:hidden;background:transparent;" sandbox="allow-forms allow-popups allow-same-origin allow-scripts allow-storage-access-by-user-activation allow-top-navigation-by-user-activation" src="https://embed.music.apple.com/cn/album/kitaro-the-best-of-ten-years-1976-1986/159667527"></iframe>

小芋头君@ [yootou.com](https://yootou.com)

<p align="center">
    <a href="https://frontend-modern-share.vercel.app/">在线 Slide 地址</a>
</p>
<p align="center">  
    <a href="https://frontend-modern-share.vercel.app" target="_blank">
        <img src="https://img.shields.io/badge/Online%20Slide-on%20Vecel-green?&labelColor=000&style=for-the-badge&logo=vercel" />
    </a>
    <a href="https://github.com/yu-tou/frontend-modern-experience-share-slide">
        <img src="https://img.shields.io/badge/-Github-green?&labelColor=000&style=for-the-badge&logo=github" />    
    </a>
    <a href="https://github.com/yu-tou/frontend-modern-experience-share-slide/discussions" target="_blank">
        <img src="https://img.shields.io/badge/Join%20the%20Discussion-on%20Github-green?&labelColor=000&style=for-the-badge&logo=github" />
    </a>
</p>

### 在线 Slide

<iframe src="https://frontend-modern-share.vercel.app/1" style="width:100%;border:none;height:23.6em;border-radius:8px;box-shadow: rgba(0, 0, 0, 0.1) 0px 5px 5px 0px, rgba(0, 0, 0, 0) 0px 0px 0px 0px;overflow:hidden;background:#fff;"></iframe>

### 声明

<p align="center">本分享内容具有一定局限性，不代表社区真实现状，仅做参考！</p>

 <p align="center">部分内容面向非专业用户（偏产品向）。</p>

### 感谢

<p align="center">
    <a href="https://sli.dev/" target="_blank">
        <img src="https://d33wubrfki0l68.cloudfront.net/9a47dde680cca08e326c07824009ed1adc29626e/6a1c0/logo-title.png" style="width:100px;"/>
    </a>
</p>

## 本文主要内容

- 实时体验（Realtime experience）
- 在线协作（Online collaboration）
- 快速建站（Jamstack）
- 浏览器计算能力
- 浏览器系统能力
- 移动端
- 区块链

## 一 实时体验

### 实时体验



[https://vercel.com](https://vercel.com)



在 2021 年 6 月，Next.js 框架背后的开发商 Vercel 在 C 轮融资中筹集了 1.02 亿美金，使其估值达到 11 亿美元，正式成为了一家独角兽企业。11 月份，Vercel 再次宣布获得新的融资，其在 D 轮融资中筹集了 1.5 亿美元，使其估值相比之前翻了一番，达到 25 亿美元。

- 70 CITIES
- 24B+ REQUESTS PER WEEK
- 10PB DATA SERVED
- 99.99% GUARANTEED UPTIME



部署前端应用的云服务，也支持简单的服务端、存储等。

## vercel



### 核心特色



- 产品定位和产品功能极简，官方定义：Develop. Preview. Ship.
- 部署过程极简，连接 Github 账号、导入 Github 项目、Deploy、Preview，最小步骤完成。
- 在 vercel 里部署应用的各种过程都不需要刷新页面，所有状态都会 <span style="color:#ff0000;">自动推进</span> 和 <span style="color:#ff0000;">实时生效</span>，触发往往只需一个点击

## vercel



 <video src="/modern/1.mp4" controls   />

### 背后技术



#### Next.js

- The React Framework for Production
- hybrid static & server rendering, TypeScript support, smart bundling, route pre-fetching, and more

#### SWR

![](/modern/swr.vercel.app_zh-CN.png)

#### Hyper

- https://hyper.is/
- a beautiful and extensible experience for command-line interface

## 二 在线协作

### 1. figma cursor chat



<img src="/modern/3.gif" >



[https://github.com/yomorun/react-cursor-chat](https://github.com/yomorun/react-cursor-chat)

### 2. figma comments



<img src="/modern/4.gif" >

### 3. figma voice



<img src="/modern/5.png" >

### 3. figma voice



<img src="/modern/5.png" >

### 4. FigJam



<img src="/modern/6.png" >

### 5. Miro



cursor chat & live chat & video chat & screen share & comments & collaborative editing

<video src="/modern/7.mp4" controls    />

### Figma

#### 2021 年 6 月融资 2 亿美金，市值 100 亿美金



### Miro

#### 2022 年 1 月融资 4 亿美金，市值 175 亿美金



有意思的是这些专业工具的一大核心初始卖点就是协同。

只是随着疫情格局变化，这种协同一直在被强化、扩展、创新（cursor chat）。

除了核心功能的协同，这些工具反向把 chat、meet、voice 搬到了工具内。

这么大的市值中，在线办公、在线会议的价值占比不会低，而不是单纯的设计工具或者白板工具。

### 6. Next.js live (vercel)



https://vercel.com/live

Join/Draw/Chat/Code，这里不难理解为什么 vercel 要做 Next.js live，借鉴的是以上几个工具的颠覆思路

<img src="/modern/8.png" >

### 技术框架

<img src="/modern/9.png" style="width:50px;display:block; ">



https://replicache.dev/

Replicache makes it easy to add realtime collaboration, lag-free UI, and offline support to web apps.

<img src="/modern/10.png" >

## 三 快速建站

### Jamstack

[https://jamstack.wtf/](https://jamstack.wtf/)

#### DEFINITION

"JAM" stood for JavaScript / API / Markup。

可以认为 Jamstack 是一组开发网站的最佳实践，包括目标定义、最佳实践、工作流、实用工具等。一些云产品或者框架，遵循类似的理念实现。

#### MEANING

Decoupled 前后端解耦 / Static-first 静态内容优先 / Progressively enhanced 渐进增强

#### BEST PRACTICES

CDN 网络分发 / 原子化部署 / 缓存自动失效 / 版本管理 / 自动化构建

#### WORKFLOW

Develop / Version Control / Automated build( assets. pre render. deploy.) / Update CDN.

### DEVELOPMENT

#### The Fullstack React Framework

- 基本、纯粹、场景无关的底层框架。
- Next.js。可能是使用最广泛的 Fullstack 框架，主要是其 SSR&SSG 能力，0 配置，及 Vercel 的无缝官方支持。
- Remix.js。知名团队最新出品的框架，主要是更强的前后端代码融合能力（同一个组件文件中），更好的加载体验（组件级别的 SSG）。

#### Web Application Framework

- 通常是一系列基础工具的整合，例如 GraphQL、Prisma 等，将数据定义和服务开发等部分 less 掉。
- Redwood.js。an opinionated, full-stack, serverless-ready web application framework。
- Blitz.js。a batteries-included framework that features a "Zero-API" data layer abstraction that eliminates the need for REST/GraphQL

#### Static Site Generators

- 主要用来建站、博客的一些工具框架，例如管理文章、主题、构建等，通常用来写博客、写文档等。
- Gatsby / Hugo / Jekyll / VuePress / Docsify / GitBook 等

#### Site Builders

- 低代码搭建工具，如 Stackbit / Builder.io / CloudCannon

### DYNAMIC PARTS

#### 函数服务

- AWS lambda functions / Netlify Functions / Vercel Functions

#### 数据管理

- Prisma / AirTable / Fauna / Hasura / MongoDB Atlas / AWS DynamoDB

#### 表单工具

- AirTable / Netlify Forms / Getform / FormKeep

#### 评论工具

- walinejs / Staticman / Disqus

#### 商品交易

- Shopify / Snipcart / Commerce Layer

#### 搜索服务

- Algolia / fuse.js / Lunr.js

### DEPLOY

- Netlify，2020 年 5300 万 C 轮
- Vercel，2021 年 1.5 亿美金 D 轮，估值 25 亿美金
- Github Pages
- Fly.io
- Digital Ocean
- Azure Static Web Apps

### 简单总结

- 利用一系列工具组合，快速构建和部署自己的网站。
- 在国外是一个很大的市场，在国内基本不存在这些场景，国内同样的场景是给每个小程序平台写小程序，各平台有自己的部署、函数、存储。。
- 国外社区这种整体架构分解、细分领域发力、灵活重组、生态上互相支持，共赢且推动社区向同一个方向发展的做法值得深思。
- 很多公司和产品以 Jamstack 为核心理念推动和宣导。

## 四 浏览器计算能力

- 多线程（上一代）
- WebGL（上一代）
- WebAssembly
- WebCodecs
- WebGPU

### 上一代技术

#### Web Workers

- 不阻塞主线程的计算进程，不可操作 UI
- 工程化困难，使用比较别扭，和其他 Native 语言操作多线程的体验差距比较大
- 初始化和传输数据开销比较大，非必要不要用，可能会让应用变得更慢

#### WebGL

- 在我们的地理可视化产品中，一个界面上渲染 几十万个可交互的点不卡顿基本没有问题，所以 WebGL 的性能渲染普通的图形是够用的。
- 始于 2006 年，2011 年 WebGL 1.0 标准，2017 年发布 2.0 标准。
- Apple 直到 2020 年才支持 WebGL 2.0。
- 几乎现在所有的操作系统都不再把 OpenGL 作为首要支持。
- WebGL 至今不可以做并行计算。

### WebAssembly

- 是一种低级的类汇编语言，具有紧凑的二进制格式，可以接近原生的性能运行，并为诸如 C / C ++等语言提供一个编译目标，以便它们可以在 Web 上运行
- 所以可以用来做音视频编解码（例如将 FFmpeg 编译到 wasm，然后运行于浏览器中，然而现实情况是需要对 FFmpeg 大量裁剪，并且因为逻辑比较重，效果并不理想）
- 可以用来做科学计算并行计算（WebAssembly SIMD 是最新的 Wasm 技术，它可以实现数据层面的并行处理）
- 可以用来做图形处理等（如 figma 大量使用 wasm 处理设计稿的渲染和特效等）
- 支持诸多语言编译，c/c++、rust、kotlin、swift、c# 等

[Google Meet 中使用 wasm 来实现高效的背景实时替换](https://cloud.tencent.com/developer/article/1745435)
[Chrome 91 支持 WebAssembly SIMD，加速 Web 在 AI 等领域的应用](https://mp.weixin.qq.com/s/P0opB_IMHoAy7PpJH0t-4A)

### WebCodecs

- 可使用底层硬件加速编解码
- wasm 实现编解码过程太复杂，使用复杂

[WebCodecs 对音视频进行编码解码](https://juejin.cn/post/6885482349098860558)

### WebGPU

此 demo 只能在 Chrome Canary 中开启 WebGPU 后使用

[https://playground.babylonjs.com/?webgpu#YX6IB8#36]()

### WebGPU



- [硬件级] WebGPU 是一个精确的图形 API，它完全开放了整个显卡能力，你要向显卡发送命令去控制显卡，它不再是画东西的一个库，而是一个利用 GPU 的库。
- [跨平台] WebGPU 就像一个 HAL 硬件抽象层一样，我只要对应这套标准，未来不仅可以在网页运行，甚至也可以在嵌入式、在服务器运行
- [通用计算] WebCPU 终于开始支持 GPU Compute Shader，并且是把 GPU 通用计算作为首要支持
- [着色器] 新的着色器语言 WGSL，支持面向对象编程
- [性能差距] babylonjs 示例，性能差距 6 倍以上。



[从 WebGL 到 WebGPU，网页图形的全新时代](https://mp.weixin.qq.com/s/4LfaNHP77s9n9SghucYoaA)

## 五 浏览器系统能力

### 浏览器系统能力

- 音视频。MediaStream(音视频流) / Media Source Extensions（程序媒体流输入）/ MediaStream Recording(流捕获) / MediaStream Image Capture / Capabilities（编解码探测）/ Web Audio API（音频处理、分析、效果等） / Web Speech API（语音识别和合成） / MIDI API（音乐合成）/ WebCodecs （音视频编解码）/ WebVTT（字幕）
- 桌面。Screen Capture（屏幕捕捉）/ Picture-in-Picture（画中画） / EyeDropper(拾色) / Notifications / Page Visibility / Screen Wake Lock （阻止休眠） / VisualViewport（可见窗口探测）/ Window Controls Overlay（控制标题栏）
- 存储。File System Access API / IndexedDB（大数据存储） /
- 网络。Streams API（按位读取写入）/ WebRTC API（点对点通信）
- 外设。 WebAR / WebXR / GamePad / Battery Status API / Bluetooth API / Presentation API（演示屏幕控制）/
- 传感器。Sensor（三轴传感器）/ Geolocation （位置） / Screen Orientation （屏幕旋转） / Vibration（震动传感器）
- 脱机。Service Worker API(离线) / Background Fetch API（页面关闭后仍可请求数据） / Background Synchronization（背景同步资源） /
- 虚拟设备。Payment Request API（信用卡支付）/ Clipboard API / Pointer Lock（鼠标锁定）
- 内容。Encoding API / Intersection Observer API(内容探测) /
- 性能探测。Performance API（延时探测） / Navigation Timing（页面跳转性能探测） / Long Tasks API（高损耗探测）/ Resource Timing（资源加载探测）

### 注意点



- 部分功能需要 https 环境，特别是涉及到隐私和安全的。
- 部分功能需要用户确认，通过 permissions api 管理。
- 以上提到的大部分功能都在 90 之前版本可用，极少数是最新的规范

### Google Meeting

<video src="/modern/11.mp4" controls    />

## 五 移动端

### flutter

- [flutter 2.8 更新](https://medium.com/flutter/announcing-flutter-2-8-31d2cb7e19f5)
- 支持 Mobile / Web / Desktop（windows/linux/mac） / Embedded App
- 开始集成越来越多 Google 自己的服务（和浏览器一个套路）
- 发布 flame 游戏框架

### iOS

- Xcode Cloud，持续集成，在线构建和分发
- iPad Swift Playground 可以直接编译成 app
- Live Text / SharePlay / RealityKit / ARKit /

## 六 区块链

- web3.js ethers.js web3modal.js ...
- 不展开说了

## More

欢迎 PR 补充内容

[GitHub 链接](https://github.com/yu-tou/web-front-end-technology-forward-share-slide)
