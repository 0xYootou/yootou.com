---
title: JS 装饰器（Decorator）场景实战
date: 2017-10-26 22:36:00
featuredImage: js-decorator-code/cover.jpeg
author: 芋头
categories: ["技术"]
tags: ["Decorator", "JavaScript"]
---

本文不会大篇幅介绍装饰器（Decorator）的概念和基础用法，核心介绍我们团队如何将装饰器应用于实际开发，和一些高级用法的实现。

## 装饰器简介
Decorator 是 ES7 的一个新语法，正如其“装饰器”的叫法所表达的，他可以对一些对象进行装饰包装然后返回一个被包装过的对象，可以装饰的对象包括：类，属性，方法等。Decorator 的写法与 Java 里的注解（Annotation）非常类似，但是一定不要把 JS 中的装饰器叫做是“注解”，因为这两者的原理和实现的功能还是有所区别的，在 Java 中，注解主要是对某个对象进行标注，然后在运行时或者编译时，可以通过例如反射这样的机制拿到被标注的对象，对其进行一些逻辑包装。而 Decorator 的原理和作用则更为简单，就是包装对象，然后返回一个新的对象描述（descriptor），其作用也非常单一简单，基本上就是获取包装对象的宿主、键值几个有限的信息。

关于 Decorator 的详细介绍参见文章： [zhuanlan.zhihu.com/FrontendMag…](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2FFrontendMagazine%2F20139834) 

简单来说，JS 的装饰器可以用来“装饰”三种类型的对象：类的属性/方法、访问器、类本身，简单看几个例子吧。

## 针对属性/方法的装饰器
```Javascript
// decorator 外部可以包装一个函数，函数可以带参数
function Decorator(type){
    /**
     * 这里是真正的 decorator
     * @target 装饰的属性所述的类的原型，注意，不是实例后的类。如果装饰的是 Car 的某个属性，这个 target 的值就是 Car.prototype
     * @name 装饰的属性的 key
     * @descriptor 装饰的对象的描述对象
     */
    return function (target, name, descriptor){
        // 以此可以获取实例化的时候此属性的默认值
        let v = descriptor.initializer && descriptor.initializer.call(this);
        // 返回一个新的描述对象，或者直接修改 descriptor 也可以
        return {
            enumerable: true,
            configurable: true,
            get: function() {
                return v;
            },
            set: function(c) {
                v = c;
            }
        }
    }
}
```
注意这里的 target 对应的是被装饰的属性所属类的原型，如果是装饰一个 A 类的属性，并且 A 类是继承自 B 类的，这时候你打印 target，获取到的是 A.prototype，它的结构是这样的，这里一定要注意：

![](/js-decorator-code/187FCC2A-8CC4-46C4-B8A3-A7FD5E0376F6.png)

如果需要操作 target，可能需要搞清楚这个问题。

## 针对 访问操作符的装饰
与属性方法类似，就不详述了。
```js
class Person {
  @nonenumerable
  get kidCount() { return this.children.length; }
}

function nonenumerable(target, name, descriptor) {
  descriptor.enumerable = false;
  return descriptor;
}
```
## 针对类的装饰
```js
// 例如 mobx 中 @observer 的用法
/**
 * 包装 react 组件
 * @param target
 */
function observer(target) {
    target.prototype.componentWillMount = function() {
        targetCWM && targetCWM.call(this);
        ReactMixin.componentWillMount.call(this);
    };
}
```
其中的 target 就是类本身（而不是其 prototype）

## 真实场景应用
今天，我们要介绍的主要是，如何将 Decorator 这个特性应用于数据定义层，实现一些类似于类型检查、字段映射等功能。

关于数据定义层（Model），其实就是应用内出现的各种实体数据的定义，也就是 MVVM 中的 M 层，注意，和 VM 层做好区分，Model 本身不提供数据的管理和流通，只负责定义某个实体本身的属性和方法，例如页面里有一辆车的模块，我们就定义一个 CarModel，它用来描述车辆的颜色、价格、品牌等信息。

关于为什么要在前端应用内定义明确的 Model，这个我之前在知乎上也早有论述，核心几点：

* 提高可维护性。将数据源头的实体做一个固定而准确的描述，这个对于串联理解整个应用非常重要，特别是在重构或者接手别人的代码的时候，你需要准确的知道一个页面（或者是一个模块）它会包含哪些数据，这些数据分别有哪些字段，这样更便于理解整个应用的数据逻辑。
* 提高确定性。当你要给你的界面增加几个车辆字段的时候，你不清楚之前是否已经定义过这些字段，服务端是否会返回这些字段，可能要请求一下（并且要有权限取到所有字段）才能知道，但是如果有 model 的明确定义，有什么字段就一目了然了。
* 提高开发效率。在这一层统一做一些数据映射和类型检查等工作，这也是今天要讲的重点。

以我们团队 RN 开发框架中 Model 部分的实现为例，我们至少提供了三个基础的基于 Decorator 的功能：类型检查，单位转换，字段映射。接下来我会先简单介绍下这几个功能是做什么的，随后介绍如何实现这些 Decorator。

先来看看最终调用时候的代码
```js
class CarModel extends BaseModel {
    /**
     * 价格
     * @type {number}
     */
    @observable
    @Check(CheckType.Number)
    @Unit(UnitType.PRICE_UNIT_WY)
    price = 0;

    /**
     * 卖家名
     * @type {string}
     */
    @observable
    @Check(CheckType.String)
    @ServerName(‘seller_name’)
    sellerName = ‘’;
}
```
可以看到我们有三个自定义的 decorator ：
```js
@Unit,         // 单位转换装饰器
@Check,        // 类型检查装饰器，
@ServerName    // 数据字段映射装饰器，当前后端定义的字段名不一致的时候用
```

@Unit 是一个比较特殊的装饰器，它的作用是在前后端之间自动转换单位，也就是前端和后端交换某些带单位的数据的时候，会把根据各端的注解和装饰器，把真实值转换成带单位的值传给另一端，然后另一端会在框架层自动转成它定义的单位，以此解决前后端单位不一致，交换数据时混乱导致的问题。

被 @Unit 装饰过的属性，读写的时候都是按照前端的单位读写，然后再转换成 JSON 的时候就会特殊处理成类似 12.3_$wy 这样的格式，表示这个数的单位是万元。

@Check 更为容易理解，就是用来检查字段类型，或者检查字段格式，或者一些自定义检查，例如正则表达式等。

@ServerName 则用来做映射，例如前后端对同一个界面元素的命名不同，这时候不需要完全按照服务端的命名来决定，可以在前端用另外一个属性名，然后将其装饰成服务端的字段名。

## 基础实现
我们的目标就是实现这几个 Decorator，按照之前对 Decorator 的科普，其实要独立实现这几个功能其实非常简单。

以 @Check 为例，我们改写被包装属性的 descriptor，返回一个新的 descriptor，将被包装属性的 getter 和 setter 重新定义，然后在其调用 setter 的时候先检查传入参数的类型和格式，做一些对应的处理。
```js
/**
 * 此注解如果赋值的时候匹配到的类型有问题，会在控制台显示警告
 * @param type CheckType 中定义的类型
 * @returns {Function}
 * @constructor
 */
function CheckerDecorator(type){
    return function (target, name, descriptor){
        let v = descriptor.initializer && descriptor.initializer.call(this);
        return {
            enumerable: true,
            configurable: true,
            get: function() {
                return v;
            },
            set: function(c) {
                // 在此对传入的 c 的值做各种检查
                var cType = typeof(c);
                // …
                v = c;
            }
        }
    }
}
```
非常简单，其他几个 Decorator 的实现也类似，可能像@Unit 这种实现起来会稍显复杂，不过只要在 Decorator 中记住每个属性标注的单位，在序列化的时候获取对应的属性对应的单位然后做转换就可以了。
## 基础实现的问题
但是，到这里，问题其实还没有完！

我们的确实现了一个可用的 Decorator，但是这些 Decorator 可以叠加使用吗？另外可以和业界常用的一些 Decorator 混用吗？例如 mobx 中的 @ observable。也就是我上面最开始的实例的用法：

```js
@observable
@Check(CheckType.String)
@ServerName(‘seller_name’)
sellerName = ‘’;
```
如果你按照我刚才的方式实现 @Check 和 @ServerName 的话，你会发现两个致命的问题：
* 这两个自己实现的 Decorator 首先就没法叠加使用。
* 这两个 Decorator 都无法和 @observable 这个同时使用。
* 为什么呢？问题就出在我们改写属性的 getter 和 setter 的实现原理上。首先，每次给一个属性定义 getter 和 setter 都会覆盖前一次的定义，也就是这个动作只能有一次。然后，mobx 的实现非常依赖对 getter 和 setter 的定义（可以参考我之前的文章： [如何自己实现一个 mobx - 原理解析](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F26559530) ）
事实上，Decorator 本身叠加使用时没问题的，因为你的每次包装，都会将属性的 descriptor 返回给上一层的包装，最后就是一个函数包函数包函数的效果，最终返回的还是这个属性的 descriptor 。

## 进阶实现
那我们就需要摒弃掉定义 getter 和 setter 的实现方式。其实除了这种方式，还有很多方式可以实现上述的功能，核心就是一点，在装饰器函数里，将你需要处理的属性和对这个属性需要做的处理的对应关系都记录下来，然后在处理实例化数据和序列化数据的时候，把对应关系取出来，执行相关逻辑即可。

废话不说，我们直接上一种将这个对应关系挂载到类的原型上的一个实现方式。
```js
function Check (type) {
    return function (target, name, descriptor) {
        let v = descriptor.initializer && descriptor.initializer.call(this);
        /**
         * 将属性名字以及需要的类型的对应关系记录到类的原型上
         */
        if (!target.constructor.__checkers__) {
            // 将这个隐藏属性定义成 not enumerable，遍历的时候是取不到的。
            Object.defineProperty(target.constructor, “__checkers__”, {
                value: {},
                enumerable: false,
                writeable: true,
                configurable: true
            });
        }
        target.constructor.__checkers__[name] = {
            type: type
        };
        return descriptor
    }
}
```
注意，我前面提到的一个信息，装饰函数的第一个参数 target 是包装属性所属的类的原型（prototype），这个通过看 babel 编译后的结果可以看到。然后我这里为什么将对应关系挂载到 target.constructor 上，是因为我所有的 Model 类，都是继承自我提供的一个 Model 基类的（BaseModel），target 拿到的不是子类的原型，而是基类的原型，target.constructor 拿到的才是最终的子类。也就是我把对应关系挂载到了开发定义的子类上。

接下来看看基类的代码，核心提供两个方法，分别是映射数据和序列化的方法。
```js
class BaseModel {
    /**
    * 将后端数据直接映射到当前的示例上
    */
    __map (json) {
        let alias = this.constructor.__aliasNames__;
        let units = this.constructor.__unitOriginals__;
        let checkers = this.constructor.__checkers__;
        for (let I in this) {
            if (!this.hasOwnProperty(i)) return;
            // 如果有多层装饰器，需要经过多个逻辑处理最终产生一个最终值 realValue
            let realValue = json[I];
            // 接下来一步一步处理数据
            // 首先检查别名数据，并做映射
            if (alias && typeof(alias[I]) !== ‘undefined’) {
                // ……
            }
            // 然后针对数据检查类型
            if (checkers && checkers[I]) {
                // ……
            }
            // 最终，对数据做单位转换
            if (units && units[I]) {
                // ……
            }
            // 赋值
            this[I] = realValue;
        }
    }
    /**
    * 复写 JSON.stringify 时自动调用的函数
    */
    toJSON () {
        let result = {};
        let units = this.constructor.__unitOriginals__;
        for (let I in this) {
            if (!this.hasOwnProperty(i)) return;
            if (units && units[I]) {
                // 序列化时，有需要加单位的加上单位
                result[I] = this[I] + ‘_$’ + units[I];
            } else {
                result[I] = this[I];
            }
        }
        return result;
    }
}
```
在 __map 函数中，我们将当前类（this.constructor）上的对应关系都取出来，然后做数据校验和映射，这里应该不难理解了。

最终应用的代码就是我们开篇贴出来最终使用的代码，只要相应的 Model 类继承自 BaseModel 即可。

通过这样的方式实现的 Decorator ，因为没有用到任何 getter setter 相关的功能，所以可以和 mobx 这样的库完美融合，并且可以无限叠加使用，不过如果你用到了多个三方库，他们都提供了对应的 Decorator，然后又都修改了 getter 和 setter，那就没有办法了！

## 总结
Decorator 虽然原理非常简单，但是的确可以实现很多实用又方便的功能，目测前端领域很多框架和库都会大规模使用这个特性，但是也希望这些库在实现 Decorator 的时候考虑下通用性，考虑下叠加和共存的问题。像上面 mobx 的 @observable，不关无法叠加，而且和我自己实现的 Decorator 的顺序都不能乱，必须在最外层，因为它改变了整个属性的性质，不写在最外层的时候，会发现一些莫名其妙的问题。
