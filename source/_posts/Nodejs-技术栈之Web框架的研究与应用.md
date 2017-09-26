---
title: Nodejs 技术栈之Web框架的研究与应用
date: 2017-03-10 15:02:57
tags: 
 - nodeJs
 
---
### Nodejs 技术栈之Web框架的研究与应用

@(Nodejs)[express|koa|egg]

-------------------
[TOC]

#### 1.引言
Nodejs是一种基于事件驱动的服务端javascript。目前基于nodejs的web框架有很多，可以提供一系列强大的特性来帮助我们创建各种web应用及丰富的HTTP工具。现有的框架中，express、koa、egg应用比较广泛，而这三种框架之间又存在很大的差异。因此，本文将从框架自身的角度由表及里的来分析研究三者的优劣性及原理。最后结合实际代码，来进行进一步剖析。
> express：express 是 node 社区广泛使用的框架，简单且扩展性强，非常适合做个人项目。但框架本身缺少约定。

>koa：koa.js 是下一代的node.js框架，由Express团队开发，通过生成器（generators JavaScript 1.7新引入的，用于解决回调嵌套的方案），减少异步回调，提高代码的可读性和可维护性，同时改进了错误处理。
koa 的先天优势在于 generator，带来的主要好处如下：
更优雅、简单、安全的中间件机制，后面章节会详细说明
更优雅、简单的异常处理
更优雅、简单的异步编程方式

>egg：egg 为企业级框架和应用而生，可以帮助开发团队和开发人员降低开发和维护成本。egg继承于koa，而egg 选择了 koa 作为其基础框架，在它的模型基础上，进一步对它进行了一些增强。
特性：
      深度框架定制
      高度可扩展的插件机制
      内置多进程管理
      基于 koa 开发，性能优异
      框架稳定，测试覆盖率高
      渐进式开发

#### 2.中间件
koa与express最大的不同之处在于中间件的加载方式不一样。koa是基于洋葱模型来加载的，而express是从上到下依次加载，首先装入的中间件函数也首先被执行。
```javascript
-------------------express----------------
var express = require('express');
var app = express();
var myLogger = function (req, res, next) {
  console.log('LOGGED');
  next();
};
app.use(myLogger);
// 打印hello，world
app.get('/', function(req,res){
    console.log('主页请求')；
    res.send('hello world');
});
// app.use(myLogger);  如果放在app.get()之后呢？
app.listen(3000);
```
在学习koa中间件之前，我们需要弄清什么，有哪些问题。
- koa 的中间件机制是如何实现？
- 为什么中间件必须是 generator function？
- next 实参指向是什么？为什么可以通过 yield next 可以执行下一个中间件？
- 为什么中间件从上到下执行完后，可以从下到上执行 yield next 后的逻辑？

koa洋葱模型：
![](/images/22.jpg)

```javascript
//请思考下面的打印结果是什么
var app = koa();
app.use(function *(next){
    this.body = '1';
    yield next;
    this.body += '5';
    console.log(this.body);
});
app.use(function *(next){
    this.body += '2';
    yield next;
    this.body += '4';
});
app.use(function *(next){
    this.body += '3';
});
app.listen(3000);
```
如果不用var app ＝ koa()这个封装好的框架中间件，怎么模拟出它执行的机制？

```javascript
var co = require('co');//什么是co？后面将做解析
function NewKoa(){
    this.middlewares = [];
}
NewKoa.prototype = {
    //注入个中间件
    use: function(generatorFn){
        this.middlewares.push(generatorFn);
    },
    //执行中间件
    listen: function(){ //app.listen(3000)  实际就是调用这里
        this._run();
    },
    _run: function(){
        var ctx = this;
        var middlewares = ctx.middlewares;
        return co(function *(){
            var prev = null;
            var i = middlewares.length;
            //从最后一个中间件到第一个中间件的顺序开始遍历
            while (i--) ｛
                //prev 将前面一个中间件传递给当前中间件
                prev = middlewares[i].call(ctx, prev);
            }
            //执行第一个中间件
            yield prev;
        })();
    }
};
var app = new NewKoa();
....
```
 egg 是基于 koa 1 实现的，所以 egg 的中间件形式和 koa 的中间件形式是一样的，都是基于 generator function 的洋葱圈模型，因而在此不做说明。
 
#### 3.路由
在nodejs中，当我们建立了http请求后，需要建立路由来确定谁来响应客户端的请求。在HTTP请求中，我们可以通过路由来提取请求的url及get/post参数。
通常情况下，创建服务需要使用require来载入http模块，并将实例化的http赋值。
##### nodejs 服务实例
```javascript
var http = require('http');
var server = http.createServer(function(req,res){
  req.writeHead(200,{'Content-Type': 'text/plain'});
  res.end('hello world');
});
server.listen(3000);
```
#####  express 的路由
```javascript
var express = require('express');
var app = express();
// 打印hello，world
app.get('/', function(req,res){
    console.log('主页请求')；
    res.send('hello world');
})
// 打印about
app.get('/about', function (req, res) {
  res.send('about');
});
app.listen(3000,function(){
  console.log('server is start on 3000...');
});
```
##### koa路由
所有的koa中间件，必须是 generator function ，即 function *(){} 语法。后面章节进行介绍。
```javascript
//简单路由
var koa = require('koa');
var app = koa();
// koa 是通过 app.use注入来加载中间件的
app.use(function *(){
    var path = this.path;
    this.body = path; //打印当前请求路径
});
app.listen(3000);

//可以注入多个中间件，koa 会从上到下加载，然后从下到上执行，关于中间件的执行顺序问题下一章进行分析
app.use(function *(next){ 
    this.demo = 'test text';
    yield next;//跳到下一个中间件
});

app.use(function *(){
    this.body = this.demo; // 打印test text
});
app.listen(3000);
// 引入路由中间件的写法
var router = require('koa-router');
app.use(router(app));
app.get('/', function *(next) {
    //我是首页
    //this 指向请求
});

//param()法，用于路由的参数处理,当访问 /detail/:id 路由时，会先执行定义的 generator逻辑。
app.param('id',function *(id,next){
    this.id = Number(id);
    if ( typeof this.id != 'number') return this.status = 404;
    yield next;
}).get('/detail/:id', function *(next) {
    //我是详情页面
    var id = this.id; //123
    this.body = id;
});

```

##### egg路由
egg是基于koa框架开发的，其路由是通过描述请求的url及承担的执行动作controller来实现的，框架中约定了app/router.js文件用于统一所有的路由规则。
```javascript
// app/router.js
module.exports = app => {
app.get('/user/:id', 'user.info');
};
// app/controller/user.js
exports.info = function* () {
  this.body = {
    name: `hello ${this.params.id}`,
  };
};
```
#### 4.高级
本章节主要是讲解node中一些比较重要的理论，回调地狱，es6 promise -> generator -> co 以及nodejs框架的异常错误处理方式。
##### callback hell 
>Node.js 本身是单线程的，但通过其异步加载的特性可以实现任务以并行的方式进行。当业务逻辑复杂的时候，回调的嵌套过多，代码复杂度增加，可读性降低，维护起来也复杂，调试也复杂，就会造成回调地狱。

```javascript
asyncFun1(function(err, a) {
    if (err){
    callback(err);
  }
    asyncFun2(function(err, b) {
        if (err){
        callback(err);
      }
        asyncFun3(function(err, c) {
           if (err){
          callback(err);
        }
        });
    });
});
```
JS的回调让我们可以很轻易的写出异步执行的代码，而缺点也是由异步引起的，当太多的异步步骤需要一步一步执行，或者一个函数里有太多的异步操作，这时候就会产生大量嵌套的回调，使代码嵌套太深而难以阅读和维护。
>问题：那怎么改进？

方法1：具名函数  --使用具名函数并保持代码层级不要太深
```javascript
function fun3(err, c) {
  // 在函数3中处理c
}
function fun2(err, b) {
  // 在函数2中处理b
    asyncFun3(fun3);
}
function fun1(err, a) {   
  // 在函数1中处理a
    asyncFun2(fun2);
}
asyncFun1(fun1);

```
方法2：Anync 
```javascript
async.series([
    function(callback) {
        // do some stuff ...
        callback(null, 'one');
    },
    function(callback) {
        // do some more stuff ...
        callback(null, 'two');
    }
],
// optional callback
function(err, results) {
    // results is now equal to ['one', 'two']
});
```
方法3：Promise --虽然没有了嵌套，还是需要不少的回调
```javascript
asyncFun1().then(function(a) {
    // do something with a in function 1
    asyncFun2();
}).then(function(b) {
    // do something with b in function 2
    asyncFun3();
}).then(function(c) {
    // do somethin with c in function 3
});
```
Promise对象有以下两个特点。
（1）对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：Pending（进行中）、Resolved（已完成，又称 Fulfilled）和Rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象代表一个异步操作，有三种状态：Pending（进行中）、Resolved（已完成，又称 Fulfilled）和Rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

>问题还有没有更好的方法？

##### generator
ES6新特性中带来了新一代解决回调地狱的神器：Generator，同时generator 也是 koa 的基础，想要用好 koa 离不开对 generator 的理解。那么什么是generator？
Generator是一种方便按照某种规则生成元素的迭代器，不过鉴于其特殊的语法和运行原理，可以通过某种方式写出同步化的异步代码，从而避免回调。
```javascript
var r = 3;
function* genfn(a) {
    for( var i = 0; i < 3 ; i++) {
        a = a + r ;
        yield a;
    }
}

var sum = genfn(5);

console.log(sum.next()); // returns { value : 8, done : false }
console.log(sum.next()); // returns { value : 11, done: false }
console.log(sum.next()); // returns { value : 14, done: false }
console.log(sum.next()); //return { value: undefined, done: true }
```
函数genfn 定义了一个执行3次的循环，每次执行，给a变量加3。
yield a 会暂停执行并保存当前堆栈，返回当前的a。
 与普通函数不同，generator只会定义遍历器，而不会执行，每次调用这个遍历器的next方法，就从函数体的头部或者上一次停下来的地方开始执行，直到遇到下一个yield语句为止。
 
##### co
co是什么？
co 函数库是著名程序员 TJ Holowaychuk 于2013年6月发布的一个小工具，用于 Generator 函数的自动执行。

前面说过，Generator 函数就是一个异步操作的容器。它的自动执行需要一种机制，当异步操作有了结果，能够自动交回执行权。使用 co 的前提条件是，Generator 函数的 yield 命令后面，只能是 Thunk 函数或 Promise 对象。
```javascript
// 利用Generator 函数，依次读取两个文件。
var co = require('co');
var fs = require('fs');

function read(file) {
  return function(fn){
    fs.readFile(file, 'utf8', fn);
  }
}
co(function *(){//generator 函数只要传入 co 函数，就会自动执行

  var a = yield read('.gitignore');//读取文件1
  console.log(a.length);

  var b = yield read('package.json');//读取文件2
  console.log(b.length);
}).then(function(){
console.log('执行完毕')//co 函数返回一个 Promise 对象，因此可以用 then 方法添加回调函数。
});
```
co 要求所有的异步函数必须是偏函数，称之为 thunk :
```javascript
function read(file) {
  return function(fn){
    fs.readFile(file, 'utf8', fn);
  }
}
```
对实现原理感兴趣的可以阅读其源码：https://github.com/tj/co/blob/master/index.js

co的嵌套使用：（并发的异步操作）
co 支持并发的异步操作，即允许某些操作同时进行，等到它们全部完成，才进行下一步。只要把并发的操作都放在数组或对象里面。
```javascript
// 数组的写法
co(function* () {
  var res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ];
  console.log(res); 
}).catch(onerror);

// 对象的写法
co(function* () {
  var res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2),
  };
  console.log(res); 
}).catch(onerror);
```
##### error
在 Node.js 中，错误处理的方法主要有下面几种：

和其他同步语言类似的 throw / try / catch 方法
callback(err, data) 回调形式
通过 EventEmitter 触发一个 error 事件

错误处理是应用健壮性非常重要的一部分，koa 在错误处理的便利上比 express 好非常多。koa 有 error 事件，当发生错误时，可以通过该事件，对错误进行统一的处理。
```javascript
//express
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});

//koa
const fs = require('fs');
const Promise = require('bluebird');

let filename = '/nonexists';
let statAsync = Promise.promisify(fs.stat);
try {
  yield statAsync(filename);
} catch(e) {
  // error here
}
```
koa 自定义中间件 统一的错误处理
```javascript
app.use(function* (next) {
  try {
    yield* next;
  } catch(e) {
    let status = e.status || 500;
    let message = e.message || '服务器错误';

    if (e instanceof JsonError) { // 错误是 json 错误
      this.body = {
        'status': status,
        'message': message
      };
      if (status == 500) { 
        // 触发 koa 统一错误事件，可以打印出详细的错误堆栈 log
        this.app.emit('error', e, this);
      }
      return;
    }
    
    this.status = status;
    // 根据 status 渲染不同的页面
    if (status == 403) {
      this.body = yield this.render('403.html', {'err': e});
    }
    if (status == 404) {
      this.body = yield this.render('404.html', {'err': e});
    }
    if (status == 500) {
      this.body = yield this.render('500.html', {'err': e});
      // 触发 koa 统一错误事件，可以打印出详细的错误堆栈 log
      this.app.emit('error', e, this);
    }
  }
});
```
egg
```javascript
try {
  const res = yield this.ctx.curl('http://eggjs.com/api/echo', { dataType: 'json' });
  if (res.status !== 200) throw new Error('response status is not 200');
  return res.data;
} catch (err) {
  this.logger.error(err);
  return {};
}
```
##### node process
nodejs代码是单线程的，一个 node 进程只能运行在一个 CPU 上。那么如果用 node 来做 web server，就无法享受到多核运算的好处。
>如何利用上多核 CPU 的并发优势？

- 同时启动多个进程。

nodejs 提供了child_process 模块来创建子进程，方法有 exec、spawn、fork三种。

1. exec 使用子进程执行命令，缓存子进程的输出，并将子进程的输出以回调函数参数的形式返回。
2. spawn 使用指定的命令行参数创建新进程。
3. fork 是 spawn()的特殊形式，用于在子进程中运行的模块，如 fork('./son.js') 相当于 spawn('node', ['./son.js']) 。与spawn方法不同的是，fork会在父进程与子进程之间，建立一个通信管道，用于进程之间的通信。

实例：
```javascript
//worker.js
console.log("进程 " + process.argv[2] + " 执行。" );

//master.js
const fs = require('fs');
const child_process = require('child_process');

for(var i=0; i<3; i++) {
   var workerProcess = child_process.exec('node support.js '+i,
      function (error, stdout, stderr) {
         if (error) {
            console.log(error.stack);
         }
         console.log('stdout: ' + stdout);
         console.log('stderr: ' + stderr);
      });
      workerProcess.on('exit', function (code) {
      console.log('子进程已退出，退出码 '+code);
   });
}
```
执行结果：
```
 node master.js 
子进程已退出，退出码 0
stdout: 进程 1 执行。

stderr: 
子进程已退出，退出码 0
stdout: 进程 0 执行。

stderr: 
子进程已退出，退出码 0
stdout: 进程 2 执行。

stderr: 
```
- EGG中的多进程模型
在服务器上同时启动多个进程。
每个进程里都跑的是同一份源代码（好比把以前一个进程的工作分给多个进程去做）。
这些进程可以同时监听一个端口
```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;
if (cluster.isMaster) {
  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  cluster.on('exit', function(worker, code, signal) {
    console.log('worker ' + worker.process.pid + ' died');
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer(function(req, res) {
    res.writeHead(200);
    res.end("hello world\n");
  }).listen(8000);
}
```

#### 5.应用
 express + swig
```javascript
//nodejs server.js
var app = require("express")(),
    serveStatic = require('serve-static'),
    bodyParser = require('body-parser'),
    swig = require('swig'),
    session = require('express-session'),
    ueditor = require("ueditor-nodejs"),
    path = require('path');

var userRouter = require("./backend/router/userRouter"),
    registerRouter = require("./backend/router/registerRouter"),
    signinRouter = require("./backend/router/signinRouter"),
    orderRouter = require("./backend/router/orderRouter"),
    paperRouter = require("./backend/router/PaperShare"),
    uploadRouter = require("./backend/router/UploadRouter"),
    articleRouter = require("./backend/router/articleRouter");

var allowCrossDomain = function(req, res, next) {
  //console.log("get request!");
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE');
  //res.header('Access-Control-Allow-Credentials',true);
  res.header('Access-Control-Allow-Headers', 'Content-Type');
  next();
};

swig.setDefaults({
  cache: false
});

// var swig2 = new swig.Swig();
app.engine('html', swig.renderFile);
app.set('view engine', 'html');
app.set('views', './templates')

app.use(log('dev'));
app.use(bodyParser.urlencoded({
  extended: true
}));
app.use(bodyParser.json());

app.use(allowCrossDomain);
// app.use('/admin',checkUser);
app.use(serveStatic('public'));

app.use(function (req, res, next) {
  console.log('filter session.user is ',req.session.user);
  if (req.session.user){
    res.locals.currentUser = req.session.user;
  }
  next();
});

app.use('/user', userRouter);
app.use('/regist',registerRouter);
app.use('/signin',signinRouter);
app.use('/order',orderRouter);
app.use('/paper',paperRouter);
app.use('/upload',uploadRouter);
app.use('/article',articleRouter);

// Swig will cache templates for you, but you can disable
// that and use Express's caching instead, if you like:
app.set('view cache', false);
//welcome index 首页
app.get('/', function(req, res) {
  //res.redirect("./admin");
  res.render('index',{user:req.session.user})
});

app.listen(18080, function() {
  logger.log("connect successful! Listen 18080!");
});

```

 koa + xtml 

```javascript
//node app.js
var debug = require('debug')('koa-example');
var koa = require('koa');
//配置文件
var config = require('./config/config');

var app = koa();
app.use(function *(next){
    //config 注入中间件，方便调用配置信息
    if(!this.config){
        this.config = config;
    }
    yield next;
});

//log记录
var Logger = require('mini-logger');
var logger = Logger({
    dir: config.logDir,
    format: 'YYYY-MM-DD-[{category}][.log]'
});

//router use : this.logger.error(new Error(''))
app.context.logger = logger;

var onerror = require('koa-onerror');
onerror(app);

//xtemplate对koa的适配
var xtplApp = require('xtpl/lib/koa');
//xtemplate模板渲染
xtplApp(app,{
    //配置模板目录
    views: config.viewDir
});
var session = require('koa-session');
app.use(session(app));

//post body 解析
var bodyParser = require('koa-bodyparser');
app.use(bodyParser());
//数据校验
var validator = require('koa-validator');
app.use(validator());

//静态文件cache
var staticCache = require('koa-static-cache');
var staticDir = config.staticDir;
app.use(staticCache(staticDir+'/js'));
app.use(staticCache(staticDir+'/css'));

//路由
var router = require('koa-router');
app.use(router(app));

//应用路由
var appRouter = require('./router/index');
appRouter(app);
app.listen(config.port);

console.log('listening on port %s',config.port);
```
#### 6.总结

koa 与 express 是共享底层库的，本身先天优势在于 generator。koa和express在表现上的一点不同是采用ctx一个参数来调用中间件，而不是express的req, res。express的设计是串联的，设计思路很简洁。而koa的某一个中间件可以自行选择之后中间件的执行位置的。路由的设计上也不一样。express是通过router，而koa是通过中间价加载的方式来统一跳转路径。

Egg自己做了一套进程管理机制，采用微核 + 插件体系，本身大部分功能由插件提供，高度灵活，功能强大。egg 的设计机制，旨在遵循同一套规范的同时，完美的达成生态共建和差异化定制的平衡点。

![](/images/111.jpg)

