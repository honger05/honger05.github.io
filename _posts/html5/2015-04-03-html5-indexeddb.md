---
layout: post
title: html5 之 IndexedDB 篇
date: 2015-04-03
categories: html5
tags: [html5, indexeddb, project]
shortContent: ""
---
{% include JB/setup %}

#### IndexedDB 概述

##### 浏览器端数据存储方案

- cookie 小于4kb，且每次请求都会发送给服务器端。
- window.name 内容少，操作不方便，不安全
- localStorage 2.5mb - 10mb 之间，不同浏览器不同。

##### IndexdDB 特点：

- 键值对存储： 使用对象仓库存储数据 -- object store
- 异步：任何对 indexedDB 的操作都是异步的， localStorage 是同步的。
- 支持事务： transaction 对数据库的CRUD必须在事务中进行。
- 同域限制: 每个数据库都对应产生该数据库的域名，来自不同域名下的网页，只能访问自身域名下的数据库。
- 存储空间大：一般来说不少于250mb。ie上限是250mb，chorm、opera 是剩余空间的某个百分比。firefox 则没有上限。
- 支持二进制存储

##### 浏览器支持情况

![支持情况][indexeddb]

[indexeddb]: /assets/images/indexeddb.png
