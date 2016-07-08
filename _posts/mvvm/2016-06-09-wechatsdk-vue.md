---
layout: post
title: 微信SDK + vue + webpack 实践
date: 2016-06-09
categories: ionic
tags: [ionic, webapp, webpack, angular]
---
{% include JB/setup %}

# 微信SDK + vue + webpack 实践
---

#### 一、Vue 组件化开发提升开发体验

> 我是一个特别注重开发体验的人，我坚信注重开发体验，才会把用户体验做好。

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

#### 二、微信 SDK 开发的经验

1. 授权的封装

授权的目的是为了获取 openid, 用户名和图像。微信


&emsp;

#### 三、效果

<div class="row">
  <div class="span4" align="center" 

  style="margin-top:15px;padding:10px 0;border-radius:8px;border:1px solid #ccc;">
    <img width="90%" src="/assets/images/IMG_9882.jpg">
  </div>
  <div class="span4" align="center" 

  style="margin-top:15px;padding:10px 0;border-radius:8px;border:1px solid #ccc;">
    <img width="90%" src="/assets/images/1.pic.jpg">
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

