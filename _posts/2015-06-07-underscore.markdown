---
layout: post
title: underscore 源码解析
date: 2015-06-07 22:58:51
categories: code javascript resource
---

###1. 针对于宿主环境

{% highlight bash linenos %}
//node环境
if (typeof exports !== 'undefined') {
  if (typeof module !== 'undefined' && module.exports) {
    exports = module.exports = _;
  }
  exports._ = _;
} else {
  //浏览器环境
  root['_'] = _;
}
{% endhighlight %}

因为现在基于模块开发已是大势所趋。所以这里需要加上一段代码，以供模块加载

从服务器端的`commonjs`规范，到浏览器端的践行者`AMD`、`CMD`规范。

兼容浏览器端的模块加载器（require & seajs） 的写法, 以 `CMD` 为例
{% highlight javascript %}
if (typeof define !== 'undefined' && define.cmd) {
  define('/path/underscore',function() {
    return _;
  })
}
{% endhighlight %}

最近spm社区也是极力推荐使用commonjs的语法风格，去cmd的语法风格，以保证代码不用改来改去。

---
###2. 一些灵活的代码
{% highlight javascript %}
//判断obj是 数组 还是 对象
if (obj.length === +obj.length) {
  //......
}
{% endhighlight %}
因为对象的length是`undefined`，而 `+obj.length` 的作用跟 `Number(obj.length)` 是一样的，
得出的结果是NaN。 `+` 的用法，我在 *Arale* 库里面也看到了，可能是写起来方便和cool，所以在库中用的比较多。









