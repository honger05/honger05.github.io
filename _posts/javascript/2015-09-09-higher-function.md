---
layout: post
title: js 函数式编程之高阶函数
date: 2015-09-09
categories: javascript
tags: [js, skill]
---
{% include JB/setup %}

# js 函数式编程之高阶函数
---

满足两点就是高阶函数

1. 以函数为参数。
2. 以函数为返回值。

它是更高层次的抽象，比如，将一个小的运算封装成一个函数，就是一层抽象，而在这个抽象的基础上，再进行一次抽象，即为高阶抽象

> 在函数编程语言中这个概念是比较基本的。而在其他的命令式编程语言，如C，java等中，提供的是模板或者接口这样的概念

比如：我要判断一个数是不是偶数。可以抽象成一个函数。

````js
function isEven(x) {
	return x % 2 === 0；
}
````

那么我要判断一个数是不是奇数的话，则要用 `!isEven(x)` 了。但这样并不直观，可以再抽象一个层级

````js
function isOdd(x) {
	return !isEven(x);
}
````

这时，`isOdd` 就是一个高介函数。

<!--break-->

#### 高阶函数的妙用

再比如：我要判断一个元素是否存在某个类样式，我需要定义 hasClass(element, className) 函数。我要判断它不存在这要写一个 hasNotClass(element, className) 函数。

这时，他们都存在一些相似的地方。就可以再抽象一个层级，定义一个 not(fn) 函数。

````js
function not(fn) {
	return function() {
    return !fn.apply(this, arguments);
  }
}
````

有了这个 `not(fn)` 函数，我们就能轻易的用它来构造其它函数了。

````js
var isOdd = not(isEven);

var hasNotClass = not(hasClass);
````

`not(fn)` 是高阶函数，高阶抽象（二级抽象），而 `not(isEven)` `not(hasClass)` 是它的具体实现（一级抽象）

#### 何时使用高阶函数

当你发现很多相似的函数时，他们有共同点时，可以根据共同点来再次抽象封装。

----
#### 高阶函数实践

待续...




