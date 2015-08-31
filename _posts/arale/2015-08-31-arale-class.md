---
layout: post
title: arale 之 class 篇
date: 2015-08-31
categories: arale
tags: [js, arale]
shortContent: ""
---
{% include JB/setup %}
#arale 之 class 篇
---

> arale 是阿里、开源社区明星人物--玉伯，开发的一套组件，代码相当优美，大赞玉伯的开源精神。

### 先谈谈基于原型的继承。

先看看 segementfault 上讨论的一道题。

````js
function F() {}
Object.prototype.a = function() {}
Function.prototype.b = function() {}
var f = new F()
// F.a F.b f.a
````

<!--break-->

F 可以调用 a 和 b，因为 F 的原型链是这样的。(直观解释：F 是 Function 实例，F 继承自 Object)

````js
F ----> Function.prototype ----> Object.prototype ----> null

//既 F.__proto__ === Function.prototype
// Function.prototype.__proto__ === Object.prototype
// Object.prototype.__proto__ === null
````

而 f 只能调用 a ， f 的原型链是这样的。（f 是 F 的实例，一切皆对象，f 继承自 Object）

````js
f ----> F.prototype ----> Object.prototype ----> null

//既 f.__proto__ === F.prototype
// F.prototype.__proto__ === Object.prototype
// Object.prototype.__proto__ === null
````

在 f 的原型链上并没有 Function.prototype . 因此访问不到 b 。

> 注意，访问原型对象 `__proto__` 是非标准方法，ES5 标准方法是 Object.getPrototypeOf();

到这里，基于原型链的继承已经很明显了，只需要

````js
function Animal() {}

function Dog() {}

Dog.prototype.__proto__ = Animal.prototype;

var dog = new Dog();

// dog.__proto__ --> Dog.prototype;
// dog.__proto__.__proto__ --> Animal.prototype
````

基于 ES5 标准写法是

```js
Dog.prototype = Object.create(Animal.prototype);
```

---
### 来看看 arale 的封装

````js
// 创建原型链
function Ctor() {};

var createProto = Object.__proto__ ? function(proto) {
  return {
    __proto__: proto
  }
} : function(proto) {
  Ctor.prototype = proto;
  return new Ctor();
}
````

有三种写法可以实现原型链继承，但是我测试 new Ctor 是最慢的啊，Object.creaete 其次，`__proto__` 是最快的。

```js
function Ctor() {}

function getProto1(proto, c) {
  Ctor.prototype = proto;
  var o = new Ctor();
  o.constructor = c;
  return o;
}

function getProto2(proto, c) {
  return Object.create(proto, {
    constructor: {
      value: c
    }
  })
}

function getProto3(proto, c) {
  return {
    __proto__: proto,
    constructor: c
  }
}
```

接着往下看。。。

````js
function Class(o) {
  if (!(this instanceof Class) && isFunction(o)) {
    return classify(o);
  }
}

function classify(cls) {
  cls.extend = Class.extend;
  cls.implement = implement;
  return cls;
}
````

这种写法是，当不使用 `new` 关键字调用时，将参数类化，如：

````js
function Animal() {}

Class(Animal);// Animal 拥有了 extend 和 implement 方法
````




