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

> Backbone.Marionette 的中文资料真是少之又少，因此这篇会尽量介绍的较为详细。

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

----
### 视图 View & 区域 Region

Marionette 扩展了非常丰富的视图 View 组件：`ItemView` `CollectionView` `CompositeView` `LayoutView` .

不仅于此，Marionette 还使用 Region （区域）来配合 View （视图）。

一般会先添加一个 Region 来定位一块地方，再决定这块地方显示哪个 View 。


````js
// 你可以理解为一个中心 APP 对象。当一切准备就绪的时候，调用 App.start(options) 启动应用。
var MyApp = new Backbone.Marionette.Application();

// 添加一个 region，它对应一个 dom 节点
App.addRegions({
  mainRegion: '#content'
});

// 让这个 region 显示一个视图, 这个视图会立即渲染
App.mainRegion.show(new MyView());
````

从头说起吧，你可能注意到了，上面实例化了一个 Marionette.Application 对象。

通常需要定义一个 App ，通过 Initializers 把所有的事都绑定在上面。等待 start 方法调用的时候，开始执行。

````js
// start 方法调用后，立即执行 Initializers
MyApp.addInitializer(function(options) {
  // 实例化 compositeView
  var angryCatsView = new AngryCatsView({
    collection: options.cats
  });
  // 显示这个视图
  MyApp.mainRegion.show(angryCatsView);
})

MyApp.start({cats: cats});
````

这里只绑定了一个 Initializer，在一个复杂的应用中，你可能会绑定多个的。start(options) 中的参数 options 会传递个每个 Initializer。


通过扩展 Backbone.Events, 实现 Aspect（切面编程） 你可以监听这些事件，让应用更加灵活。

> 之前分析过 Arale 的 Events 代码，现在的 Backbone.Events 就是从 Arale 的 Events 合并过来的，看我的 [gitbook](http://code.honger05.com/arale/aspect.html)

````js
App.on('initialize:before', function(options) {
     // doSomething...
});
App.on('initialize:after', function(options) {
     // doOtherthing...
});
App.on('start', function(options) {
  Backbone.history.start();
});
````

好了，现在已经知道了 Marionette 是怎么启动的了，下一步是了解它是怎么管理视图 View 的。

----
#### LayoutView

布局视图，比如你的界面上可能用 header main footer 等区域。

````html
<div id="content">
  <div id="header"></div>
  <div id="main"></div>
  <div id="footer"></div>
</div>
````

你可以这样来定义布局视图，这样你就掌控全局了。

````js

var RootLayout = Backbone.Marionette.LayoutView.extend({

  el: '#content',

  regions: {
    header: '#header',
    main: '#main',
    footer: '#footer'
  }

})
````

一般会把这个 root 挂载到 App 上。

````js
var MyApp = Backbone.Marionette.Application.extend({
  setRootLayout: function () {
    this.root = new RootLayout();
  }
});

// App 启动前，实例化它，得到 App.root
MyApp.on('before:start', function () {
  MyApp.setRootLayout();
});
````

现在 App 上有了 root 的控制权了，可以任意设置某个区域显示某个视图了。

````js
MyApp.root.showChildView('header', new HeaderLayout());
````


----
#### ItemView 与 CompositeView

单条记录 Model 对应 ItemView, CompositeView 不仅对应的是一个包含 ItemView ，还对应有其他一些相关视图.

他们通常在一起使用，并且 ItemView 是 CompositeView 的 childView 属性值。

比如这样两个 template

````html
<!-- CompositeView 模板 -->
<thead>
  <tr><th>Name</th></tr>
</thead>
<tbody>
</tbody>

<!-- ItemView 模板 -->
<tr><td>{{name}}</td></tr>
````

我需要在 tbody 标签下加入多个 ItemView。这样你就需要使用 childViewContainer 来指定 childView 被加在什么地方。

很显然此处： `{childViewContainer: 'tbody'}`.

这里是以上代码

````js
var AngryCatView = Backbone.Marionette.ItemView.extend({
  template: require('./tpl/angrycat.handlebars'),
  tagName: 'tr',
  className: 'angry_cat'
});

var AngryCatsView = Backbone.Marionette.CompositeView.extend({

  tagName: 'table',
  id: 'angry_cats',
  className: 'table-striped table-bordered',

  template: require('./tpl/angrycats.handlebars'),

  childView: AngryCatView,

  childViewContainer: 'tbody'

})
````

细心的童鞋应该已经注意到了，模板中为什么不用 table 标签包裹起来。这是因为 Marionette 会为你包裹一层，若你不指定 `tagName` 则默认是 div 标签。 指定的 `className`, `id` 属性也是加在这层上面的。

*纯 Backbone 代码需要自己来实现 render 渲染 DOM，在这里 Marionette 通过指定的 template 属性自动渲染了。*


*重点* 在 这一层 我们要做的是监听 Model，Collection 的变化，来同步视图。和 Backbone 做法一样。不同的是，Marionette 可以指定是全部重绘（render）还是部分重绘（renderCollection）

----
### Event Aggregator

事件聚合器，这个东西主要是用来解耦的，例如：从 ItemView 上来个事件，改变了 model 的属性。影响了集合的排列。需要更新到 CompositeView 上。

在 ItemView 中调用 CompositeView ？ 这样做是不对的，因为会让应用越来越复杂的。视图 View 也不应该去处理 business logic。

使用 Event Aggregator 会让程序解耦，它相当于一种 Publish/Subscribe 模式。视图只需要去通知 notice 模型 Model or Collection 来处理。

````js
MyApp.trigger('rank:up', this.model);
````

在模型 Model or Collection 初始化的时候就要 Subscribe 订阅事件 `rank:up`

````js
MyApp.on('rank:up', function(cat) {
  if (cat.get('rank') === 1) {
    return true;
  }
  self.rankUp(cat);
  self.sort();
})
````

这样的话，business logic 就是在 Model or Collection 中维护的。


----
## 以上总结

1. Marionette.View 在 Backbone.View 之上多做了很多事情，包括自动渲染和重绘等等。
2. Marionette.View 接到 Dom 事件后，可以通知 notice 集合 Collection 去处理。也可以直接命令 commands 模型 Model 去处理。

（这里只介绍了一些入门知识 Marionette 未完待续...）

![mn](https://pic2.zhimg.com/a0c7dbedbcc10868db2db704e69cd9ad_b.jpg)
















