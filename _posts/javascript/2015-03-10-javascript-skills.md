---
layout: post
title: javascript 常用技巧
date: 2015-03-10
categories: javascript
tags: [js, skills]
shortContent: ""
---
{% include JB/setup %}

# JavaScript 常用技巧
----

### !! 双感叹号

作用同 Boolean(value), 总是返回一个 布尔值。

````js
// obj 如果是 null 的话，flag 则为 null
var flag = obj && obj.a === 1;
````

因为 逻辑运算符 && 和 || 并不总是返回 true of false。如果有一个操作数是 对象 的话，有可能返回的是 null

所以要满足 flag 总是一个布尔值要这样做

````js
var flag = !!(obj && obj.a === 1);
````

<!--break-->

----
### ~ 取反操作符

简单的理解，对任一数值 x 进行按位非操作的结果为 -(x + 1)

判断数值中是否有某元素时，以前这样判断：

````js
if(arr.indexOf(ele) > -1){...} //易读
````

现在可以这样判断，两者效率：

````js
if(~arr.indexOf(ele)){...} //简洁
````

那么，双取反 `~~` 就代表  -(-(x+1) + 1)

对于浮点数，~~value可以代替parseInt(value)，而且前者效率更高些

````js
parseInt(-1.1) //-1
~~(-1.1) //-1
parseInt(true) // NaN
+true // 1
~~true // 1
Number(true) // 1
````

> 总结: !! 相当于 Boolean(v), ~~ 相当于 （浮点数）parseInt() / (非浮点数) Number(), + 相当于 Number().

----
### 运算符与函数写在一起，组成函数表达式，而非函数声明。

````javascript

!null(or undefined) //true
!!null(or undefined) //false

var o = {};
var test = !!o.flag // false   等同于 var test = o.flag || false
//结论是：o.flag 取值为 null，undefined，0 等值时，返回结果为 false。

````

* 感叹号，+， function  ：
* 核心是，函数声明不能和函数调用混合在一起使用（会语法报错），
* 运算符 + function(){}() 的写法目的是将 函数声明变为一个表达式。
* 主要考虑的是性能问题

````javascript
+ function() {alert(1)} () // NaN    + 号的作用跟 Number 一样，将数值转变成 number 类型。无法转变时 返回 NaN
! function() {alert(1)} () // true   ！号的作用是 ·取反·
! function a() {alert(1)} () // true , 并且 a 不是函数名，因为这不是函数声明。
(function(){ alert(1) }) () // undefined 函数屋无返回时，默认返回 undefined
(function() {alert1} ()) // undefined
````

---
### 其他

* 银行卡 4 位 空格

````javascript
!function () {
    document.getElementById('bankCard').onkeyup = function (event) {
        var v = this.value;
        //当匹配到 5 个非空字符时。
        if(/\S{5}/.test(v)){
        		//先把空格去掉，然后再每4个任意符后面加一个空格
            this.value = v.replace(/\s/g, '').replace(/(.{4})/g, "$1 ");
        }
    };
}();
````

* 遍历内置对象的所有属性

````javascript
var arrObj = Object.getOwnPropertyNames(Function.prototype);//Array.prototype
for (var funcKey in arrObj) {
	console.log(arrObj[funcKey])
}

//Object.prototype
["constructor", "toString", "toLocaleString", "valueOf", "hasOwnProperty", "isPrototypeOf", "propertyIsEnumerable", "__defineGetter__", "__lookupGetter__", "__defineSetter__", "__lookupSetter__", "__proto__"]

//Function.prototype
["length", "name", "arguments", "caller", "constructor", "bind", "toString", "call", "apply"]

//Array.prototype
["length", "constructor", "toString", "toLocaleString", "join", "pop", "push", "concat", "reverse", "shift", "unshift", "slice", "splice", "sort", "filter", "forEach", "some", "every", "map", "indexOf", "lastIndexOf", "reduce", "reduceRight", "entries", "keys"]

//Boolean.prototype
["constructor", "toString", "valueOf"]

//Number.prototype
["constructor", "toString", "toLocaleString", "valueOf", "toFixed", "toExponential", "toPrecision"]

//String.prototype
["length", "constructor", "valueOf", "toString", "charAt", "charCodeAt", "concat", "indexOf", "lastIndexOf", "localeCompare", "match", "normalize", "replace", "search", "slice", "split", "substring", "substr", "toLowerCase", "toLocaleLowerCase", "toUpperCase", "toLocaleUpperCase", "trim", "trimLeft", "trimRight", "link", "anchor", "fontcolor", "fontsize", "big", "blink", "bold", "fixed", "italics", "small", "strike", "sub", "sup"]

//Date.prototype
["constructor", "toString", "toDateString", "toTimeString", "toLocaleString", "toLocaleDateString", "toLocaleTimeString", "valueOf", "getTime", "getFullYear", "getUTCFullYear", "getMonth", "getUTCMonth", "getDate", "getUTCDate", "getDay", "getUTCDay", "getHours", "getUTCHours", "getMinutes", "getUTCMinutes", "getSeconds", "getUTCSeconds", "getMilliseconds", "getUTCMilliseconds", "getTimezoneOffset", "setTime", "setMilliseconds", "setUTCMilliseconds", "setSeconds", "setUTCSeconds", "setMinutes", "setUTCMinutes", "setHours", "setUTCHours", "setDate", "setUTCDate", "setMonth", "setUTCMonth", "setFullYear", "setUTCFullYear", "toGMTString", "toUTCString", "getYear", "setYear", "toISOString", "toJSON"]
````

* 理解 constructor 构造函数
* new 关键字只能加在 ’function‘ 上，而非 ’object‘

````javascript
function A(name) {
	this.name = name;
}

A.prototype.say = function() {alert(this.name)}

var a = new A('aa');

a; // {name: 'aa'}

typeof a; // 'object'

a.constructor; // function A(name) {this.name = name} 指向 A 本身  既：a.constructor === A

a.constructor.prototype; // {say: function(){alert(this.name)}}  既：a.constructor.prototype === A.prototype === a.__proto__

a.__proto__ === A.prototype // 在ff、chorme中 a.__proto__ === a.constructor.prototype

a.__proto__.__proto__ === Object.prototype   // 原型链循环的最终结果是 Object.prototype . 因为所有对象都继承了Object

typeof A; // 'function'

A.constructor // function Function(){[native code]} 指向的是内置 Function 对象。

a instanceof A // true 因为 a.constructor === A
````

* 读取上传文件的 data-url ，本地预览

````javascript
<input type="file" onchange="previewImage(this)">

function previewImage(file) {
	//chorm, ff
	if (file && file.files[0]) {
		var reader = new FileReader();
		reader.onload = function(evt) {
			img.src = evt.target.result
		}
		reader.readAsDataURL(file.files[0]);
	} else {
		//兼容 ie ，使用滤镜
		var sFilter='filter:progid:DXImageTransform.Microsoft.AlphaImageLoader(sizingMethod=scale,src="';
		file.select();
		var src = document.selection.createRange().text;
		div.innerHTML = '<img id=imghead>';
		var img = document.getElementById('imghead');
		img.filters.item('DXImageTransform.Microsoft.AlphaImageLoader').src = src;
		div.innerHTML = '<div id=divhead style="'+sFilter+src+'\"></div>';
	}
}
````

* 图片预加载
* 思路：先加载一张缩略图，再加载高清图

````javascript
<img src="img_s.jpg" style= "width: 409px; height: 307px;" />//模糊图

var img = new Image();
var url = '../images/img.jpg';
$(img).load(function () {
    $('img').attr('src', url);//下载完了 就赋值给img标签
});
img.src = url;//下载高清图

````

* 图片随 div 大小 等比例展示

````javascript
function autoImageSize(){
	var imageArr=document.getElementById("welcome");
	var maxWidth = $(window).width(),maxHeight = $(window).height();
	var offsetWidth = 1800,offsetHeight = 1000;
  var imageRate = offsetWidth / offsetHeight;
  if(offsetWidth > maxWidth){
  	var h = (maxWidth / imageRate);
    imageArr.style.width=maxWidth + "px";
    imageArr.style.height= (h > maxHeight ? maxHeight : h) + "px";
  }
}
````
