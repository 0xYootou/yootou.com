+++
title = "pkg 打包 Nodejs native addons 的 bug"
pubdate = "2022-01-07"
featuredImage = "/pkg/cover.jpeg"
categories = ["web"]
+++

## 问题

先说结论：pkg@5.5.1 和 pkg@5.4.1 打包出来的可执行文件，如果包含 native addons，可能会在某些版本的 centos 中报错。

原因是 pkg 本身的bug，和系统有一定关系，但是关系不大，而且此问题出现概率较高，等待官方修复。

## 什么是 pkg

最近在公司的产品里尝试使用了 prisma + nestjs + graphql 的选型，涉及到将这个应用部署到用户环境的问题，我们公司的产品比较特殊，虽然不需要给客户定制，但是部署环境经常需要部署到客户的内网，所以应用的部署环境标准化是非常迫切的需求。

在用户私网环境内部署，最重要要解决的问题有几个：
1. 环境的复杂性，可能需要兼容各种系统，因为可能需要和客户其他服务共享机器。
2. 机器可能和外网不通，不能方便地安装比较复杂的环境。

所以保持部署的一致性和简单化非常重要，我们现在通常都是将应用、nginx、系统软件等打包成一个标准的 rpm 安装包，客户处的运维一键安装和升级。

而之前在公司里使用 Nodejs，通常都是在机器上安装 Nodejs 运行环境，然后再运行应用代码。

集中部署的产品这样没什么问题，但是分散部署的情况下，最基本要做到的一点就是不依赖机器上的系统环境，包括 Nodejs 运行时这样的环境。

而 Nodejs 中是有这样的解决方案的，即将应用本身和 Nodejs 运行时、一些静态资源等一起打入一个二进制的可执行文件，在任意机器上直接运行此可执行文件即可运行应用，不需要安装 Nodejs 运行环境，而且这样分发出去的代码是二进制的，基本无法还原，不容易暴露源码。

打包的工具叫 pkg，github 地址：[https://github.com/vercel/pkg](https://github.com/vercel/pkg)

这个工具本身就不过多介绍了，使用起来很简单，直接执行命令即可将项目内指定的文件打包。

```bash
pkg package.json --out-path bin/ --targets node16-linux-x64   --public --public-packages '*'
```

## native addons

这里只想探讨下如果 Nodejs 应用依赖的包里有 native addosn 的话需要怎样处理，在我们应用中即存在一个库：prisma，他的 query engine 是用 rust 编译出的 native addons，而且不同的 system platform 会使用不同的 addons。

首先，我们看看 pkg 官方如何描述他们对 native addon 的处理的。

> Native addons (.node files) use is supported. When pkg encounters a .node file in a require call, it will package this like an asset. In some cases (like with the bindings package), the module path is generated dynamicaly and pkg won't be able to detect it. In this case, you should add the .node file directly in the assets field in package.json.

> The way Node.js requires native addon is different from a classic JS file. It needs to have a file on disk to load it, but pkg only generates one file. To circumvent this, pkg will create a temporary file on the disk. These files will stay on the disk after the process has exited and will be used again on the next process launch.

> When a package, that contains a native module, is being installed, the native module is compiled against current system-wide Node.js version. Then, when you compile your project with pkg, pay attention to --target option. You should specify the same Node.js version as your system-wide Node.js to make compiled executable compatible with .node files.

其实这里的意思是，不需要特殊处理，正常依赖的 native addons 包不需要特殊处理即可被包含到最终打包的文件中，另外在运行的时候，会将这些文件解到系统的tmp目录中（例如 /tmp/pkg/xxx，其他文件都不会做这个处理）

不过我还是在这个事情上折腾了很久，主要是因为打包出来的包，在 ubuntu x64 上运行正常，系统 /tmp/pkg 目录中也有对应的 addons 文件，但是将文件放在 centos 中运行，却报错：

```bash
spawn /snapshot/xxx/node_modules/.prisma/client/query-engine-rhel-openssl-1.1.x ENOENT
```

也就是找不到 addons 文件。而且进入 /tmp/pkg 文件夹里也找不到 addons 文件。

开始我以为是 prisma 本身的问题，但是因为这种问题不太好定位，我也做不了什么，只能修改 prisma 和 pkg 的各种参数做尝试。另外就是查阅官方文档，不过文档中的描述，按照道理来说不应该有这个问题。

不过 pkg 这个包虽然现在是大厂在维护（vercel），但是之前我就遇到过一个版本的问题，也就是最新版包打出来的文件有问题，回退一个版本就ok的现象，不过我不记得具体问题是什么了。

于是我就尝试了一下，把 pkg 版本往回推，结果推了两个版本之后，打出来的包，在 ubuntu 和 centos 的各种版本上都正常了。

最终基本确认是最近几个版本(@5.5.1 & @5.4.1) pkg 包引入的一个 bug，我已经给 vercel 提了 issues，有回复了我会在博客中同步。

有遇到同样问题的同学可以尝试回滚一下 pkg。



