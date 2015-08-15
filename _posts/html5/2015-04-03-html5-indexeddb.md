---
layout: post
title: html5 之 IndexedDB 篇
date: 2015-04-03
categories: html5
tags: [html5, indexeddb, project, js]
shortContent: ""
---
{% include JB/setup %}

# html5 之 IndexedDB 篇
---

### 一、IndexedDB 概述
---

##### 1. 浏览器端数据存储方案


- cookie 小于4kb，且每次请求都会发送给服务器端。
- window.name 内容少，操作不方便，不安全
- localStorage 2.5mb - 10mb 之间，不同浏览器不同。

##### 2. IndexdDB 特点：

- 键值对存储： 使用对象仓库存储数据 -- object store
- 异步：任何对 indexedDB 的操作都是异步的， localStorage 是同步的。
- 支持事务： transaction 对数据库的CRUD必须在事务中进行。
- 同域限制: 每个数据库都对应产生该数据库的域名，来自不同域名下的网页，只能访问自身域名下的数据库。
- 存储空间大：一般来说不少于250mb。ie上限是250mb，chorm、opera 是剩余空间的某个百分比。firefox 则没有上限。
- 支持二进制存储

##### 3. 浏览器支持情况

![支持情况][indexeddb]

[indexeddb]: /assets/images/indexeddb.png

<!--break-->

### 二、项目中使用情况
---


将基础数据存储在浏览器中，数据较稳定，不常更新。主键是 表名 bb

创建一个单例缓存管理类

```javascript
requrie('indexeddb')($);

function CacheManager() {

	//数据库存在的话就打开连接，不存在的话就创建数据库
	this.dbPromies = $.indexedDB(dbname, {
		'schema': {
			'2': function(transaction) {
				transaction.createObjectStore('cachebb', {'keyPath': 'bb'});
			}
		}
	})

}

CacheManager.prototype = {

	refresh: function() {

		//拿到 objectStore 对象
		var cachebbStore = this.dbPromies.objectStore('cachebb');

		//清除所有数据  -- 异步的方法
		var storePromies = cachebbStore.clear();

		//清除成功
		storePromies.done(function() {
			//更新数据 -- keyPath一样时
			cachebbStore.put(data);
			//添加数据
			cachebbStore.add(data)
		});
	},

	//根据主键查询
	query: function(bb) {
		return $.Deferred(function(dfd) {
			//拿到 objectStore 对象
			var cachebbStore = this.dbPromies.objectStore('cachebb');

			cachebbStore.get(bb).done(function(result, event) {
				dfd.resolve(result.records);
			}).fail(function(error, event) {
				dfd.reject(error);
		  })

		})
	},

	deleteDB: function(dbname, callback){
			$.indexedDB(dbname).deleteDatabase().done(function() {
				isFunction(callback) || callback.call();
			})
	},


}

return new CacheManager();
```



### 三、使用 jquery-indexedDB 插件
---


详细介绍 -- [jquery-indexedDB](https://github.com/axemclion/jquery-indexeddb/blob/master/docs/README.md)

##### 1. 打开数据库 (Open Database)

打开一个indexeddb数据库。不存在时，根据 schema 创建一个数据库。

```javascript

var dbOpenPromise = $.indexedDB('database_name', {
	'version': 3,  //打开这个版本的数据库
	'upgrade': function(transaction) {
		//版本更新时，这个方法被调用
	},
	'schema': {
		'1': function(transaction) {
			//1、2 表示版本号，版本更新时，数值只能增不能减
		},
		'2': function(transaction) {
			//使用事务对象的例子
			var obj1 = transaction.objectStore('store1');
			var obj2 = transaction.createObjectStore(store2);
			obj1.createIndex('index')
		}
	}
})

```

* 连接打开完成
* 连接正在打开
* 连接打开失败

```javascript
dbOpenPromise.done(function(db, event) {
	db; // 原生的indexeddb数据库对象
	event; // IndexdbDB Event 对象
	this; // 原生 IDBRequest 函数内的上下文
})

dbOpenPromise.progress(function(db, event) {
	error; // 错误对象
	event; // IndexdbDB Event 对象
	event.type; // 包括 blocked . upgrade
	this; // 原生 IDBRequest 函数内的上下文
})

dbOpenPromise.fail(function(error, event){
    error; // 错误对象
    event; // IndexdbDB Event 对象
    this; // Context inside the function is the native IDBRequest
});
```

##### 2. 打开一个事务（Transactions）

在数据库存在的情况下，不需要先打开数据库链接，直接使用下面的方法打开事务即可。

数据库不存在时，操作不成功

```javascript
//打开一个事务
var transactionPromise = $.indexedDB("dbName").transaction(storeNames, mode);

//事务在进行中的状态时，才能对 ObjectStore 进行操作
transactionPromise.progress(function(trans){
	//打开一个 ObjectStore
	var objectStore = trans.objectStore("objectStoreName");

	//创建一个 ObjectStore
	var objectStore = trans.createObjectStore("objectStoreName", {
    "autoIncrement" : true,  // 默认是true，主键自增长
    "keyPath" : id // 对象的主键
	});

	//删除一个 ObjectStore
	trans.deleteObjectStore(objectStoreName);

});

transactionPromise.done(function(event){
    // Called when transaction is completed
    event; // Indicated the transaction complete event.
});

transactionPromise.fail(function(event){
    // Called when the transaction is aborted or fails
    event.type; // indicates the reason for failure being error, exception or abort
});
```

##### 3. 对象表 （Object Stores）

在数据库存在的情况下，不需要先打开数据库链接和事务，直接使用下面的方法即可操作对象表。

数据库不存在时，操作不成功

```javascript
//mode 默认是 READ_WRITE
var objectStore = $.indexedDB("database_name").objectStore("objectStoreName", /* Optional */ mode );

//CURD 操作
var promise = objectStore.add(/*Javascript Object*/ value, /*Optional*/ key); // Adds data to the objectStore
var promise = objectStore.get(key); Gets the object with the key
var promise = objectStore.put(/*Javascript Object*/ value, key); // Updates the object for the specified key
var promise = objectStore.delete(key); // Deletes the object with the specified key
var promise = objectStore.count(); // Gets all the objects
var promise = objectStore.clear(); // Removes all data from the object store;

promise.done(function(result, event){
    result; // Result of the operation. Some operations like delete will return undefined
    event; // Success Event
});

promise.fail(function(error, event){
    error; // Type of error that has occured
    event; // Error event
    event.type; // indicates if there was an error or an exception
});

//迭代
var iterationPromise = objectStore.each(function(item){
    // Called for each element during the iteration
    item.key, item.value; // The key and the value of current object
    item.delete(); // Deletes the current item
    item.update(newItem); // Updates the current item with newItem;

    return;
    // false - do not continue iteration
    // integer 'n' - n can be 1,2,... indicating skip next n objects and continue
    // object - continue to item with object as the next key
    // default - no return, or undefined return, continue iteration

}, /*Optional*/ range, /*Optional*/ direction);

iterationPromise.done(function(result, event){
    // Iteration completed
    result ; // null, indicating that there are no more objects for iteration
    event ; // Success event
})

iterationPromise.fail(function(error, event){
    error; // Error during iteration
    event; // Error event, can be exception or event
});
```

##### 4. 索引 （Indexes）

索引只有在 Object Stores 打开的情况下才能使用

```javascript
var index = objectStore.index("indexName");
index.each(function(item){
    // Iterate over objects in index
    // Similar to iterating over objects in an ObjectStore
    item; // same as iterating over objects (see above)
}, /*Optional*/ range, /*Optional */ direction);

index.eachKey(function(item){
    //  Iterate over the keys of the object
});
```

索引在数据库版本更新后，照样有用

```javascript
// trans is created when a database upgrade is in progress
var objectStore = trans.objectStore("objectStoreName");
var index = objectStore.createIndex("object.property" , /*Optional*/ {
    "unique" : false, // Uniqueness of Index, defaults to false
    "multiEntry" : false // see explanation below
}, /* Optional */ "indexName")
objectStore.deleteIndex("indexName"); //returns nothing
```

##### 5. 删除数据库

```javascript
var deletePromise = $.indexedDB("database_name").deleteDatabase();

deletePromise.done(function(null, event){
    /* Called when the delete is successful*/
    event; // The success event
});
deletePromise.fail(function(error, event){
    /* Called when the delete is successful*/
    error; // Reason for the error
});
deletePromise.progress(function(db, event){
    // Called when the deleting is blocked due to another transaction
    db; // Database that is opened
    event.type // Indicates it is blocked, etc.
});
```

##### 6. 参考

- [IndexedDB API playground and examples](http://nparashuram.com/IndexedDB/)
- [IndexedDB W3C Specification](http://www.w3.org/TR/IndexedDB/)
- [My work on IndexedDB](http://blog.nparashuram.com/search/label/indexeddb)







