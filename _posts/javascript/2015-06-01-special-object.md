---
layout: post
title: javascript 中的一些特殊对象
date: 2015-06-01
categories: javascript
tags: [js, skills]
shortContent: ""
---
{% include JB/setup %}

#### FormData

---

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
  contentType:"multipart/form-data",
  data: formData,
  processData: false,
  type: 'POST'
});
````

#### Blob

---



#### ArrayBuffer

---