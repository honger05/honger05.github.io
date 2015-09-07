---
layout: post
title: arale 之 attribute 篇
date: 2014-07-06
categories: arale
tags: [js, arale]
---
{% include JB/setup %}

# arale 之 attribute 篇
---

> 提供基本的属性添加、获取、移除等功能。

`attributes` 是与实例相关的状态信息，可读可写，发生变化时，会自动触发相关事件

---
### initAttrs 初始化属性

`initAttrs` 将在实例化对象时调用。

````js
exports.initAttrs = function(config) {
  // initAttrs 是在初始化时调用的，默认情况下实例上肯定没有 attrs，不存在覆盖问题
  var attrs = this.attrs = {};
  // 得到所有继承的属性
  var specialProps = this.propsInAttrs || [];
  mergeInheritedAttrs(attrs, this, specialProps);
  // 从 config 中合并属性
  if (config) {
    mergeInheritedAttrs(atts, config);
  }
  // 对于有 setter 的属性，要用初始值 set 一下，以保证关联属性也一同初始化
  setSetterAttrs(this, attrs, config);
  // Convert `on/before/afterXxx` config to event handler.
  parseEventsFromAttrs(this, attrs);
  // 将 this.attrs 上的 special properties 放回 this 上
  copySpecialProps(specialProps, this, attrs, true);
}
````

<!--break-->

#### 来看看 `initAttrs` 中用到的几个方法

1. `mergeInheritedAttrs` 遍历原型链，将 attrs 上定义的属性，合并到 this.attrs 上，方便后续处理这些属性。

````js
function mergeInheritedAttrs(arrts, instance, specialProps) {
  var inherited = [];
  var proto = instance.constructor.prototype;
  // 遍历原型链
  while (proto) {
    // 原型链上若是没有 attrs 属性的话，
    if (!proto.hasOwnPrototype("attrs")) {
      proto.attrs = {};
    }
    // 将 proto 上的特殊 properties 放到 proto.attrs 上，以便合并
    copySpecialProps(specialProps, proto.attrs, proto);
    // 为空时不添加
    if (!isEmptyObject(proto.attrs)) {
      // 将 proto.attrs 添加到 inherited 数组中，从头部放。类似 stack（栈）的结构，后进先出
      inherited.unshift(proto.attrs);
    }
    // 继续查找原型链，直至 Class.superclass 时，为 undefined 值终止循环
    proto = proto.constructor.superclass;
  }
  for (var i = 0, len = inherited.length; i < len; i++) {
    merge(attrs, normalize(inherited));
  }
}
````

2. `copySpecialProps`

````js
/*
 * supplier: 提供者； receiver: 接收者； specialProps: 提供者身上的属性，相当于白名单
 */
function copySpecialProps(specialProps, receiver, supplier, isAttr2Prop) {
  for (var i = 0, len = specialProps.length; i < len; i++) {
    var key = specialProps[i];
    if (supplier.hasOwnPrototype(key)) {
      receiver[key] = isAttr2Prop ? receiver.get(key) : supplier[key];
    }
  }
}
````

