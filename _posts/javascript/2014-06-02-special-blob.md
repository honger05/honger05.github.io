---
layout: post
title:  文件类型之 Blob 对象
date: 2014-06-02
categories: javascript
tags: [js, skills]
shortContent: ""
---
{% include JB/setup %}

# 文件类型之 Blob 对象
---

#### Blob

---

一个Blob对象就是一个包含有只读原始数据的类文件对象.

* Blob对象中的数据并不一定得是JavaScript中的原生形式.
* File接口基于Blob,继承了Blob的功能,并且扩展支持了用户计算机上的本地文件.

##### 创建Blob对象

1. 调用Blob构造函数.
2. 使用一个已有Blob对象上的slice()方法切出另一个Blob对象.
3. 调用canvas对象上的toBlob方法.

<!--break-->

*注: 需要注意的是,一些浏览器上的slice()方法仍带有前缀:Firefox 12之前的版本上为blob.mozSlice(),Safari上为blob.webkitSlice().*

*注: 一些浏览器提供了BlobBuilder接口,但并不是所有的浏览器都支持BlobBuilder,而且现有的BlobBuilder实现都是带前缀的.更主要的是BlobBuilder已经被废弃,你应该尽可能的使用Blob构造函数来代替.*

##### 构造函数

````javascript
Blob Blob(
  [optional] Array parts,
  [optional] BlobPropertyBag properties
);
````

1. parts：一个数组，包含了将要添加到Blob对象中的数据.数组元素可以是任意多个的ArrayBuffer,ArrayBufferView (typed array), Blob,或者 DOMString对象.

2. properties：一个对象，设置Blob对象的一些属性。

##### Blob 构造函数用法举例：

````javascript
//下面的代码:

var aFileParts = ["<a id=\"a\"><b id=\"b\">hey!<\/b><\/a>"];
var oMyBlob = new Blob(aFileParts, { "type" : "text\/xml" }); // the blob
//等价于:

var oBuilder = new BlobBuilder();
var aFileParts = ["<a id=\"a\"><b id=\"b\">hey!<\/b><\/a>"];
oBuilder.append(aFileParts[0]);
var oMyBlob = oBuilder.getBlob("text\/xml"); // the blob
````

*BlobBuilder接口提供了另外一种创建Blob对象的方式,但该方式现在已经废弃,所以不应该再使用了.*

##### 使用类型数组和Blob对象创建一个对象URL

* 你可以像使用一个普通URL那样使用它,比如用在img.src上。
* 应用在有 download 属性的 A 标签上，将下载该文件。

````javascript
var typedArray = GetTheTypedArraySomehow();
var blob = new Blob([typedArray], {type: "application/octet-binary"}); // 传入一个合适的MIME类型
var url = URL.createObjectURL(blob);
// 会产生一个类似blob:d3958f5c-0777-0845-9dcf-2cb28783acaf这样的URL字符串
````


