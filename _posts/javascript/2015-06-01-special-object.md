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


#### Blob

---

一个Blob对象就是一个包含有只读原始数据的类文件对象.

* Blob对象中的数据并不一定得是JavaScript中的原生形式.
* File接口基于Blob,继承了Blob的功能,并且扩展支持了用户计算机上的本地文件.

##### 创建Blob对象

1. 调用Blob构造函数.
2. 使用一个已有Blob对象上的slice()方法切出另一个Blob对象.
3. 调用canvas对象上的toBlob方法.

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


#### ArrayBuffer 类型化数组

---

> 类型化数组是JavaScript操作二进制数据的一个接口。

> 这要从WebGL项目的诞生说起，所谓WebGL，就是指浏览器与显卡之间的通信接口，为了满足JavaScript与显卡之间大量的、实时的数据交换，它们之间的数据通信必须是二进制的，而不能是传统的文本格式。

> 比如，以文本格式传递一个32位整数，两端的JavaScript脚本与显卡都要进行格式转化，将非常耗时。这时要是存在一种机制，可以像C语言那样，直接操作字节，然后将4个字节的32位整数，以二进制形式原封不动地送入显卡，脚本的性能就会大幅提升。

> 类型化数组（Typed Array）就是在这种背景下诞生的。它很像C语言的数组，允许开发者以数组下标的形式，直接操作内存。有了类型化数组以后，JavaScript的二进制数据处理功能增强了很多，接口之间完全可以用二进制数据通信。

##### 分配内存

类型化数组是建立在ArrayBuffer对象的基础上的。它的作用是，分配一段可以存放数据的连续内存区域。

````javascript
var buf = new ArrayBuffer(32);
//ArrayBuffer 对象的 byteLength 属性，返回所分配的内存区域的字节长度。
buffer.byteLength // 32
````
##### 视图

ArrayBuffer作为内存区域，可以存放多种类型的数据。不同数据有不同的存储方式，这就叫做“视图”。目前，JavaScript提供以下类型的视图：

* Int8Array：8位有符号整数，长度1个字节。
* Uint8Array：8位无符号整数，长度1个字节。
* Int16Array：16位有符号整数，长度2个字节。
* Uint16Array：16位无符号整数，长度2个字节。
* Int32Array：32位有符号整数，长度4个字节。
* Uint32Array：32位无符号整数，长度4个字节。
* Float32Array：32位浮点数，长度4个字节。
* Float64Array：64位浮点数，长度8个字节。

每一种视图都有一个BYTES_PER_ELEMENT常数，表示这种数据类型占据的字节数。

* Int8Array.BYTES_PER_ELEMENT // 1
* Uint8Array.BYTES_PER_ELEMENT // 1
* Int16Array.BYTES_PER_ELEMENT // 2
* Uint16Array.BYTES_PER_ELEMENT // 2
* Int32Array.BYTES_PER_ELEMENT // 4
* Uint32Array.BYTES_PER_ELEMENT // 4
* Float32Array.BYTES_PER_ELEMENT // 4
* Float64Array.BYTES_PER_ELEMENT // 8

每一种视图都是一个构造函数，有多种方法可以生成：

1. 在ArrayBuffer对象之上生成视图。

同一个ArrayBuffer对象之上，可以根据不同的数据类型，建立多个视图。

````javascript
// 创建一个8字节的ArrayBuffer
var b = new ArrayBuffer(8);

// 创建一个指向b的Int32视图，开始于字节0，直到缓冲区的末尾
var v1 = new Int32Array(b);

// 创建一个指向b的Uint8视图，开始于字节2，直到缓冲区的末尾
var v2 = new Uint8Array(b, 2);

// 创建一个指向b的Int16视图，开始于字节2，长度为2
var v3 = new Int16Array(b, 2, 2);
````

2. 直接生成

视图还可以不通过ArrayBuffer对象，直接分配内存而生成。

````javascript
var f64a = new Float64Array(8);
f64a[0] = 10;
f64a[1] = 20;
f64a[2] = f64a[0] + f64a[1];
````

3. 将普通数组转为视图数组

````javascript
var typedArray = new Uint8Array( [ 1, 2, 3, 4 ] );
//上面代码将一个普通的数组，赋值给一个新生成的8位无符号整数的视图数组。

//视图数组也可以转换回普通数组。

var normalArray = Array.apply( [], typedArray );
````

##### 视图的操作

1. 数组操作

普通数组的操作方法和属性，对类型化数组完全适用。

````javascript
var buffer = new ArrayBuffer(16);

var int32View = new Int32Array(buffer);

for (var i=0; i < int32View.length; i++) {
  int32View[i] = i*2;
}
````

2. buffer属性

类型化数组的buffer属性，返回整段内存区域对应的ArrayBuffer对象。该属性为只读属性。

````javascript
// a对象和b对象，对应同一个ArrayBuffer对象，即同一段内存
var a = new Float32Array(64);
var b = new Uint8Array(a.buffer);
````

3. byteLength属性和byteOffset属性

byteLength属性返回类型化数组占据的内存长度，单位为字节。byteOffset属性返回类型化数组从底层ArrayBuffer对象的哪个字节开始。这两个属性都是只读属性。

````javascript
var b = new ArrayBuffer(8);

var v1 = new Int32Array(b);
var v2 = new Uint8Array(b, 2);
var v3 = new Int16Array(b, 2, 2);

v1.byteLength // 8
v2.byteLength // 6
v3.byteLength // 4

v1.byteOffset // 0
v2.byteOffset // 2
v3.byteOffset // 2
````

注意将byteLength属性和length属性区分，前者是字节长度，后者是成员长度

````javascript
var a = new Int16Array(8);

a.length // 8
a.byteLength // 16
````

4. set方法

类型化数组的set方法用于复制数组，也就是将一段内容完全复制到另一段内存。

````javascript
var a = new Uint8Array(8);
var b = new Uint8Array(8);
var c = new Uint8Array(8);

b.set(a);
//从第二个开始
c.set(a,2);
````

5. subarray方法

subarray方法是对于类型化数组的一部分，再建立一个新的视图

````javascript
var a = new Uint16Array(8);
var b = a.subarray(2,3);

a.byteLength // 16
b.byteLength // 2
````

6. ArrayBuffer与字符串的互相转换

ArrayBuffer转为字符串，或者字符串转为ArrayBuffer，有一个前提，即字符串的编码方法是确定的。假定字符串采用UTF-16编码（JavaScript的内部编码方式），可以自己编写转换函数。

````javascript
// ArrayBuffer转为字符串，参数为ArrayBuffer对象
function ab2str(buf) {
   return String.fromCharCode.apply(null, new Uint16Array(buf));
}

// 字符串转为ArrayBuffer对象，参数为字符串
function str2ab(str) {
    var buf = new ArrayBuffer(str.length*2); // 每个字符占用2个字节
    var bufView = new Uint16Array(buf);
    for (var i=0, strLen=str.length; i < strLen; i++) {
         bufView[i] = str.charCodeAt(i);
    }
    return buf;
}
````

##### 应用

1. Ajax

传统上，服务器通过Ajax操作只能返回文本数据。XMLHttpRequest 第二版允许服务器返回二进制数据，这时分成两种情况。如果明确知道返回的二进制数据类型，可以把返回类型（responseType）设为arraybuffer；如果不知道，就设为blob。

````javascript
xhr.responseType = 'arraybuffer';
````

如果知道传回来的是32位整数，可以像下面这样处理。

````javaascript
xhr.onreadystatechange = function () {
if (req.readyState === 4 ) {
    var arrayResponse = xhr.response;
    var dataView = new DataView(arrayResponse);
    var ints = new Uint32Array(dataView.byteLength / 4);

    xhrDiv.style.backgroundColor = "#00FF00";
    xhrDiv.innerText = "Array is " + ints.length + "uints long";
    }
}
````
2. Canvas

网页Canvas元素输出的二进制像素数据，就是类型化数组。

````javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

var imageData = ctx.getImageData(0,0, 200, 100);
var typedArray = imageData.data;
````
需要注意的是，上面代码的typedArray虽然是一个类型化数组，但是它的视图类型是一种针对Canvas元素的专有类型Uint8ClampedArray。这个视图类型的特点，就是专门针对颜色，把每个字节解读为无符号的8位整数，即只能取值0～255，而且发生运算的时候自动过滤高位溢出。这为图像处理带来了巨大的方便。

举例来说，如果把像素的颜色值设为Uint8Array类型，那么乘以一个gamma值的时候，就必须这样计算：

u8[i] = Math.min(255, Math.max(0, u8[i] * gamma));
因为Uint8Array类型对于大于255的运算结果（比如0xFF+1），会自动变为0x00，所以图像处理必须要像上面这样算。这样做很麻烦，而且影响性能。如果将颜色值设为Uint8ClampedArray类型，计算就简化许多。

pixels[i] *= gamma;
Uint8ClampedArray类型确保将小于0的值设为0，将大于255的值设为255。注意，IE 10不支持该类型。

3. File

如果知道一个文件的二进制数据类型，也可以将这个文件读取为类型化数组。

reader.readAsArrayBuffer(file);
下面以处理bmp文件为例。假定file变量是一个指向bmp文件的文件对象，首先读取文件。

var reader = new FileReader();
reader.addEventListener("load", processimage, false); 
reader.readAsArrayBuffer(file);
然后，定义处理图像的回调函数：先在二进制数据之上建立一个DataView视图，再建立一个bitmap对象，用于存放处理后的数据，最后将图像展示在canvas元素之中。

function processimage(e) { 
 var buffer = e.target.result; 
 var datav = new DataView(buffer); 
 var bitmap = {};
 // 具体的处理步骤
}
具体处理图像数据时，先处理bmp的文件头。具体每个文件头的格式和定义，请参阅有关资料。

bitmap.fileheader = {}; 
bitmap.fileheader.bfType = datav.getUint16(0, true); 
bitmap.fileheader.bfSize = datav.getUint32(2, true); 
bitmap.fileheader.bfReserved1 = datav.getUint16(6, true); 
bitmap.fileheader.bfReserved2 = datav.getUint16(8, true); 
bitmap.fileheader.bfOffBits = datav.getUint32(10, true);
接着处理图像元信息部分。

bitmap.infoheader = {};
bitmap.infoheader.biSize = datav.getUint32(14, true);
bitmap.infoheader.biWidth = datav.getUint32(18, true); 
bitmap.infoheader.biHeight = datav.getUint32(22, true); 
bitmap.infoheader.biPlanes = datav.getUint16(26, true); 
bitmap.infoheader.biBitCount = datav.getUint16(28, true); 
bitmap.infoheader.biCompression = datav.getUint32(30, true); 
bitmap.infoheader.biSizeImage = datav.getUint32(34, true); 
bitmap.infoheader.biXPelsPerMeter = datav.getUint32(38, true); 
bitmap.infoheader.biYPelsPerMeter = datav.getUint32(42, true); 
bitmap.infoheader.biClrUsed = datav.getUint32(46, true); 
bitmap.infoheader.biClrImportant = datav.getUint32(50, true);
最后处理图像本身的像素信息。

var start = bitmap.fileheader.bfOffBits;
bitmap.pixels = new Uint8Array(buffer, start);
至此，图像文件的数据全部处理完成。下一步，可以根据需要，进行图像变形，或者转换格式，或者展示在Canvas网页元素之中。