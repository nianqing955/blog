---
title: socket io connect problem with koa & react
date: 2017-09-26 15:13:39
tags: NodeJs react koa
---

### 问题背景
在项目中，我需要实时记录差分包的生成状态，并将差分包及apk上传到阿里OSS中，当前的做法是通过api轮询的方式，来控制前端的进度展示。但这会带来的问题是，如果其中某一环节出现问题，那么前端就会陷入死循环。这样我考虑通过socket io 来建立持久化链接，来实时展示文件的生成及上传进度。

经过不断尝试及文档的查阅，终于成功将客户端与服务端socket io 链接。

网上现有的文档中，大都是以express做为参考依据的，并没有koa等相关的例子，踩了好多坑。下面将我的构建方式分享给大家，希望对大家能有所帮助。

环境：{
    node: 7.8.0
    socket:2.3.0
    react: 15.4.2
    webpack: 2.5.1
}
node server: localhost: 3000
web client: localhost:3001

根据官网上文档[socket.io](https://socket.io/docs)来初步搭建socket io服务。

```
npm install socket.io --save 
npm install socket.io-client --save

```

- node server

```js

const koa = require('koa');
const app = new koa();
const port = process.env.PORT || 3000;
//socket io 
let socket = require('socket.io').listen(app.listen(port));
socket.on('connection',(client)=>{
    //sock client func
})

---------------------------------------------------
//也可以这样写
const koa = require('koa');
const app = new koa();
const port = process.env.PORT || 3000;
const server = require('http').createServer(app.callback());
server.listen(port);
//socket io 
let socket = require('socket.io').listen(server);
socket.on('connection',(client)=>{
    //sock client func
})

```

- web clinet

可以通过两种方式来引入socket.io-client,一种是webpack的

```js
// one way
 plugins: [
     new webpack.ProvidePlugin({
         io: 'socket.io-client'
    })
 ]
// the another way
// 引入babel 文件后

import io from 'socket.io-client';
var socket = io('http://localhost:3000/');
socket.on('connect', ()=>{
    // code you want to do something
})

```

其它：

在跨域问题上，我尝试过用koa-cors 和cors 但是都不能解决我的问题，后来发现，这两种都是对于node 本身的api 请求上做的跨域，或者说，其内部的源码写的是`'Access-Control-Allow-Origin', true`，也就是允许跨域，但是问题依然存在。设置socket configure 也不能解决问题。
后来项目启动的js 脚本中，加上了cross-env 解决了问题

>  参考文档：

[webpack providePlugin](https://webpack.js.org/plugins/provide-plugin/)
[socket.io](https://github.com/socketio/socket.io)
[socket 404](https://github.com/socketio/socket.io/issues/1113)
[koa with socket](https://github.com/socketio/socket.io/issues/1390)






