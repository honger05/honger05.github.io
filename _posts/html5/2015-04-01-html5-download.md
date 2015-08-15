---
layout: post
title: html5 之 a标签 download 篇
date: 2015-04-01
categories: html5
tags: [html5, file, project]
shortContent: "在前端即可下载数据了。"
---
{% include JB/setup %}
# html5 之 a标签 download 篇
---

* download 指定下载的文件名
* 内容为 ’download 测试数据‘

````javascript
<a download="download.txt" href="data:text/txt;charset=utf-8,download 测试数据">download</a>
````

* 用 wps 打开不会乱码，但用 office 打开会乱码，
* 解决办法是，在数据前面加上

````javascript
<a onclick="clickDownload(this)" download="downlaod.csv" href="#">中文不乱码</a>

function clickDownload(aLink) {
   var str = "栏位1,栏位2,栏位3\n值1,值2,值3";
   str =  encodeURIComponent(str);
   //防止excel打开中文乱码\ufeff
   aLink.href = "data:text/csv;charset=utf-8,\ufeff"+str;
   aLink.click();
}
````

<!--break-->

* 大数据时，利用 Blob 对象, URL.createObjectURL(blob) 产生一个可以下载的 url

````javascript
<a id="mylink" href="">download blob</a>

!function() {
	var data = '栏位1,栏位2,栏位3\n值1,值2,值3'
  //防止excel打开中文乱码\ufeff
	data = "\ufeff"+data;
  var blob = new Blob([data], { type: 'text/csv' }); //new way
  var csvUrl = URL.createObjectURL(blob);
  var mylink = document.getElementById("mylink");
  mylink.href = csvUrl;
  mylink.setAttribute("download", "aaa.csv")
}();
````
