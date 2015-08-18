---
layout: post
title: html5 之 websql 篇
date: 2015-08-18
categories: html5
tags: [html5, websql, project, js]
shortContent: ""
---
{% include JB/setup %}

# html5 之 websql 篇
---

我是在看 36Kr 源码时，看到了对 websql 的使用。刚好自己的项目中也需要用到。

* websql 由于使用了 SQLite 方言，所以 w3c 已经将其搁置了，取而代之的是 NoSQL 类型的 indexeddb。但是在手机移动端本身就支持 SQLite ，特别是低版本的 Android 机。这也使得 websql 成了目前唯一的商用方案。

所以在这里记录下使用心得：

> 使用的是 KenCorbettJr 写的 html5sql.js 。

  - [github 地址](https://github.com/KenCorbettJr/html5sql)
  - 里面介绍了用法和 API。这里就不在赘诉了。

### 一、数据库初始化

````js
  cl.dbReady = function(successCallback, errorCallback) {
    websql.openDatabase('cl', 'cl db', 5 * 1024 * 1024);
    if (websql.database.version === OLD_VERSION_NUMBER) {
      websql.changeVersion(OLD_VERSION_NUMBER, DB_VERSION_NUMBER, SQL_TABLE, function() {
        successCallback && successCallback(true);
      }, function(error, failingQuery) {
        errorCallback && errorCallback(error, failingQuery);
      });
    } else {
      successCallback && successCallback(false);
    }
  };

````

<!--break-->

1. 根据 version 的值来更新数据库结构。

    在开发的时候经常改动表的结构，所以需要判断下版本号。

2. 如果有改动，会将一个布尔值传递给回调函数。

### 二、 定义 SQL

```js
  var SQL_TABLE = 'DROP TABLE IF EXISTS cl_goods;CREATE TABLE cl_goods(goodsId INTEGER PRIMARY KEY,goodsCode Text, barCode TEXT, goodsName TEXT, imgUrl TEXT, ext1 TEXT, ext2 TEXT);';
  var SQL_SELECT = 'SELECT goodsId, goodsCode, barCode, goodsName, imgUrl, ext1, ext2 FROM cl_goods WHERE 1 = 1';
  var SQL_INSERT = 'INSERT INTO cl_goods(goodsId, goodsCode, barCode, goodsName, imgUrl, ext1, ext2) VALUES(?, ?, ?, ?, ?, ?, ?);';
  var SQL_UPDATE = 'UPDATE cl_goods SET imgUrl = ? WHERE goodsId = ?;';
  var SQL_DELETE = 'DELETE FROM cl_goods WHERE 1 = 1;';
  var SQL_SELECT_BY_BARCODE = 'SELECT * FROM cl_goods WHERE barCode = ?';
```

### 三、 增、删、改、查

```js

  cl.getGoodsByBarCode = function(barCode, successCallback, errorCallback) {
    websql.process([{
      "sql": SQL_SELECT_BY_BARCODE,
      "data": [barCode]
    }], function(tx, results) {
      console.log('websql (get goods by barCode):success');
      var goods = {};
      var rows = results.rows;
      goods.goodsName = rows.item(0).goodsName;
      goods.goodsCode = rows.item(0).goodsCode;
      goods.imgUrl = rows.item(0).imgUrl;
      goods.goodsInfo = [];
      for (var i = rows.length - 1; i >= 0; i--) {
        goods.goodsInfo.push({
          goodsId: rows.item(i).goodsId,
          goodsSku: {
            "size": rows.item(i).ext2,
            "color": rows.item(i).ext1
          }
        })
      }
      successCallback && successCallback.call(this, goods);
    }, function(error, failingQuery) {
      console.log('websql (get goods by barCode):error --> ' + error);
      errorCallback && errorCallback.apply(this, arguments);
    })
  };

  cl.getGoods = function(successCallback, errorCallback) {
    websql.process(SQL_SELECT, function(tx, results) {
      console.log('websql (get goods):success');
      successCallback && successCallback.call(this, results.rows);
    }, function(error, failingQuery) {
      console.log('websql (get goods):error --> ' + error);
      errorCallback && errorCallback.apply(this, arguments);
    });
  };

  cl.addGoods = function(goods, successCallback, errorCallback) {
    var sqls = [];
    $.each(goods, function(index, item) {
      sqls.push({
        "sql": SQL_INSERT,
        "data": item
      });
    });
    websql.process(sqls, function(tx, results) {
      successCallback && successCallback(true);
      console.log('websql (add goods):success');
    }, function(error, failingQuery) {
      errorCallback && errorCallback.apply(this, arguments);
      console.log('websql (add goods):error --> ' + error);
    });
  };

  cl.updateGoods = function(quantity, goodsId, successCallback, errorCallback) {
    websql.process([{
      "sql": SQL_UPDATE,
      "data": [quantity, goodsId]
    }], function(tx, results) {
      successCallback && successCallback();
    }, function(error, failingQuery) {
      errorCallback && errorCallback(error, failingQuery);
    })
  };

  cl.deleteGoods = function(successCallback, errorCallback) {
    websql.process(SQL_DELETE, function(tx, results) {
      console.log('websql (delete goods): success');
      successCallback && successCallback();
    }, function(error, failingQuery) {
      console.log('websql (delete goods): error --> ' + failingQuery);
      errorCallback && errorCallback(error, failingQuery);
    });
  };

```
