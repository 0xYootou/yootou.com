---
resources: []
title: 以太坊(Web3)DAPP前端开发要点#2 连接合约和发送交易
pubdate: '2022-02-05 09:00:00'
categories: ["web3"]
featuredImage: /covers/2.jpeg
---

## 概述

如何开发一个以太坊的 dapp？本系列将尽量以比较详细的方式介绍一个 dapp 开发中需要掌握的基本概念和其核心用法。

本系列会不定时更新，有想了解的问题可以在此博文下留言，我会加入到系列文章内容中。本人也接触此领域不久，不保证以下所有做法都是最佳实践。

本系列涉及的合约为了方便大家学习，直接内置了一个私钥、一个账户地址、一个合约地址，请大家不要做破坏性的使用

合约项目地址：[https://github.com/yu-tou/eth-self-token-example](https://github.com/yu-tou/eth-self-token-example)



本文内容大致如下：

1. 如何连接合约？
2. 如何发送交易？
3. 如何使用类型？



## 正文

我们首先分别看两个最基本的连接合约的示例，这是基础方法，在实际的项目中，情况会更为复杂，这里暂时只讲最基本的用法。

本文因为只是最简单的合约连接，所以使用了 nodejs 脚本，没有涉及到真实的第三方钱包以及前端连接合约的用法，这些在稍后的教程中会继续分享，因为要理解在前端页面如何连接钱包，首先要理解如何连接网络节点和发送交易等基础知识，这样才能循序渐进掌握清楚在前端如何连接钱包及请求合约。

### 使用 web3.js 连接合约和发送交易

#### 连接合约

```js
const Web3 = require("web3");
const abi =
  require("../artifacts/contracts/YuTouMintable.sol/YuTouMintable.json").abi;

const config = require("./../config");

// 合约地址，这里我放了一个名叫 YuTou 的代币合约的地址，在 bsctestnet 网络中
const contractAddress = "0xeD0F31xxxxxx";
const account = "0x122xxxxx";
// 创建web3对象
const web3 = new Web3();
// 连接到网络，这里是bsc测试网
web3.setProvider(
  new Web3.providers.HttpProvider(
    "https://data-seed-prebsc-1-s1.binance.org:8545"
  )
);

// 初始化合约对象
const contract = new web3.eth.Contract(abi, contractAddress, {
  from: account,
  gasPrice: "20000000000",
  gas: "1000000",
});

const main = async () => {
  // 查询余额
  const tokenBalance = await contract.methods.balanceOf(account).call();
  console.log("tokenBalance", tokenBalance);
};

main();

```

这里主要是三个步骤：
1. 设置网络对象（provider）
2. 连接自定义合约
3. 查询余额

#### 发送交易

web3js 的版本一直在迭代，一些用法也在发生变化，刚才的代码应该兼容性是比较高的，但是这里只涉及到了最基本的连接合约和合约读取，如果你需要调用会在链上产生更改的方法，在最新的 web3js 中，需要设置一下钱包信息（web3@1.7.0）

以上的合约中有一个使用创世账号，给任意用户发token的操作（mint），我们可以调用一下。

```js

// 初始化合约对象
// 这里的第三个参数，设置此次所有操作默认使用的发送账号和gas信息
const contract = new web3.eth.Contract(abi, contractAddress, {
  from: account,
  gasPrice: "20000000000",
  gas: "1000000",
});


contract.methods
  // 发送 1024 个 YuTou 币
  .mint(account, 1024 * 10 ** 8)
  .send()
  .then(function (receipt) {
    console.log("call result", receipt);
  })
  .catch((error) => {
    console.error( error);
  });

```

和 balanceOf 的调用类似，区别是 balanceOf 是读的接口，mint 是写的接口，读的接口使用 call() 调用，写的接口使用 send() 调用。

不过当你执行以上的代码的时候会报一个错误：Returned error: unknown account 。

这里需要在 web3.js 中执行一个初始化钱包的动作

```js
// 向web3 示例中添加一个钱包账号，这个账号就是 from 的账号，也是合约的部署者，使用私钥或者助记词来导入账号
web3.eth.accounts.wallet.add(config.private_key);

```

此时再调用写的合约即可调用成功，修改调用方法可以给任意地址发送 token，注意这里的私钥必须是创世账号的私钥，只有创世账号才有权限给所有人发 token，这是合约里的限制。

#### 如何获取可以调用的合约方法

刚才我们调用了两个合约的方法：balanceOf() 以及 mint()，这些方法是从哪里来的？如何获取所有可以调用的方法？

两个办法：
1. 看合约源码
2. 看合约编译后的 abi 文件

其实还有第三种方法，不过在 web3.js 中我没有很好的实践过，但是等会在 ethers.js 中可以看到比较好的融合，也就是使用 typechain 编译出完整的 typescript 类型，不过看 typechain 的官方，也支持 web3.js 的类型，见[示例](https://github.com/dethcrypto/TypeChain/tree/master/examples/web3-v1)

这里只简单展示一下 abi 内部的结构，例如刚才的 balanceOf 方法，在 abi 文件中可以看到：

```
{
    "inputs": [
    {
        "internalType": "address",
        "name": "_owner",
        "type": "address"
    }
    ],
    "name": "balanceOf",
    "outputs": [
    {
        "internalType": "uint256",
        "name": "balance",
        "type": "uint256"
    }
    ],
    "stateMutability": "view",
    "type": "function"
},
```
可以看到方法名，输入和输出参数及类型等信息，如果来不及搭建 typechain 的开发工具链，也可以用这种笨拙的方法直接开发。

### 使用 ethers.js 连接合约

#### 连接合约

以下代码见：test/ethers.js 文件

```js
const ethers = require("ethers");
const config = require("../config");

const abi =
  require("../artifacts/contracts/YuTouMintable.sol/YuTouMintable.json").abi;

const provider = new ethers.providers.JsonRpcProvider(config.network.url);

const contract = new ethers.Contract(config.contract_address, abi, provider);

const main = async () => {
  const balanceOf = await contract.balanceOf(config.deployer);
  // 格式化一下真实的token余额
  const realBalanceOf = ethers.utils.formatUnits(balanceOf, 8);
  console.log("realBalanceOf", realBalanceOf);
};

main();

```

可以看到和 web3.js 非常类似，也是三步，其实他们完成的事情是完全一样的。

不过 ethers.js 实例化合约之后，合约方法直接挂到了实例上，另外就是不需要调用 call 或者 send 方法，直接调用合约暴露的方法即可，这种写法其实本质上依然没有变化，但是对于大部分js开发者来说，使用起来会更直观简单一些。

#### 发送交易

我们跟 web3.js 一样，尝试发送一个写的交易。

```

const result = await contract.mint(config.deployer, 1024 * 10 ** 8);
console.log("result", result);

```

依然会报错：

```
sending a transaction requires a signer
```

添加以下代码，同 web3.js 一样，我们需要引入钱包的概念，生成一个成为 signer 的对象。

什么是 signer ，见官方解释：

> A Signer in ethers is an abstraction of an Ethereum Account, which can be used to sign messages and transactions and send signed transactions to the Ethereum Network to execute state changing operations.

我们可以把 provider 直接包装成一个 signer，之后使用这个 provider 实例化的合约对象，即可直接发送交易。

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

### 合约强类型 TypeChain

我通常使用 TypeChain 来编写合约的前端代码，据我了解，社区里大部分人都是使用这个库的。

#### 关于

[项目地址](https://github.com/dethcrypto/TypeChain)

#### 编译

前端开发可以直接忽略这个步骤，可以找合约的开发者要生成后的 TypeChain 文件夹。

因为我们合约是使用 hardhat，所以直接使用一个零配置的库 [@typechain/hardhat](https://github.com/dethcrypto/TypeChain/tree/master/packages/hardhat)

安装依赖后，在编写合约的 hardhat 项目中，直接引入即可：

```js
require('@typechain/hardhat')
require('@nomiclabs/hardhat-ethers')
require('@nomiclabs/hardhat-waffle')
```

引入后，每次编译合约的时候都会生成对应的 TS 文件。

```bash
typechain-types                  
├─ factories                     
│  ├─ ERC20__factory.ts          
│  ├─ Ownable__factory.ts        
│  ├─ YuTouMintable__factory.ts  
│  └─ YuTou__factory.ts          
├─ ERC20.ts                      
├─ Ownable.ts                    
├─ YuTou.ts                      
├─ YuTouMintable.ts              
├─ common.ts                     
├─ hardhat.d.ts                  
└─ index.ts                      
```

#### 使用

在连接合约的代码中（nodejs或者前端项目中）可以直接使用 typechain 生成的类型。

如

```ts
import YuTouMintable from './typechain-types/YuTouMintable.ts'

const contract = new ethers.Contract(config.contract_address, abi, signer) as YuTouMintable;

// 此时 contract 变成了强类型，所有合约中定义的方法，输入输出都定义的非常清楚

```

在 web3.js 中同样可以使用，只需要安装不同的插件即可，不过我没有尝试过，有兴趣可以自己尝试一下。

对于习惯使用 typescript 进行编程的同学来说，使合约具备强类型是非常必要的，希望大家可以将其作为最佳实践在项目中使用和集成。

## 预告

下期，我们将回到前端页面，带大家看一下如何连接钱包和合约。






