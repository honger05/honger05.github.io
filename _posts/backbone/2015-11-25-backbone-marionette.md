---
layout: post
title: A Tutorial of Backbone.Marionette
date: 2015-11-25
categories: backbone
tags: [js, backbone, marionette]
---
{% include JB/setup %}

# 使用 Backbone.Marionette 管理复杂 UI 交互
---

很多人抱怨和吐槽 `Backbone` , 觉得它太简单了。什么事都要自己做。然而，Backbone 的优点也是它太简单了，它的思想就是不作任何绑定， 只提供一个骨架。正如 Backbone 的中文意思。

所以大量的问题都留给开发者自己想办法来解决，因此遭到吐槽...

当然，使用纯 Backbone 开发一个复杂应用时，情况就会变得非常糟糕。

纯 Backbone 的工作流程是这样的：  *MVP*


````js
                      events                               commands
Template/DOM (View) ----------> Backbone.View (Presenter) ----------> Backbone.Model (Model)
        |                           |         |                                 |
        |            updates        |         |             events              |
         <---------------------------         <---------------------------------|

````

其中涉及的问题有：


1. `业务逻辑`: model 和 collection 处理大部分逻辑。他们对应着服务端后台的资源，也对应着视图显示的类容。

2. `构建 DOM`：一般是 handlebars。

<!--break-->

3. `视图逻辑`：Backbone.View ，其中的逻辑要自己维护。

4. `视图和模型同步`: 自己维护。

5. `管理复杂的 UI 交互`：自己维护。

6. `管理状态和路由`：Backbone.Router（不支持管理视图和应用状态）

7. `创建与连接组件`: 手动实现。

<hr>

## 管理复杂的 UI 交互

那么，这篇文章着重于讲 UI 交互，所有的 UI 交互都可以被划分为

1. `简单交互`：使用观察者同步（Observer Synchronization），被动控制显示，操作 DOM 事件来改变集合模型，视图监听集合模型的变化来改变自身。Backbone.View 就是这样工作的。

2. `复杂交互`：使用流同步（Flow Synchronization），主动控制显示 [SUPERVISING PRESENTERS](http://victorsavkin.com/post/49767352960/supervising-presenters)









