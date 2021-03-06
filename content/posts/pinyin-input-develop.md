---
title: 手把手教你写一个全功能全拼输入法
date: 2016-10-24 22:57:00
featuredImage: /pinyin-input-develop/cover.jpeg
author: 芋头
categories: ["web"]
---

时间不多还要陪老婆睡觉，我准备直入主题，本文会手把手教你写一个智能的全拼汉语输入法。

其实是语言无关的，我也不打算把具体的代码放出来，我之前用ios实现过一个，效果很棒，可惜ios写涉及数据结构的代码真的是蛋疼，后来放弃维护了。

好的，一步一步手把手来，不涉及什么算法，也不难理解。

### 第一部分 拼音分解

第一步，用户输入一串拼音，假设是这样的：

woshiyimingshuaibichengxuyuan （预期结果：我是一名帅比程序员）

首先，当然是要把拼音分解开，这一步其实有一些深入优化的点，不过就普通的拼音分解来说，其实一个正则足够了。

```js
[^aoeiuv]?h?[iuv]?(ai|ei|ao|ou|er|ang?|eng?|ong|a|o|e|i|u|ng|n)?
```

不要问我怎么知道这个正则的，这条正则的大概意思就是按照元音辅音的规则对一串拼音进行分解。

到了这一步，我们得到这样一个单音数组：

```js
[‘wo’,’shi’,’yi’,’ming’,’shuai’,’bi’,’cheng’,’xu’,’yuan’]
```

发明拼音的人也是挺屌的，想想觉得很佩服，这里是一些关于拼音的历史介绍  [汉语拼音_互动百科](https://link.zhihu.com/?target=http%3A//www.baike.com/wiki/%25E6%25B1%2589%25E8%25AF%25AD%25E6%258B%25BC%25E9%259F%25B3) 

另外先给大家看看预期看到的候选词列表

![](/pinyin-input-develop/v2-73b9ca6bd1b0f3fff439c6cf719053bc_1440w.png)

![](/pinyin-input-develop/v2-1d8e431e1a59c5ca3ce9f4a0453cc6f2_1440w.png)



### 第二部分 整句查询

先说说，我们用的是怎样一个词库，这个词库其实很简单，大概结构是这样的



![](/pinyin-input-develop/v2-1bd37ea7f41d6bd1c0ab2f04243557d3_1440w.png)

这个词库是网上一个输入法里扒出来的，大概的结构就是里面每个起始单词的词都被分到一个单独的表里，可能是为了提升查询性能。每个表中，主要由两列：code和words，分别就是对应的拼音和汉语。另外，还有一列（count）用来指示优先级，他代表这个词出现的优先级，查询的时候会用这个字段来排序，每次在界面上选词的时候会来更新对应code的count，以此可以记录每个词的优先级，另外，我还会把每个人的输入都发送到服务端，在服务端对这个字段做优化，然后在输入法本地和网络做合并，就是传说中的“云智能”输入法。这个词库的大小会随着使用不断变大，新输入并手动选择的词都会被记录到词库中。



暂且不说这些细节，我们就说，我们如何把一句拼音尽可能精确的转换成一句汉语。

1. 第一步，循环全匹配

顾名思义，就是把整个拼音数组，拼起来，去刚才的sqlite里查询，有没有匹配的结果，这是我们最希望看到的结果，如果完全匹配，那这一部分的运算到这里可以结束了。but，通常是不会匹配滴。

注意接下来的操作：将拼音数组末尾一个元素去掉，继续走上述全匹配逻辑，一直到有匹配结果或者只剩下两个拼音元素为止。此时结束此循环，返回匹配到的部分的拼音和对应的汉字。

例如，因为使用了初始化的词库，所以用此算法，我只提取到了前两个拼音元素的匹配，执行此方法后返回结果：

pinyin: ‘woshi’, word: ‘我是’

2. 第二步，剩余匹配

此时，回到整个拼音分割数组，接下来继续来一个循环，将刚才第一步返回的结果，从拼音数组开始将已经匹配过的元素切割掉，将剩下的数组部分继续扔进第一步的方法，重复此操作，直到耗尽所有的数组元素，此时把所有通过第一步操作返回的数据拼起来，就可以得到一个整句的汉语句子。

此例子，用没有训练过的词库，最终会循环4此，分别返回4次结果：

pinyin: ‘woshi’, word: ‘我是’

pinyin: ‘yiming’, word: ‘一名’

pinyin: ‘shuaibi’, word: ‘帅比’

pinyin: ‘chengxuyuan’, word: ‘程序员’

可能会更多，如果词库里没有帅比这样的内置词的话。

通常这个句子不一定是你想要的最终结果，不过下面还有几步操作会提供一些其他选择，当你每次选择其他结果的时候，都会对词库进行优化，慢慢训练之后，整句输入会越来越智能。

注意，这里用了两次循环，产生了N次数据库查询，在同步运算的语言里一定要记得开个线程来做这个事情，例如ios中，从输入 到 计算 到 渲染，推荐用 dispatch_queue_t + dispatch_async 来实现，亲测效果杠杠的。

### 第三部分 精确词查询

刚才做的这些事情，只是ci为了在输入法提供的选择词列表第一个产生一个全句的选择，可以快速输入。然后接下来，还需要做些基础的查询，把整个候选词列表填充起来。

首先，向候选词列表填充尽可能精确的匹配词。这里又是一个循环过程，用整个拼音数组去数据库匹配词语，当然一般都匹配不到，好的，回来，把拼音数组最后一个元素去掉，再去精确匹配，循环此过程，直到返回了精确的双拼词，将其排在候选词列表第二位，此时还要继续循环，只要有精确匹配的词，就将其塞到候选词列表的后面，一直到所有拼音元素只剩下一个的时候。

此过程的数据变化就不做演示了，太长了。

### 第四部分 模糊词查询

这一步其实只是对整个查询做一个补充，用所有拼音做一次模糊查询，类似于sql的like查询，查询一些最后的备选词作为候选词最后的补充。

### 第五部分 填充单字

此时你获取到一个很不错的候选词列表了，里面第一个元素是整句，第二个元素是全匹配的比较长的词组，第三个元素可能是一个双拼的词，第四个元素是模糊搜索出来的全匹配。

好棒！再来最后一步，总要把候选词列表填充满吧。

用第一个拼音元素，去库里匹配100个结果，用他们把候选词列表填充满，这些单字不要小看，如果输入法不够智能，通常这些单字才是用户想要的输入。

### 第六部分 智能优化

到此，我们的候选词查询基本结束了，接下来，我们需要做一些其他的处理来让输入法更智能。

1. 每个候选词都保留他们的code和汉语，当用户手动选择某个候选词，去库里查询是否有此候选词的全匹配，如果有，对其优先级+1，如果没有，创建此候选词。

2. 每次输入经过用户同意，都同步到远程数据库，这样基于大数据，可以算出用户整体输入习惯，然后将云数据和本地词库合并，做云优化，原理同上。

### 总结

当然，一个实际的输入法中蕴含的远不止这些，我这里主要是讲一个不复杂的算法实现的输入法候选词查询，希望对大家能有用，也能写出一个自己的输入法。

如果需要我提到的很初级的这个词库，可以发邮件给我索取，我的邮箱：xinyu198736@gmail.com

