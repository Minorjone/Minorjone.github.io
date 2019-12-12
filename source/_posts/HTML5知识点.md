---
title: HTML5知识点
date: 2019-08-21 22:41:19
tags: 前端
categories: 前端
keywords: [前端,HTML5]
description: 
---

+ 最近阅读了HTML5权威指南，给HTML5中新增的知识点做了张思维导图，方便二次记忆和梳理。
+ 针对其中部分知识点，已做举例补充。
+ 有关Canvas部分的知识点不包括在内，将作为一个单独的专题进行梳理。
<!--more-->

思维导图
-------------
![Alt text][01]
[01]: ../assets/20190821/html5.png

History API
-------------
### 作用
用于在不刷新页面的前提下，通过脚本语言的方式来进行页面上某块局部内容的更新。

### 基本语法
```
history.pushState(state,null,"edit.php?id="+id);
```

### 事件处理
```
window.addEventListener("popstate",function(e){
    if(e.state)
    loadPage(e.state.userType,e.state.id);
});
```

本地存储
-------------
### Web Storage
#### sessionStorage
+ 解释：数据保存在session对象中。
+ 方法：
```
//setItem()
sessionStorage.setItem('key','value');
//or
sessionStorage.key = value;

//getItem()
var val = sessionStorage.getItem('key');
//or
var val = sessionStorage.key;
```

#### localStorage
+ 解释：数据保存在客户端本地的硬件设备中。
+ 方法：
```
//setItem()
localStorage.setItem('key','value');
//or
localStorage.key = value;

//getItem()
var val = localStorage.getItem('key');
//or
var val = localStorage.key;
```

#### storage事件
```
window.addEventListener('storage',function(event){
    //sessionStorage或localStorage的值发生变动时所要执行的处理
},false);
```
+ event.key：在session或localStorage中被修改的数据键值
+ event.oldValue：在session或localStorage中被修改前的值
+ event.newValue：在session或localStorage中被修改后的值
+ event.url：修改session或localStorage的页面的URL地址
+ event.storageArea：改动的为session或localStorage

### 本地数据库
#### SQLLite
```
var db = openDatabase('mydb','1.0','Test DB',2*1024*1024);
db.transaction(function(tx){
    tx.executeSql('INSET INTO MsgData VALUES(?,?,?)',[name,message,time],
        function(tx,rs){},
        function(tx,error){});
});
```

#### indexedDB
```
//兼容浏览器
window.indexedDB = window.indexedDB || window.webkitIndexedDB || window.mozIndexedDB || window.msIndexedDB;
window.IDBTransaction = window.IDBTransaction || window.webkitIDBTransaction || window.msIDBTransaction;
window.IDBKeyRange = window.IDBKeyRange || window.webkitIDBKeyRange || window.msIDBKeyRange;
window.IDBCursor = window.IDBCursor || window.webkitIDBCursor || window.msIDBCursor;

//连接数据库
var dbName = 'indexedDBTest';
var dbVersion = 20190824;
var idb;
var dbConnect = indexedDB.open(dbName,dbVersion);
dbConnect.onsuccess = function(e){
    idb = e.target.result;

    //开启事务
    var tx = idb.transaction(['Users'],"readwrite");
    var store = tx.objectStore('User');
    
    //保存数据
    var value = {
        userId:1,
        userName: '张三',
        address: '住址'
    };
    var req = store.put(value);  //会覆盖
    //or
    var req = store.add(value);  //不会覆盖
    req.oncomplete = function(e){};
    req.onabort = function(e){};

    //获取数据
    var req = store.get(1);
    var idx = store.index('userNameIndex');
    var req = idx.get('张三');

    //根据索引检索
    var range = IDBKetRange.bound('用户A','用户D');
    var direction = "next";
    var req = idx.openCursor(range,direction);

    //统计对象数据数量
    var req = store.count();
}
dbConnect.onerror = function(e){
}
dbConnect.onupgradeneeded = function(e){
    idb = e.target.result;
    
    //创建仓库
    var tx = e.target.transaction;
    var name = 'User';
    var optionalParameters = {
        keyPath: 'userId',
        autoIncrement: false
    };
    var store = idb.createObjectStore(name,optionalParameters);

    //创建索引
    var name = 'userNameIndex';
    var keyPath = 'userName';
    var optionalParameters = {
        unique: false,
        multiEntry: false
    };
    var idx = store.createIndex(name,keyPath,optionalParameters);
};
```

离线存储
-------------
### manifest文件
列举了需要被缓存或不需要被缓存的资源文件的名称，以及这些资源文件的访问路径。

### applicationCache对象
#### 事件
```
applicationCache.onchecking = function(e){};
applicationCache.onnoupdate = function(e){};
applicationCache.ondownloading = function(e){};
applicationCache.onprogress = function(e){};
applicationCache.onUpdateReady = function(e){};
applicationCache.oncached = function(e){};
applicationCache.onerror = function(e){};
```

#### swapCache
立即更新本地缓存。

文件API
-------------
### FileList对象与file对象
```
var file;
for(var i = 0;i<document.geElementById("file").files.length;i++){
    file = document.getElementById("file").files[i];
}
```

### ArrayBuffer
用于二进制数据缓存字符串。
```
var buf = new ArrayBuffer(32);
```

#### ArrayBufferView对象
```
var Int32Array = new Int32Array(ArrayBuffer);
```

#### DataView对象
```
var view = new DataView((uffer,byteOffset,byteLength);
```

#### Blob对象
```
var blob = new Blob ([blobParts,type]);
var newBlob = blob.slice(start,end,contentType);
```

### FileReader对象
用来把文件读入内存，并且读取文件中的数据。
#### 方法
+ readAsText：以文本方式读取，读取的结果即这个文本文件中的内容
+ readAsBinaryString：读取为二进制字符串
+ readAsDataURL：读取一串Data URL字符串
+ readyArrayBuffer：读取为一个ArrayBuffer

#### 事件
+ onabort()
+ onprogress()
+ onerror()
+ onload()
+ onloadstart()
+ onloadend()

### FileSystem API
用于用户需要永久保存数据，但是本地数据库的利用不能满足用户需求。
#### 适用场合
+ 文件分段上传
+ 在视频游戏或其他使用大量与媒体数据相关的应用程序中
+ 在离线或使用本地缓存的音频编辑或图片编辑应用程序中
+ 离线视频播放器
+ 离线邮件客户端

#### 基本操作
+ 请求访问文件系统：
window.requestFileSystem(type,size,successCallback,opt_errorCallback)
+ 申请磁盘配额：
window.webkitStorageInfo.requestQuota(PERSISTENT,1024*1024,successHandler,errorHandler)
+ 创建文件：
fs.root.getFile(filename,{create:true},successHandler,errorHandler)
+ 写入文件：fileEntry.createWriter(successHandler,errorHandler)
+ 在文件中追加数据：fileWriter.seek(fileWrer.length)
+ 读取文件：fileEntry.file(successHandler,errorHandler)
+ 复制磁盘中的文件：fileWriter.write(file);
+ 删除文件：fileEntry.remove(successHandler,errorHandler)
+ 创建目录：
fs.root.getDirectory(path,{create:true},successHandler,errorHandler)
+ 读取目录中的内容：
fs.root.createReader().readEntries(successHandler,errorHandler)
+ 删除目录：dirEntry.remove(successHandler,errorHandler)
+ 复制文件或目录：
fileEntry.copyTo(parent,newName,successHandler,errorHandler)
+ 移动文件或目录与重命名文件或目录：
fileEntry.moveTo(parent,newName,successHandler,errorHandler)

### Base64编码支持
+ 理解为二进制：window.btoa
+ 理解为ASCII码：window.atob

通信 API
-------------
### 跨文本信息传输
```
window.addEventListener("message",function(){...},false);
otherWindow.postMessage(message,targetOrigin);
```

### WebSockets
#### 实现方法
```
var webSocket = new WebSocket("ws://localhost:8005/socket");
webSocket.send("data");
webSocket.onmessage = function(event){};
webSocket.onopen = function(event){};
webSocket.onclose = function(event){};
```

#### 发送对象
```
webSocket.send(JSON.stringify({
    result: successFlag,
    time: currentTime
}));
```

#### 适用场景
+ 多人在线游戏网站
+ 聊天室
+ 实时体育或新闻评论网站
+ 实时交互用户信息的社交网站

### Server-Sent Events API
#### 适用场景
+ 在股票软件中实时显示股票的在线数据
+ 在新闻网站中实时显示最近刚刚发生的重大新闻
+ 在在线聊天软件中实时显示当前聊天室中的用户名及用户人数
+ 在其他任何需要实时显示服务端数据的场合

#### 实现方法
```
var source = new EventSource("test.php",{withCredentials:true});
source.close();
source.onmessage = function(e){};
source.onopen = function(e){};
source.onerror = function(e){};
```

拖放 API
-------------
```
var target = document.querySelector('#drop-target');
var dragElements = document.querySelectorAll('#drag-elements li');

// 追踪被拖动元素的变量
var elementDragged = null;

for (var i = 0; i < dragElements.length; i++) {

  dragElements[i].addEventListener('dragstart', function(e) {
    e.dataTransfer.setData('text', this.innerHTML);
    elementDragged = this;
  });

  dragElements[i].addEventListener('dragend', function(e) {
    elementDragged = null;
  });

};

target.addEventListener('dragover', function(e) {
  e.preventDefault();
  e.dataTransfer.dropEffect = 'move';
  return false;
});

target.addEventListener('drop', function(e) {
  e.preventDefault(); 
  e.stopPropagation();

  this.innerHTML = "Dropped " + e.dataTransfer.getData('text');

  document.querySelector('#drag-elements').removeChild(elementDragged);

  return false;
});
```