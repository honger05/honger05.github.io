 ---
layout: post
title: arale 之 events 篇
date: 2015-08-26
categories: arale
tags: [js, arale]
---
{% include JB/setup %}
# arale 之 events 篇
---

这是 读 arale 源码 系列的第二篇， events 篇。

> Events 提供了基本的事件添加、移除和触发功能。

仅提供非常纯粹的事件驱动机制。同类产品以下两种

- 重点借鉴 Backbone.Events .
- NodeJS 的 EventEmitter 功能点较多，主要考虑服务器端。

### on  `object.on(event, callback, [context])`

````js
// 注册事件，全保存在 this.__events 中，可以指定回调函数执行时上下文 [ context ]
Events.prototype.on = function(events, callback, context) {
  var cache, event, list;
  if (!callback) return this;
  cache = this.__events || (this.__events = {});
  // 可以接收多个参数，如 obj.on('x y', fn) 等同于 obj.on('x').on('y')
  events = events.split(/\s+/);
  while (event = events.shift()) {
    // 同一种事件放在同一个集合中 callback，context 两两保存的方式
    // Backbone 的内部结构是链表，改为数组后，性能大幅提升。Backbone 已合并 Arale 代码。
    list = cache[event] || (cache[event] = []);
    list.push(callback, context);
  }
  // 链式操作 返回 this
  return this;
}
````

<!--break-->

----
### off `object.off([event], [callback], [context])`

从对象上移除事件回调函数。三个参数都是可选的，当省略某个参数时，表示取该参数的所有值

````js
Events.prototype.off = function(events, callback, context) {
  // 变量无块作用域，所以都定义在最顶部
  var cache, event, list, i;
  if (!(cache = this.__events)) return this;
  // 三个参数都不存在时，移除 this.__events 所有事件
  if (!(events || callback || context)) {
    delete this.__events;
    return this;
  }
  events = events ? events.split(/\s+/) : keys(cache);
  // 遍历事件集合
  while (event = events.shift()) {
    list = cache[event];
    if (!list) continue;
    // 后面两个参数都不存在时，移除所有 event 事件
    if (!(callback || context)) {
      delete cache[event];
      continue;
    }
    for (i = list.length - 2; i >= 0; i -= 2) {
      // 当 callback 或 context 匹配时，删除之。。
      if (!(callback && list[i] !== callback || context && list[i + 1] !== context)) {
        list.splice(i, 2);
      }
    }
  }
  return this;
}
````

----
### trigger

````js
Events.prototype.trigger = function(events) {
  var cache, event, all, list, i, len, rest = [], args, returned = {status: true};

  if (!(cache = this.__events)) return this;
  events = events.split(/\s+/);
  // 循环要比 [].slice() 复制数组 快！
  for (i = 1, len = arguments.length; i < len; i++) {
    rest[i-1] = arguments[i];
  }
  while (event = events.shift()) {
    if (all = cache.all) all = all.slice();
    if (list = cache[event]) list = list.slice();
    callEach(list, rest, this, returned);
    callEach(all, [ event ].concat(rest), this, returned);
  }
  return returned.status;
}
````










