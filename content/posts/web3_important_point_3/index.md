---
resources: []
title: 以太坊(Web3)DAPP前端开发要点#3 连接钱包和发起交易
pubdate: '2022-02-23 09:00:00'
categories: ["web3"]
draft: true
featuredImage: /covers/11.jpeg
---

## 概述

如何开发一个以太坊的 dapp？本系列将尽量以比较详细的方式介绍一个 dapp 开发中需要掌握的基本概念和其核心用法。

本系列会不定时更新，有想了解的问题可以在此博文下留言，我会加入到系列文章内容中。本人也接触此领域不久，不保证以下所有做法都是最佳实践。

本系列涉及的合约为了方便大家学习，直接内置了一个私钥、一个账户地址、一个合约地址，请大家不要做破坏性的使用

合约项目地址：[https://github.com/yu-tou/eth-self-token-example](https://github.com/yu-tou/eth-self-token-example)

前端项目地址：

## 本文内容

本文为了尽量让用户理解在前端代码中连接钱包和合约的细节，而不只是复刻和使用封装库，所以尽量使用前端框架无关的方式连接钱包和合约。

本文内容大致如下：
1. 为什么需要独立的钱包？ 
2. 准备abi、typechain和网络配置
3. 连接钱包
4. 网络检测
5. 等待连接
6. 获取 provider
7. 获取账号信息
8. 请求合约数据（读
9. 发起交易
10. 等待结果
11. 事件监听

以下话题放在下一篇内容中分享：
1. 其他钱包连接
2. 不用钱包能不能在前端发起交易？
3. 什么时候需要 approve 操作？如何理解 approve 操作？如何优雅处理 approve？
4. gas limit 如何处理？
5. 错误信息如何显示？
6. 如何获取交易记录？


## 正文

### 1. 为什么需要独立的钱包？

经过上一篇文章的介绍，大家应该已经了解了使用 web3.js 或者 ethers.js 连接合约的操作，不过之前的操作都是在 Nodejs 中执行的，理论上只适用于在服务器上运行或者在开发部署阶段使用，正常用户在浏览器中看到的页面中，连接合约的方式还是有一些差异的，本文就是总结这部分内容。

本文依然使用 web3.js 和 ethers.js 这两个库，事实上我们只需要其中之一，我们以 ethers.js 为主，他在等待交易的写法、对 async/await 的支持、ts 类型等方面都更友好一些。

上篇文章，我们在展示发送交易的时候，提到过 web3.js 和 ethers.js 中的 wallet 概念，也就是钱包，不过因为我们是在服务端或者本机的命令行中执行的脚本，所以我们直接在代码中创建了一个 wallet 实例，导入了一个私钥生成了一个账号，但是私钥实际上是非常隐私的信息，不能暴露给任何人，所以在浏览器中，你是不能这样操作的，也就是你不能把私钥写在代码中，也尽量不要让用户在你的网页中填写私钥（实际上用户也不敢这样做，任何让你给出私钥的网页可能都是诈骗），那你就很难在代码中生成一个 wallet 实例，也就无法连接合约发起交易。

回忆一段上篇文章的代码：

```js
const ethers = require("ethers");
const config = require("../config");

const abi =
  require("../artifacts/contracts/YuTouMintable.sol/YuTouMintable.json").abi;

const provider = new ethers.providers.JsonRpcProvider(config.network.url);

// 实例化钱包，使用私钥
const Wallet = ethers.Wallet;
const signer = new ethers.Wallet(config.private_key, provider);

// 第三个参数由 provider 更改为 signer
const contract = new ethers.Contract(config.contract_address, abi, signer);

const main = async () => {
  const balanceOf = await contract.balanceOf(config.deployer);
  // 格式化一下真实的token余额
  const realBalanceOf = ethers.utils.formatUnits(balanceOf, 8);
  console.log("realBalanceOf", realBalanceOf);

  // 此时即可发送交易
  const trans = await contract.mint(config.deployer, 1024 * 10 ** 8);
  console.log("trans", trans);
  // 上一句调用只是获取了交易本身的信息，这里我们继续等待交易返回，交易完全执行之后即可获取到真实的结果
  const result = await trans.wait();
  console.log("result", result);
};

main();

```

实际上，这段代码在浏览器中也是可以跑的，唯一的障碍就是 `new ethers.Wallet(config.private_key, provider)` 这句话中的 private_key ，也就是私钥，私钥不能写在前端代码中，也不能让用户提供给你，所以这时候，我们就需要一个中间者，它的作用是，在应用端，替代代码中的生成 wallet 实例的操作，直接给你提供 wallet 示例，也就是 signer，另外对用户端，它帮助用户导入私钥、管理账号等，因为它是一个中立的、高信任度的、足够安全的中间者，所以用户可以将私钥提供给它管理，但是不暴露给应用或者网站，这样就可以解决上述的问题了。

所以，可以将钱包理解成一个帮助用户管理私钥的工具，当用户选择将钱包连接并授权某个应用的时候，钱包用用户的私钥直接生成了一个 web3.js 的 wallet 实例，注入到当前应用的前端代码中，这时候应用就可以按照上篇文章中的方式连接合约和发起交易了。

这里实际上还涉及到不少概念，如果大家有兴趣了解，可以留言，本文需要分享的内容还是有点多的，这里不再深入探讨钱包是什么了。

### 2. 准备abi、typechain和网络配置

开始开发前，我们有一些内容要提前准备一下。







