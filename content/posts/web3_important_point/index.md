---
lastmod: '2022-01-10T16:55:06.811Z'
title: 以太坊(Web3)DAPP前端开发要点#1 js库选择对比
pubdate: '2022-01-09'
categories: ["web3"]
featuredImage: /eth_important/cover.jpeg
---


## 概述

如何开发一个以太坊的 dapp？本系列将尽量以比较详细的方式介绍一个 dapp 开发中需要掌握的基本概念和其核心用法。

本系列会不定时更新，有想了解的问题可以在此博文下留言，我会加入到系列文章内容中。本人也接触此领域不久，不保证以下所有做法都是最佳实践。


大概以下几个方面：
1. 如何选择连接智能合约的js库？ 
1. 如何连接钱包以及获取钱包基本信息？
2. 如何实例化合约？
3. 如何在ts中编写强类型合约调用代码？
4. 如何发起交易？
5. 如何处理大数单位？
6. 何时需要 approve？
7. 发起交易时如何费用估算？
8. 如何获取交易结果？
9. 如何拉取交易记录？



## 如何选择js库？

连接智能合约，比较基本的可以使用 web3.js，中文文档：[https://web3.tryblockchain.org/](https://web3.tryblockchain.org/)

这个库不止是在前端可以使用，在 nodejs 脚本中也可以使用，所以一些工具会使用 web3.js 来部署合约等，它的能力还有很多，前端调用合约，发起交易只是它非常小的一部分，不过如果纯粹作为前端，实际上不需要对其他概念了解，了解调用合约、发起交易、连接钱包等就可以完成 dapp 的开发。

不过按照我的了解，web3.js 比较底层，覆盖功能比较广，调用起来比较繁琐，所以我都是使用 ethers.js 的，官方文档：[https://web3.tryblockchain.org/](https://web3.tryblockchain.org/)

ethers.js 主要是提供了一些钱包方法，然后提供了一些工具方法（例如处理大数转换和单位格式化等），另外就是调用方式更加友好。

ethers.js 的合约调用写法更纯粹，也更简单，特别是一些事件监听之类的，配合 typechain 强类型使用也更方便，建议初学者使用 ethers.js 来连接合约。

ethers 官方定义：

> ethers.js 库旨在成为一个完整而紧凑的库，用于与以太坊区块链及其生态系统进行交互。它最初是为与ethers.io一起使用而设计的，后来扩展为一个更通用的库。

特征

- 保持你的私钥在客户端，安全和声音
- 导入和导出JSON 钱包（Geth、Parity 和众筹）
- 导入导出 BIP 39 个助记词（12 个单词备份词组）和高清钱包（英文、意大利文、日文、韩文、简体中文、繁体中文；更多即将推出）
- 元类从任何合约 ABI 创建 JavaScript 对象，包括ABIv2和Human-Readable ABI
- 通过JSON-RPC、INFURA、Etherscan、Alchemy、Cloudflare或MetaMask连接到以太坊节点。
- ENS 名字是一等公民；它们可以在任何可以使用以太坊地址的地方使用
- 小（~88kb 压缩；284kb 未压缩）
- 满足您所有以太坊需求的完整功能
- 广泛的文档
- 大量的测试用例被维护和添加到
- 完全准备好TypeScript，带有定义文件和完整的 TypeScript 源
- MIT 许可证（包括所有依赖项）；完全开源，随心所欲

ethers 与 web3.js 相比的主要优势是：

1. ENS name 是一等公民

在 ethers.js 中，您可以使用 ENS name 代替合约地址——这是 web3 中当前不可用的功能。

这在许多情况下都很有用。例如，编写需要调用 ENS 解析器的应用程序的开发人员可以简单地使用“resolver.eth”，而不是输入实际的合约地址（“0x5FfC014343cd971B7eb70732021E26C35B744cc4”）。如果解析器的实际合约地址被更新，则代码为“resolver.eth”的应用程序不需要更新，因为该 ENS 名称的所有者会重定向它（假设你信任他们！）。使用原始合约地址的应用程序必须更新该地址。

2. 很小。Ethers.js压缩后只有77kb。

3. 关注点分离——密钥管理和状态，

ethers.js 和 web3 之间的一个主要区别是它们如何处理密钥管理以及与以太坊区块链的交互。Web3 假设有一个本地节点连接到应用程序。假设该节点存储密钥、签署交易以及与以太坊区块链交互和读取。实际上，这种情况并不常见——大多数用户并没有在本地运行 geth。Metamask 通过浏览器应用程序有效地模拟了该环境，因此大多数 web3 应用程序都需要 Metamask 来保存密钥、签署交易并与以太坊主网交互。

Ethers.js 采用了一种不同的方法，我们认为这可以为开发人员提供更大的灵活性。Ethers.js 将“节点”分成两个不同的角色：
- 一个持有钥匙并签署交易的“钱包”，以及
- 一个“提供者”，充当以太坊网络的匿名连接，检查状态并发送交易

这允许开发人员分别管理这两个角色——也许使用 Infura 作为提供者，以及用于钱包的硬件设备。

4. 丰富的文档

5. 大量的测试用例

6. Ethers.js 完全是 MIT 许可的，包括所有依赖项

## 对比示例

在这个 github 仓库中可以比较直观感受到 web3.js 和 ethers.js 在具体的使用中的差别：

https://github.com/adrianmcli/web3-vs-ethers

这篇文章，有详细的对比介绍

https://blog.infura.io/ethereum-javascript-libraries-web3js-ethersjs-nov2021/

主要对比了以下几个方面：

1. How important is the popularity of the library?
2. How important is the maintenance of the library?
3. Who is responsible for the development of the library and how many projects use it?
4. How important is it that the library has tests?
5. How important is the number of downloads of the library?
6. How important is web performance?
7. How important is the quality of the documentation?
8. How important is the overall usage of the library?
9. How important is the license?
10. How important is the community and community experience?

分别是：

1. library的受欢迎程度有多重要？
2. library的维护有多重要？
3.  谁负责这个库的开发，有多少项目使用它？
4. library有测试有多重要？
5.  库的下载次数有多重要？
6.  网络性能有多重要？
7.  文档的质量有多重要？
8. library的整体使用有多重要？
9.  许可证有多重要？
10.  社区和社区经验有多重要？

## 预告

下篇文章，我们将介绍如何连接合约，以及发送交易