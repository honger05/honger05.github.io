---
layout: post
title:  二进制数据之 TypeArray 对象
date: 2015-06-03
categories: javascript
tags: [js, skills]
shortContent: ""
---
{% include JB/setup %}

# 二进制数据之 TypeArray 对象
---

#### ArrayBuffer 类型化数组

---

> 内容摘自网络

> 类型化数组是JavaScript操作二进制数据的一个接口。

> 这要从WebGL项目的诞生说起，所谓WebGL，就是指浏览器与显卡之间的通信接口，为了满足JavaScript与显卡之间大量的、实时的数据交换，它们之间的数据通信必须是二进制的，而不能是传统的文本格式。

> 比如，以文本格式传递一个32位整数，两端的JavaScript脚本与显卡都要进行格式转化，将非常耗时。这时要是存在一种机制，可以像C语言那样，直接操作字节，然后将4个字节的32位整数，以二进制形式原封不动地送入显卡，脚本的性能就会大幅提升。

> 类型化数组（Typed Array）就是在这种背景下诞生的。它很像C语言的数组，允许开发者以数组下标的形式，直接操作内存。有了类型化数组以后，JavaScript的二进制数据处理功能增强了很多，接口之间完全可以用二进制数据通信。

<!--break-->

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

````javascript
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


3. File

如果知道一个文件的二进制数据类型，也可以将这个文件读取为类型化数组。

````javascript
reader.readAsArrayBuffer(file);
//下面以处理bmp文件为例。假定file变量是一个指向bmp文件的文件对象，首先读取文件。

var reader = new FileReader();
reader.addEventListener("load", processimage, false);
reader.readAsArrayBuffer(file);
//然后，定义处理图像的回调函数：先在二进制数据之上建立一个DataView视图，再建立一个bitmap对象，用于存放处理后的数据，最后将图像展示在canvas元素之中。

function processimage(e) {
 var buffer = e.target.result;
 var datav = new DataView(buffer);
 var bitmap = {};
 // 具体的处理步骤
}
````

具体处理图像数据时，先处理bmp的文件头。具体每个文件头的格式和定义，请参阅有关资料。

