+++
resources = ["web"]
title = "TypeScript 类型小甜点"
date = 2022-01-04
featuredImage = "/TypeScript_type_tip/cover.png"
+++

TypeScript 的类型是非常复杂的，虽然日常大家经常用到的无非是那几个简单的 type interface 之类的，但是你打开某些知名的 js lib 的源码，有时候会发现一些非常复杂的类型定义，今天简单分享个我之前写 colors-web 的时候定义的一个类型，其实很简单。

首先，我们看下需求：

现有一个 class，他的方法是运行时动态赋予的，可能会有一百多个方法

如 Color 类

```typescript
class Color {
   styles = [];
   constructor() {}
   color(colorValue} {
      this.styles.push('color:'+colorValue);
      return this;
   }
   bgColor(colorValue) {
     this.styles.push('background-color:'+colorValue);
     return this;
   }
}

const cssColors = ['red','green','yello'...] // 这里定义所有的 css 标准颜色名称，供146个

// 将 colors 变成 Color 的实例方法
const ColorFactory = ()=> {
    const instance = new Color();
    cssColors.forEach((color) => {
        Object.defineProperty(inst, color, {
            get() {
                this.styles.push(`color:${color};`);
                return this;
            },
        });
        Object.defineProperty(inst, color + "Bg", {
            get() {
                this.styles.push(`background:${color};`);
                return this;
            },
        });
    });
}
```

需求是通过 d.ts 定义以上方法的类型。

## 实现

本文只提供一种实现思路，实际上这个需求应该还可以有其他实现方式，只是我对 ts 的理解还没有太深入

### 基本类型

首先，我们先把 Color 类本身的类型定义好。

#### 1. 定义好 cssColors 类型，Color 类的参数和后续的方法都依赖这个基本的类型定义

```typescript
type CSSColors = | "black" | "silver" | "gray" | "white" | ...; // css color 共有 146 个名字
type HexColor = `#${string}`;
type RGBColor = `rgb(${number},${number},${number})`;
type RGBAColor = `rgba(${number},${number},${number},${number})`;
type hslColor = `hsl(${number},${number}%,${number}%)`;
type hslaColor = `hsla(${number},${number}%,${number}%,${number})`;

type colorValue = hexColor | rgbColor | rgbaColor | hslColor | hslaColor;

```

这里没有定义的非常精确，特别是 HexColor，实际上 ts 里也可以精确定义，只是比较复杂，这里有一个关于用正则表达式定义字面量类型的讨论，其中有一个方案，可以把正则转成类型体操

[https://github.com/Microsoft/TypeScript/issues/6579#issuecomment-711022216](https://github.com/Microsoft/TypeScript/issues/6579#issuecomment-711022216)

另外这里定义用到的是 模板字符串字面量类型，是 TS 4.1 新增的特性。

#### 2. 定义 Color 类的类型

```typescript
interface Colors {
  color: (color: ColorValue) => Colors;
  bg: (color: ColorValue) => Colors;
}
```

#### 3. 将 CSSColors 附加到 Colors 中

```typescript
// 首先定义一个 ColorBase，他的属性是由146个css颜色的字符串字面量定义的
// 通过 in 关键字遍历出来
interface ColorBase {
  [key in CSSColors]:  ((o?: string) => Colors) & Colors;
}
// 然后让 Colors 继承 ColorBase 即可拥有这些方法
interface Colors extends ColorBase {

}

const color = new Colors();
color.red(); // pass

```

其实就是一个 in 和 extends 关键字的使用，不过这里还有一个比较有意思的点，就是我这个类的属性可以有两种调用方式：

```typescript
const color = new Colors();
color.red().green();

// 也可以这样
color.red.green.log();
```

在 js 中，通过 Object.defineProperty 给 red 这样的属性定义 getter ，即可在里面塞入逻辑实现下面第二种调用方式。

对应到 ts 中，就是刚才的定义：

```typescript
red: ((o?: string) => Colors) & Colors;
```

其中第一个类型是一个正常的返回当前实例的调用，第二个类型是刚才hack的getter调用，属性本身的类型就是类本身。

不得不承认，ts 里的特性是很多的，很多时候写出一个可以正常 run 的类型甚至都有一定的运气成分，本文只是一个小分享，希望对同学们能够有帮助。

本文提到的项目的 github 地址：[https://github.com/yu-tou/colors-web](https://github.com/yu-tou/colors-web)

类型定义见：[https://github.com/yu-tou/colors-web/blob/master/src/index.d.ts](https://github.com/yu-tou/colors-web/blob/master/src/index.d.ts)