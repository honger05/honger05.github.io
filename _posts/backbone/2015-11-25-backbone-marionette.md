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

> 只扯蛋，不给代码，就是耍流氓 -- honger。

完整的 tutorial 代码 [戳这里](https://github.com/honger05/spm-case/tree/master/case05-backbone/tutorial.marionette/tutorial1)， 因为我使用的是 commonjs 规范，基于 [spm](https://github.com/spmjs/spm) 的，你可以先安装,然后运行它。*[更多 spm 资料](http://spmjs.io/)*


````bash
// 安装
npm install spm -g

// 运行
spm server
````

这个 [repo](https://github.com/honger05/spm-case) 是我学习各种技术栈的一个集合，如果是初学者，可以跟我一起来学习，也可以私信我。

*你也可以 find me on [GitHub](https://github.com/honger05)*

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
        |<--------------------------|         |<--------------------------------|

````

<!--break-->

其中涉及的问题有：


1. `业务逻辑`: model 和 collection 处理大部分逻辑。他们对应着服务端后台的资源，也对应着视图显示的类容。

2. `构建 DOM`：一般是 handlebars。

3. `视图逻辑`：Backbone.View ，其中的逻辑要自己维护。

4. `视图和模型同步`: 自己维护。

5. `管理复杂的 UI 交互`：自己维护。

6. `管理状态和路由`：Backbone.Router（不支持管理视图和应用状态）

7. `创建与连接组件`: 手动实现。

<hr>

## 管理复杂的 UI 交互

那么，这篇文章着重于讲 UI 交互，所有的 UI 交互都可以被划分为：

1. `简单交互`：使用观察者同步（Observer Synchronization），被动控制显示，操作 DOM 事件来改变集合模型，视图监听集合模型的变化来改变自身。Backbone.View 就是这样工作的。

2. `复杂交互`：使用流同步（Flow Synchronization），主动控制显示 [SUPERVISING PRESENTERS](http://victorsavkin.com/post/49767352960/supervising-presenters)


使用 Marionette 工作流程是这样的：

````js
                         events for
                     complex interactions                                  notice
Template/DOM (View) ----------------------> Marionette.View (Presenter) ------------> Backbone.Model (Model)
        |                                           |        |                                |
        |               complex updates             |        |             events             |
        |<------------------------------------------|        |<-------------------------------|
                                                             |                                |
                                                             |        simple updates          |
                                                             |------------------------------->|

````

### 视图 View & 区域 Region

Marionette 扩展了非常丰富的视图 View 组件：`ItemView` `CollectionView` `CompositeView` `LayoutView` .

不仅于此，Marionette 还使用 Region （区域）来配合 View （视图）。

一般会先添加一个 Region 来定位一块地方，再决定这块地方显示哪个 View 。


````js
// 你可以理解为一个中心 APP 对象。当一切准备就绪的时候，调用 App.start(options) 启动应用。
var App = new Backbone.Marionette.Application();

// 添加一个 region，它对应一个 dom 节点
App.addRegions({
  mainRegion: '#content'
});

// 让这个 region 显示一个视图, 这个视图会立即渲染
App.mainRegion.show(new MyView());
````









