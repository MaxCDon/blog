---
title: 【JS浮点数】常见的JS浮点数精度问题及原理
date: 2021-10-22 14:50:34
tags:
---

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70617c01fab34c4395d87f6eaefe162f~tplv-k3u1fbpfcp-zoom-crop-mark:1304:1304:1304:734.awebp?)

# 起因

最近在项目线上环境中收集到客户反馈数值输入后展示值少了`0.01`，收到反馈后立刻定位代码中问题所在，原因就是上一个版本中此数值输入框改成**非四舍五入截取两位小数**，使用的方法是

```js
Math.floor(floatNum*100)/100
```
这个看似没有什么问题的方法当`floatNum`的值为`1.15`时，就出现了 Js 浮点数精度引起的异常情况，看下图。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28fdee9d86554714b5c43a575408c223~tplv-k3u1fbpfcp-watermark.image?)

可以看到 `1.15*100` 在 Js 中计算结果并非为 `115`而是`114.99999999999999`，由此通过 `Math.floor()`向下取整并除以`100`时得到的最终结果是`1.14`，与预期结果正好相差`0.01`。

定位到问题所在后立刻进行修复，决定抛弃的浮点数计算的方式，使用转换成字符串截取的方案。


```js
/**
  * 保留小数（不四舍五入）
  * @param floatNum 输入值
  * @param decimal 保留浮点数精度位数
  */
formatDecimal(floatNum, decimal) {
    let result = floatNum.toString();
    const index = result.indexOf('.');
    if (index !== -1) {
      result = result.substring(0, decimal + index + 1);
    } else {
      result = result.substring(0);
    }
    return parseFloat(result);
  }
```

在问题解决后，决定对 Js 浮点数进行梳理，并列举一些可能遇到的精度问题。

# Js 浮点数

## 标准与储存方式
Js 没有单独的浮点型，浮点数与整数都是通过 Number 类型表示，是遵循 `IEEE754标准`的 64 位双精度值，在二进制中 64 位由三个部分组成。

* sign bit（符号S）：用来表示正负号，0代表正数，1代表负数

* exponent（指数E）：用来表示次方数，中间11位

* mantissa（尾数M）：用来表示精确度, 超出的部分自动进一舍零，最后52位

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/103ee6a0c5e6471fbdb84c4b0b025b9f~tplv-k3u1fbpfcp-watermark.image?)

在二进制的科学记数法中，`IEEE754标准`的 64 位双精度值数字被公式表示为：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c096c0960fd4259a7b281d42af8d29f~tplv-k3u1fbpfcp-watermark.image?)


在十进制的科学记数法中 `1 <= M < 10`，同理在二进制科学记数法中 `1 <= M < 2`，所以 M 的整数部分只能是 1， 可以舍去，M 只存储后面小数部分。指数 E 是 11 位，在大能表示的数是 `2047 (2^11 - 1)`，由于指数包含正与负，所以取中间值 `1023` 临界值，`[1024, 2047]` 表示正指数，`[0, 1022]` 表示负指数。

以 4.5 为例转换成二进制是 `100.1`，用二进制科学记数法表示就是 `1.001 * 2^2`。 代入公式 1.001 舍去 1 尾数 M 是 001，指数 E 是 2 + 1023 = 1025 (二进制 `10000000001`)，正数 S 取 0，所以组合在一起 4.5 在 IEEE754 双精度64位标准表示如下。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14003a64d4d74d599354d69acd1e45d0~tplv-k3u1fbpfcp-watermark.image?)
图片源自于[binaryconvert](http://www.binaryconvert.com/result_double.html?decimal=052046053)


## 哪些常见的精度问题及背后原理

明白了 Js 中浮点数的标准与储存方式，再回到实际场景中看看有哪些常见的浮点数精度问题极其原因。

###  场景一  `0.1 + 0.2 != 0.3`

0.1、0.2 以及相加的和的**二进制**表示如下 

```
0.1 -> 0.0001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1010 (1001循环 进一舍零)
0.2 -> 0.0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 010 (0011循环 进一舍零)
相加 -> 0.0100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 111
```

相加所得的值转换成**十进制**恰好为 `0.30000000000000004`

([十进制小数转化为二进制小数计算方式](https://www.runoob.com/w3cnote/decimal-decimals-are-converted-to-binary-fractions.html))

([十进制小数转化为二进制小数计算工具](https://www.sojson.com/hexconvert.html))



### 场景二 `(1.335).toFixed(2) == 1.33`

浮点数精度和 toFixed 其实属于同一类问题，都是由于浮点数无法精确表示引起的，如下：

```js
(1.335).toPrecision(20);    // "1.3349999999999999645"
```

toFixed() 方法实则是对 1.3349999999999999645 四舍五入保留两位小数


### 场景三 大数问题`61453901951867050 + 5 == 61453901951867060`

因为 Javascript 的数字存储使用了 IEEE 754 中规定的双精度浮点数数据类型，而这一数据类型能够安全存储 (2^53 - 1) 到 -(2^53 - 1) 之间的数值（包含边界值）。

```js
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1 // true 最大安全整数
Number.MIN_SAFE_INTEGER === -(Math.pow(2, 53) - 1) // true 最小安全整数
```
 
这里安全存储的意思是指能够准确区分两个不相同的值，例如:
```js
Number.MAX_SAFE_INTEGER + 1 === Number.MAX_SAFE_INTEGER + 2 // true
```
将得到 true 的结果，而这在数学上是错误的
因为遵循 IEEE754 标准 JavaScript 中最大的安全整数 **Number.MAX_SAFE_INTEGER**

在场景三中，其数值 61453901951867050 超过最大安全整数，导致相加结果不准确。 
```js
61453901951867050 > Number.MAX_SAFE_INTEGER // true
```


## 解决精度问题的方案

假如我们要判断 0.1 + 0.2 是否等于 0.3

可以判断两边之差的绝对值是否小于 `Number.EPSILON`（浮点数之间最小差，如下：
```js
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON // true
```
若小于  Number.EPSILON 则认为等式成立。

但这种方式只能判断两边等式是否成立，如需要计算浮点数的精确计算值还需要利用其他方法。


目前社区已经有了很多较为成熟的库，比如 [bignumber.js](https://github.com/MikeMcl/bignumber.js)，[decimal.js](https://github.com/MikeMcl/decimal.js)，以及 [big.js](https://github.com/MikeMcl/big.js) 等。我们可以根据自己的需求来选择对应的工具。这些库不仅解决了浮点数的运算精度问题，还支持了大数运算，并且修复了原生 toFixed 结果不准确的问题。


# 结语

感谢阅读，若有不足，欢迎指正。  

提前祝 XDM 1024 节快乐~


# 参考文献

[Wikipedia-IEEE754](https://zh.wikipedia.org/wiki/IEEE_754)

[老生常谈之js浮点数精度](https://zhuanlan.zhihu.com/p/73699947)

[JS中浮点数精度问题](https://juejin.cn/post/6844903572979597319)



