---
layout: post
title: 微信sdk + vue + webpack 实践
date: 2016-06-09
categories: ionic
tags: [ionic, webapp, webpack, angular]
---
{% include JB/setup %}

# 微信sdk + vue + webpack 实践
---

> 我是一个特别注重开发体验的人，我坚信注重开发体验，才会把用户体验做好。

Vue 足够灵活，Webpack 也是如此。因此我尝试了很多种方法，比如 单页 多页 混合，公共文件缓存，按需加载。路由用的 vue-router

&emsp;

#### 一、Vue 组件化开发提升开发体验

&emsp;

1.vue 组件开发

&emsp;&emsp;组件可以扩展 HTML 元素，封装可重用代码，你可以把它当成正常的 HTML 元素使用，而这个元素是你自己定义的。它的表现（css）和行为（js）全都被封装了，调用的方式像是在写正常的 HTML 一样。

生成组件的方式也特别灵活，可以使用 .vue 结尾的文件代表一个组件，也可以使用 Vue.extend({}) 生成。


````js
var Clock = Vue.extend({
  // ...
})

// 全局注册组件
Vue.component('Clock', Clock)

// 在模板中调用
<div id="example">
  <Clock></Clock>
</div>
````

<!--break-->

&emsp;

2.vue 组件动画

&emsp;&emsp;动画原理是组件在显示隐藏或删除添加的过程中改变css的类名，默认是 v-enter, v-leave . 这里的 v 可以改成任意你想要的名字，例如下面改成了 fade.

````js
// html
<div v-if="show" v-transition="fade">hello</div>

// css
.fade-transition {
  transition: all .3s ease;
  height: 30px;
  padding: 10px;
  background-color: #eee;
  overflow: hidden;
}

/* .fade-enter 定义进入的开始状态 */
/* .fade-leave 定义离开的结束状态 */
.fade-enter, .fade-leave {
  height: 0;
  padding: 0 10px;
  opacity: 0;
}
````

有两种方式结合 animate.css 框架

a. 在vue中注册类名

````js
// js
Vue.transition('bounce', {
  enterClass: 'bounceInLeft',
  leaveClass: 'bounceOutRight'
})

// html
<div class="animated" transition="bounce" v-show="show">
````

b. 用 sass 版的 animate.css 的 @mixin 来 @include 

````css
@import "~animate/animate.scss";

/**
 * views transition based on animate.css
 */
.views-transition {
  -webkit-animation-duration: 1s;
  animation-duration: 1s;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both;
}

.views-enter {
  @include fadeInDown(
    $duration: 1s,
    $delay: 0
  );
}

.views-leave {
  @include fadeOutUp(
    $duration: 1s,
    $delay: 0
  );
}
````


&emsp;

#### 二、微信 SDK 开发的经验

&emsp;

*1. 授权的封装*

授权的目的是为了获取 openid, 用户名和图像。

> 微信开发调试阶段特别麻烦，对域名的限制需要在本地配置 host，对 SDK 的依赖需要下载微信自己的工具。而做移动开发需要使用 REM 做到自适应，要对比各种机型下的尺寸，所以我又非常需要 chrome。但是 chrome 底下又没 sdk 的支持。所以我代码一定要写的灵活一点，可以自如的切换场景。

````js
  doAuthReady: function (vm, cb) {
    var code = Utils.getQueryString('code')
    var user = Utils.STORE.getStore(Utils.STORE.user)
    console.log('openid: ' + user.openid)
    // 如果本地存在 openid
    if (user.openid) {
      // 检查服务器有无更新
      this.checkService(user.openid, function (isExist) {
        if (isExist) {
          this.copyUser(vm, user, cb)
        } else {
          Utils.STORE.setStore(Utils.STORE.user, {})
          this.checkAuth()
        }
      }.bind(this))
    } else if (code) {
      // 授权回来拿到 code 去后台获取用户信息
      this.getUserInfo(code).then(function (res) {
        var resUser = res.data.extraInfo
        this.copyUser(vm, resUser, cb)
        Utils.STORE.setStore(Utils.STORE.user, resUser)
      }.bind(this))
    } else {
      // 去授权
      this.checkAuth()
    }
  }
````

在 vue 组件生命周期中调用授权

````js
  ready: function () {
    Utils.WX.doAuthReady(this, weixinfo.bind(this))
  }
````

&emsp;

*2.微信语音调用*

&emsp;&emsp;需求是按住按钮开始录音，松手则结束录音。第一调用语音的时候，会弹出一个框问你是否授权录音。这个弹出的操作就打断了 touchstart 事件，也不会执行 touchend 事件，而是执行了 touchcancel 事件。会影响正常流程。所以需要提前调用录音，提前询问用户是否授权

````js
if (!localStorage.allowRecord) {
  wx.startRecord({
    success: function () {
      localStorage.allowRecord = 'true'
      wx.stopRecord()
    },
    cancel: function () {
      alert('用户拒绝授权录音')
    }
  })
}
````

&emsp;&emsp;另一个问题是在 Android 下，微信调起录音 startRecord 之后无法触发 touchend 事件，这是微信 Android 的一个 bug。所以在 Android 上需要用其它的方式来结束录音，比如点击一下周围其它元素来结束


&emsp;

*3.微信获取图片*

&emsp;&emsp;因为 `input type=file` 大部分 Android 只支持一次只选一张图片。所以调用微信 SDK 来解决，选择一张图片之后上传到微信服务器，再通知后台去微信服务器去下载，这个流程和上传语音是一样的。

&emsp;&emsp;另外微信的图片预览 ios 版本，只能从左往右推出来，不是很流畅，并且没有张数的显示。如果微信不做优化的话，最好自己实现一个图片预览功能。


&emsp;

*4.微信分享*

&emsp;&emsp;不能动态指定分享代码值，必须把值求出来之后，再注册分享代码。


&emsp;

#### 三、效果

<div class="row">
  <div class="span4" align="center" 

  style="margin-top:15px;padding:10px 0;border-radius:8px;border:1px solid #ccc;">
    <img width="90%" src="/assets/images/IMG_9882.jpg">
  </div>
  <div class="span4" align="center" 

  style="margin-top:15px;padding:10px 0;border-radius:8px;border:1px solid #ccc;">
    <img width="90%" src="/assets/images/IMG_9909.jpg">
  </div>
  <div class="span4" align="center" 

  style="margin-top:15px;padding:10px 0;border-radius:8px;border:1px solid #ccc;">
    <img width="90%" src="/assets/images/2.pic.jpg">
  </div>
</div>

&emsp;

#### 四、我的开发环境

<div class="row">
  <div class="span6" align="center" 
  style="margin-top:15px;padding:10px 0;border-radius:8px;border:1px solid #ccc;">
    <img width="90%" src="/assets/images/10.pic_hd.jpg">
  </div>
</div>

