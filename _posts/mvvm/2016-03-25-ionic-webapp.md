---
layout: post
title: ionic angular + webpack 实践
date: 2016-03-25
categories: ionic
tags: [ionic, webapp, webpack, angular]
---
{% include JB/setup %}

# ionic angular + webpack 实践
---

#### 一、环境搭建

使用 webpack 来构建项目，一方面可以使用任意你想用的技术栈提升你的开发体验，另一方面可以解决产品上线之后性能和用户体验方面的问题。这里记录下对 webpack 其它方面的心得。

&emsp;

*1.分离 Angular 模板文件*

&emsp;&emsp;使用 webpack 的一个好处是可以预编译 html 文件，在纯 HTML 代码的情况下，公共代码块，partial 等等只能在线上运行的时候通过 js 来加载，用户体验和代码体验上都非常差，而所有页面都自己手动写一遍又特别难以维护。在我看的第一本书上就说过一句话 “don't repeat yourself”。在 css，js 预编译器火爆的时代，我也看到了 html 预编译的好处。这是纯前端的一个非常好的实践。（另一个好的实践就是 通过 nodejs 服务器的前后端分离）

非常简单的一个 loader 就可以加载 部分 html 代码了

````js
module.exports = function(content) {
  this.cacheable && this.cacheable();
  this.value = content;
  return "module.exports = " + JSON.stringify(content);
}
module.exports.seperable = true;
````

<!--break-->

在 html 中引入

````html
<!DOCTYPE html>
<html>
  <head>
    <title>vue-template</title>
    <%= require('html!tmpl/inc-meta.html') %>
  </head>
  <body>
    <div id="app"></div>

    <%= require('html!tmpl/inc-footer.html') %>
  </body>
</html>
````

在 js 中引入

````js
$stateProvider

  .state('cover', {
    url: '/cover',
    template: require('./partials/cover.html'),
    controller: 'CoverCtrl'
  })

  .state('out', {
    url: '/out',
    template: require('./partials/out-abstract.html'),
    abstract: true
  })

  ...
````

&emsp;

*2.线上模式，开发模式的代码结构*

&emsp;&emsp;我们既要解决线上的用户体验，也要解决自身的开发体验。产品上线了，目录结构肯定是按资源划分的，这样有利于部署 cdn ，公共文件缓存等等好处的。而在开发阶段也按资源划分的话，开发体验就差了。如果要改动某一个图片，得找半天才能找的着。所以在开发阶段应该是按模块划分。所有模块要用到的资源（css,js,图片..）一定是在离它最近的地方。这个功能是 webpack 天生自带的。


&ensp;

*3.配置一份 webpack.base 配置文件。*

&emsp;&emsp;区分 `开发模式` 和 `生产模式` 。比如在生产模式中 css 要用 ExtractTextPlugin 编译成独立的文件。而在开发模式因为要实现热替换而不能这样编译成独立文件，还有生产模式需要压缩代码。所以一般会为 webpack 配置两套配置文件。有时候开发模式的配置文件改了，而生产模式的配置文件忘记改了，这样维护起来容易出 bug。作为一个从来不 repeat yourself 的人，只能通过 nodejs 去配置了。

针对 css 部分

````js
exports.cssLoaders = function (options) {
  options = options || {}
  function generateLoaders (loaders) {
    var sourceLoader = loaders.map(function (loader) {
      var extraParamChar
      if (/\?/.test(loader)) {
        loader = loader.replace(/\?/, '-loader?')
        extraParamChar = '&'
      } else {
        loader = loader + '-loader'
        extraParamChar = '?'
      }
      return loader + (options.sourceMap ? extraParamChar + 'sourceMap' : '')
    }).join('!')

    if (options.extract) {
      return ExtractTextPlugin.extract('vue-style-loader', sourceLoader)
    } else {
      return ['style-loader', 'vue-style-loader', sourceLoader].join('!')
    }
  }

  return {
    css: generateLoaders(['css']),
    postcss: generateLoaders(['css']),
    less: generateLoaders(['css', 'less']),
    sass: generateLoaders(['css', 'sass?indentedSyntax']),
    scss: generateLoaders(['css', 'sass']),
    stylus: generateLoaders(['css', 'stylus']),
    styl: generateLoaders(['css', 'stylus'])
  }
}
````

&emsp;

#### 二、ionic 的使用

&emsp;

*1. 性能与开发体验*

&emsp;&emsp;不得不说 ionic 在做单页的时候有非常多的优势，非常接近 native app 的感觉。在 ios 上基本没什么问题，运行的也非常流畅，Android 经过优化和设计规避之后也非常棒。由于 angular1 的性能问题，ionic 基于 angular2 的版本也已经出来了。虽然有被 react native 赶超之势。但我还是对 ionic2 非常有信心的。在开发体验上，angular 的 api 真的好麻烦写，这一点 vue 做的很好。

考虑到压缩代码会把关键字替换，因此 angular 在定义的时候需要这样。

````js
angular.module('antsins.controllers')

.controller('GoodsCtrl',
  ['$scope', '$log', '$ionicSlideBoxDelegate', 'ipCookie', '$ionicPopup', 'ENV',
   '$ionicLoading', '$rootScope', '$state', 'Order', '$stateParams', 'Category',
   'Gender', 'ionicDatePicker', '$ionicHistory',

  function($scope, $log, $ionicSlideBoxDelegate, ipCookie, $ionicPopup, ENV,
    $ionicLoading, $rootScope, $state, Order, $stateParams, Category,
    Gender, ionicDatePicker, $ionicHistory){
````

除了 api 有点烦之外，其它地方还是非常不错的，ionic 大大提高了开发效率。

&emsp;

*2. 原生滚动*

ionic 默认是 JavaScript 的滚动，在中低端 Android 中是有一些卡顿的。也提供了原生滚动的选择，可以在单个 view 中开启，在 ion-content 标签中加入属性

````js
overflow-scroll="true"
````

对所有 view 开启它，在 js 中执行

````js
$ionicConfigProvider.scrolling.jsScrolling(false)
````

&emsp;

*2. 在单个页面自定义返回事件*

````js
var old_back = $rootScope.$ionicGoBack

$scope.$on('$ionicView.beforeEnter', function() {
  $rootScope.$ionicGoBack = function() {
    $ionicPopup.confirm({
      title: '确定返回？',
      template: '返回将丢失您所输入的数据！',
      okText: '确定',
      okType: 'button-assertive',
      cancelText: '取消'
    }).then(function(res) {
      if(res) {
        $ionicHistory.goBack()
      }
    })
  }
})

$scope.$on('$ionicView.beforeLeave', function() {
  $rootScope.$ionicGoBack = old_back
})
````

&emsp;

#### 三、效果

&emsp;

&emsp;开发过程也是曲折，由于缺少产品经理，需求变动太大。/(ㄒoㄒ)/~~ ，也是意识到了 产品经理 的重要性。

<div class="row">
  <div class="span6" align="center" style="margin-top:5px;">
    <img width="80%" src="/assets/images/IMG_9432_s.jpg">
  </div>
  <div class="span6" align="center" style="margin-top:5px;">
    <img width="80%" src="/assets/images/IMG_9460_s.jpg">
  </div>
</div>

&emsp;

&emsp;经过多次需求更新之后的[成品](http://ins.xyisi.com/)


