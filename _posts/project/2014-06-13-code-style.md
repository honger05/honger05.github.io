---
layout: post
title: javascript 代码规范
date: 2014-06-13 20:23:13
categories: codestyle
tags: [js, codestyle]
shortContent: "为了提高代码的可读性、可维护行，约定一些开发规范"
---
{% include JB/setup %}


### javascript 代码规范

> 摘自arale的开发规范

##### 一、缩进
缩进使用 2个 或 4个 空格， 组件内保持统一， 不混合使用， 禁用tab。

##### 二、花括号 {}

* 花括号不换行
* if、 else、 while、 for 等中间的小括号 左右各空一格。
* 操作符直接要空格
* 标点符号之后必须接一个空格
* 不允许一行判断，一律换行

```javascript

for (var i in foo) {

} //ok

var a = b + c; //ok

var a=b+c; //no

if (foo) return; //no

if (foo)
  return; //ok
```

##### 三、命名约定

* 常量 全大写
* 变量 驼峰
* 类名 首字母大写


##### 四、变量声明

* 变量在使用前必须声明
* 对于单个var模式和多var模式不强作约定，但在同一个文件里，必须保持一个风格。

*声明多个变量的写法*
如果是纯声明，或简单的赋值，可以写成一行。比如：

`var chche, event, all, list = [], args`

如果赋值语句较长，建议使用多var模式

```javascript
var $ = require('jquery');
var position = require('position');
```
> 目前高级压缩器，Closure 和 UglifyJS 等，能自动处理 var 了。在压缩函数时，会自动做变量提升和var单一化

### *我的自白：*

之前不知道在哪看到的单var写法，说可以提高效率。如：

```javascript
var $ = require('$'),
    handlebars = require('handlebars'),
    chache = require('chache');
```

在我维护这个代码的时候，比如我要删掉最后一个，我还要去改倒数第二个的标点符号。

或者是增加一个的时候，我需要去调缩进。

总之、 会带来一些小麻烦。


#### 参考

[编程与文档风格讨论](https://github.com/aralejs/aralejs.org/issues/36)





