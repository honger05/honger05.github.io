---
layout: post
title: XMLHttpRequest level2 之 FormData
date: 2015-06-01
categories: javascript
tags: [js, skills]
shortContent: ""
---
{% include JB/setup %}

#### FormData

---

> XMLHttpRequest 第二版添加了对新接口 FormData 的支持。FormData 可以很方便地将表单字段和它们的值建立成键和值对应的成对形式，然后通过 XMLHttpRequest 的 sent() 方法发送表格数据。

* 通常我们提交（使用submit button）时，会把form中的所有表格元素的name与value组成一个queryString，提交到后台。这用jQuery的方法来说，就是serialize
* 但当我们使用Ajax提交时，这过程就要变成人工的了。因此，FormData对象的出现可以减少我们一些工作量。

````javascript
// 1、通过 append 添加
var formdata = new FormData();
formData.append('file', $('input[name=file]', this).get(0).files[0]);
formData.append('startX', $('#startX').val());

// 2、将 form 元素对象作参数 
var formobj =  document.getElementById("form");
var formdata = new FormData(formobj);

// 3、form 元素对象 getFormData 方法生成
var formobj =  document.getElementById("form");
var formdata = formobj.getFormData()
````

* 利用 FormData 、ajax 上传文件

````javascript
var formdata = new FormData();
formData.append('file', $('input[name=file]', this).get(0).files[0]);

$.ajax({
  url: '/upload.do',
  contentType: false, //告诉jQuery不要去设置Content-Type请求头，因为默认会加上 boundary 的。
  data: formData,
  processData: false, //告诉jQuery不要处理发送的数据。
  type: 'POST'
});
````

* 传统的上传方式：

````javascript
// 为了避免跳转，会在页面放一个隐藏的iframe，指定form的target为iframe
<form id="testForm" method="post" action="xxxx.do" target="testIframe">
   <input type="text" name="input1" value="test1"/>
   <input type="text" name="input2" value="test2"/>
   <input type="file" name="uploadfile">
   <input type="submit"/>
</form>

<iframe name="testIframe" style="display:none"></iframe>
````


