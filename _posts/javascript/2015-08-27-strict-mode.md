---
layout: post
title: javascript 之 严格模式
date: 2015-08-27
categories: javascript
tags: [js, stict, skills]
shortContent: ""
---
{% include JB/setup %}

# javascript 之 严格模式
----

> ECMAScript 5 引入了 strict mode 。会使浏览器更容易解析代码。

好了，我们开始学习用 strict mode 来写代码吧。

#### 一、语法错误 SyntaxError

- 八进制： var n = 055 和 var s = “\055”；
- 使用 delete 删除一个变量名：delete a；
- 使用 eval 或 arguments 作为变量名或函数名；
- 在语句块中使用函数声明：`if (a > b) { function f() {} }`；
- with 语句
- 使用未来保留字：implements, interface, let, package, private, protected, public, static, yield；
- 对象字面量、函数形参中使用相同名：`{a: 1, b: 3, a: 2}` 和 `function f(a, b, b) {} `；

<!--break-->

#### 二、运行时错误

- 给一个未声明的变量赋值

  ````js
    // 非严格模式下 b 会被设置成一个全局对象的值。
    function f(x) {
      'use strict'
      var a = 3;
      b = a + x; // error
    }
  ````

- 尝试删除一个不可配置的属性

  ````js
    'use strict'
    delete Object.prototype; // error
    //在非严格模式中,这样的代码只会静默失败,这样可能会导致用户误以为删除操作成功了.
  ````

- arguments 对象和函数属性

  ````js
    //在严格模式下,访问arguments.callee, arguments.caller, anyFunction.caller以及anyFunction.arguments都会抛出异常.唯一合法的使用应该是在其中命名一个函数并且重用之
    function f() {
      arguments.callee(); // error
    }
  ````

- 语义差异
  1. eval: 仅仅在你知道你在干什么的情况下使用它
  2. this: 只在它指向你之前创建的对象的情况下使用 this
  3. arguments: 总是通过他们的名字访问函数的参数，或者作为参数对象的拷贝来使用:

  ````js
    var args = Array.prototype.slice.call(arguments)//并且这样的代码应该在你的函数第一行
  ````




