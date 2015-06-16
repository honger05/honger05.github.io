---
layout: post
title: gulp 常用工具包
date: 2015-01-08
categories: tools
tags: [gulp, tools, seajs]
shortContent: ""
---
{% include JB/setup %}

#### 常用包

---

> 总结，若是 seajs 项目，最好是用 spm 来构建。但也并不完美，所以想要完美构建，就放弃 seajs 吧！

##### 1. gulp-seajs-transport 

---

这个包最大的作用是会通过 src 路径，来提取模块 id。

````javascript
var gulp = require("gulp");
var transport = require("gulp-seajs-transport");

gulp.task("default", function() {
	gulp.src("./test/**/*.js")
			.pipe(transport())
			.pipe(gulp.dest("./dist"))
})

//相对路径提取
gulp.task("default",function() {
  gulp.src("./testfiles/abc/def/ggg.js",{base:"./testfiles/abc"})
        .pipe(transport()) //此时seajs模块id为=>def/ggg
        .pipe(gulp.dest("./dist"));
````

##### 2. 遍历目录

````javascript
var gulp = require('gulp');
var transport = require('gulp-seajs-transport');
var fs = require('fs');
var path = require('path');

var scriptsPath = 'asset/';

function getFolders(dir) {
    return fs.readdirSync(dir)
        .filter(function(file) {
            return fs.statSync(path.join(dir, file)).isDirectory();
        });
}

gulp.task('default', function() {
    var folders = getFolders(scriptsPath);

    var tasks = folders.map(function(folder) {

        var src = path.join(scriptsPath, folder, '/src/*.js');
        var dest = './build/'+folder;
        var tmp = './tmp/'+folder;

        gulp.src(src)
            .pipe(transport())
            .pipe(concat(folder+'.js'))
            .pipe(gulp.dest(dest));
            
    });
});
````

##### 3. merge-stream 合并流

* 因为 gulp 是管道（pipe）思想，在有些情况下，比如循环，必须把流合并了再 return

````javascript
var mergeStream = require('merge-stream');

var stream1 = new Stream();
var stream2 = new Stream();
 
var merged = mergeStream(stream1, stream2);
 
var stream3 = new Stream();
merged.add(stream3)
````

* 删除文件以及目录 `$ npm install --save del`

````javascript
var del = require('del');
 
del(['tmp/*.js', '!tmp/unicorn.js'], function (err, paths) {
    console.log('Deleted files/folders:\n', paths.join('\n'));
});
````

#### 以下包均从网络上来，用来记录下用法。

> 以下包均可通过其它更好的方式实现，在此仅记录 gulp 用法

---

##### 1. gulp-less

---

* 编译多个less文件

````javascript
var gulp = require('gulp'),
    less = require('gulp-less');
 
gulp.task('testLess', function () {
    gulp.src(['src/less/index.less','src/less/detail.less']) //多个文件以数组形式传入
        .pipe(less())
        .pipe(gulp.dest('src/css')); //将会在src/css下生成index.css以及detail.css 
});
````

* 调用多模块（编译less后压缩css）

````javascript
var gulp = require('gulp'),
    less = require('gulp-less'),
     //确保本地已安装gulp-minify-css [npm install gulp-minfy-css --save-dev]
    cssmin = require('gulp-minify-css');
 
gulp.task('testLess', function () {
    gulp.src('src/less/index.less')
        .pipe(less())
        .pipe(cssmin()) //兼容IE7及以下需设置compatibility属性 .pipe(cssmin({compatibility: 'ie7'}))
        .pipe(gulp.dest('src/css'));
});
````

> 对于 sublime 用户来说，不用那么麻烦，因为有编译 less 的插件

* 若每修改一次less，就要手动执行任务，显然是不合理的，所以当有less文件发生改变时使其自动编译

````javascript
var gulp = require('gulp'),
    less = require('gulp-less');
 
gulp.task('testLess', function () {
    gulp.src(['src/less/*.less','!src/less/extend/{reset,test}.less'])
        .pipe(less())
        .pipe(gulp.dest('src/css'));
});
 
gulp.task('testWatch', function () {
    gulp.watch('src/**/*.less', ['testLess']); //当所有less文件发生改变时，调用testLess任务
});
````

* 当编译less时出现语法错误或者其他异常，会终止watch事件，通常需要查看命令提示符窗口才能知道，这并不是我们所希望的，所以我们需要处理出现异常并不终止watch事件（gulp-plumber），并提示我们出现了错误（gulp-notify）。

````javascript
var gulp = require('gulp'),
    less = require('gulp-less');
    //当发生异常时提示错误 确保本地安装gulp-notify和gulp-plumber
    notify = require('gulp-notify'),
    plumber = require('gulp-plumber');
 
gulp.task('testLess', function () {
    gulp.src('src/less/*.less')
        .pipe(plumber({errorHandler: notify.onError('Error: <%= error.message %>')}))
        .pipe(less())
        .pipe(gulp.dest('src/css'));
});
gulp.task('testWatch', function () {
    gulp.watch('src/**/*.less', ['testLess']);
});
````

##### 2. gulp-minify-css

---

````javascript
var gulp = require('gulp'),
    cssmin = require('gulp-minify-css');
 
gulp.task('testCssmin', function () {
    gulp.src('src/css/*.css')
        .pipe(cssmin({
            advanced: false,//类型：Boolean 默认：true [是否开启高级优化（合并选择器等）]
            compatibility: 'ie7',//类型：String 默认：''or'*' [启用兼容模式； 'ie7'：IE7兼容模式，'ie8'：IE8兼容模式，'*'：IE9+兼容模式]
            keepBreaks: true//类型：Boolean 默认：false [是否保留换行]
        }))
        .pipe(gulp.dest('dist/css'));
});
````

* 给css文件里引用url加版本号（根据引用文件的md5生产版本号），像这样

`.bc {background:url(../img/a.png?v=sjkdrjekrh)}`

````javascript
var gulp = require('gulp'),
    cssmin = require('gulp-minify-css');
    //确保已本地安装gulp-make-css-url-version [npm install gulp-make-css-url-version --save-dev]
    cssver = require('gulp-make-css-url-version'); 
 
gulp.task('testCssmin', function () {
    gulp.src('src/css/*.css')
        .pipe(cssver()) //给css文件里引用文件加版本号（文件MD5）
        .pipe(cssmin())
        .pipe(gulp.dest('dist/css'));
});
````

##### 3. gulp-rev-append

- gulp-rev-append 插件将通过正则(?:href|src)=”(.*)[\?]rev=(.*)[\”]查找并给指定链接填加版本号（默认根据文件MD5生成，因此文件未发生改变，此版本号将不会变）

````javascript
<!doctype html>
<html>
  <head>
    <link rel="stylesheet" href="css/style.css?rev=@@hash">
    <script src="js/js-one.js?rev=@@hash"></script>
    <script src="js/js-two.js"></script>
  </head>
  <body>
    <div>hello, world!</div>
    <img src="img/test.jpg?rev=@@hash" alt="" />
    <script src="js/js-three.js?rev=@@hash"></script>
  </body>
</html>
````

* 基本使用（给页面引用url添加版本号，以清除页面缓存）

````javascript
var gulp = require('gulp'),
    rev = require('gulp-rev-append');
 
gulp.task('testRev', function () {
    gulp.src('src/html/index.html')
        .pipe(rev())
        .pipe(gulp.dest('dist/html'));
});
````

##### 4. gulp-htmlmin

````javascript
var gulp = require('gulp'),
    htmlmin = require('gulp-htmlmin');
 
gulp.task('testHtmlmin', function () {
    var options = {
        removeComments: true,//清除HTML注释
        collapseWhitespace: true,//压缩HTML
        collapseBooleanAttributes: true,//省略布尔属性的值 <input checked="true"/> ==> <input />
        removeEmptyAttributes: true,//删除所有空格作属性值 <input id="" /> ==> <input />
        removeScriptTypeAttributes: true,//删除<script>的type="text/javascript"
        removeStyleLinkTypeAttributes: true,//删除<style>和<link>的type="text/css"
        minifyJS: true,//压缩页面JS
        minifyCSS: true//压缩页面CSS
    };
    gulp.src('src/html/*.html')
        .pipe(htmlmin(options))
        .pipe(gulp.dest('dist/html'));
});
````