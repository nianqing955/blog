---
title: 前端框架NextJs介绍及实践
author: allen
date: Thu Jul 20 2017 10:38:05 GMT+0800 (CST)
tags:
  - web框架
---

### 1.引言

当前利用SSR（服务端渲染）的方式来进行前端的SAP设计比较火热，其目的主要是为了解决单页面应用的 SEO 的问题，
对于一般网站影响不大，但是对于论坛类，内容类网站来说是致命的，搜索引擎无法抓取页面相关内容，也就是用户搜不到此网站的相关信息。

* 是否需要SSR

服务端渲染，视图 HTML 直出有一些显而易见的好处：

* SEO 友好，HTML 直接输出对搜索引擎的抓取和理解更有利
* 用户能够第一时间看到内容,首屏时间缩短

但是，如果使用服务端渲染，我们将面对:

> 更高的成本。开发某一组件时，我们需要考虑其运行环境NodeJS 和 浏览器 双端的，需要考虑在服务端渲染要提前编译，以及组件的浏览器兼容性。
> 用户可交互时间不一定会提前。交互行为是由组件管理的，组件从当前视图反解出数据和结构需要遍历 DOM 树，反解的时间不一定比在前端直接渲染要快
> 后台系统（CMS、MIS、DashBoard之类）大多使用 Single Page Application 的模式。显而易见，这类系统不需要使用 SSR

* SSR的应用场景

> 功能型页面，不需要使用 `SSR`。比如个人中心、我的收藏之类
> 仅在 App 的 WebView 中展现，不作为开放 Web 存在的页面，不需要使用 `SSR`
> 偏重内容型页面，可以使用 `SSR`。但是组件是管理行为交互的，对内容部分无需进行组件渲染，只需要在有交互的部分进行组件渲染


### 2.NextJs介绍

> `NextJs` 是一种React的服务端渲染框架，且`NextJs`集成度极高，框架自身集成了webpack、babel、express等，使得开发者可以仅依赖next、react、react-dom就可以非常方便的构建自己的SSR React应用，开发者甚至都不用像以前那样关心路由。

>`NextJs`的高度集成性，使得我们很容易就能实现代码分割、路由跳转、热更新以及服务端渲染和前端渲染。
同时，`NextJs`可以与express、Koa、Hapi等服务端结合使用，也可以引入单元测试Jest、代码检查flow、SVG 组件、redux、mobx、 以及Apollo框架来搭载Graphql的数据请求等。

>示例参考地址：[More Examples of NextJs](https://zeit.co/blog/next2)


### 3.NextJs使用实践
本小节中主要是详细介绍了，NextJs的使用方式，并且在demo的基础上，
利用NextJs的方式来简单实现应用发布系统的界面功能，最后给出NextJs的使用体验。

本文档中，从如何搭建nextjs开发环境，并一步步的完成简单demo，
* 前端渲染
* 后端渲染 
* 数据请求
* 样式加载

* 环境配置
```shell
mkdir hello-nextjs
cd hello-nextjs
npm inint -y
npm install --save react react-dom next
mkdir pages //nextJs 的路由入口目录
```

修改package.json
```json
{
  "scripts": {
    "dev": "next"
  }
}
```
npm run dev
完成上述步骤后，在浏览器中，敲localhost：3000，并观察命令行是什么结果。

![](/images/4-2.png)

发现NextJs中自动配置了404界面

![](/images/4-1.png)

在命令行中，在开发模式下，next会启动定时任务，进行热更新。

#### 前端渲染
在pages目录下，新建一个index.js文件
```jsx 
const Index = () => (
  <div>
    <p>Hello Next.js</p>
  </div>
)
export default Index

```
此时观察浏览器，就会看到hello nextjs的字样
当然NextJs也提供了一种代码检查机制，如果在开发过程中，我们少写了一个标签，则在浏览器中会出现以下报错。

![](/images/4-3.png)

* 界面跳转

修改pages/index.js中的代码：

```jsx 
// 引入 link 用button也可以
import Link from 'next/link'
const Index = () => (
  <div>
    <Link href="/about">  |
      <a>About Page</a>   | <button>Go to About Page</button>
    </Link>
    <p>Hello Next.js</p>
  </div>
)
export default Index
```
![](/images/4-4.png)
当点击link时，会跳转到about的界面，同时工具栏返回箭头也能返回到上一个界面。
在next/link中包含栏所有location.history的功能，使得我们不需要再去前端添加window.href等跳转代码。

* 公共组件

在NextJs中，NextJs会根据Pages目录下的文件的名称，来作为跳转的URL。同时，React一样，也可以创建公用的组件。
如图，我们新建了一个common目录，并创建一个共享的Header组件。
![](/images/4-5.png)

修改index.js

```jsx
import Header from '../common/Header'

export default () => (
  <div>
    <Header />
    <p>Hello Next.js</p>
  </div>
)
```

添加pages/about.js

```jsx
import Header from '../common/Header'

export default () => (
  <div>
    <Header />
    <p>this is about page</p>
  </div>
)
```
效果如下：

![](/images/4-1.gif)

另外，理念与html的ejs模版引擎一样，在NextJs中，也可以设置一个通用的Layout component 供每一个pages使用。code如下：

```jsx
//Layout component
import Header from './Header'

const layoutStyle = {
  margin: 20,
  padding: 20,
  border: '1px solid #DDD'
}

const Layout = (props) => (
  <div style={layoutStyle}>
    <Header />
    {props.children}
  </div>
)

export default Layout

// pages/index.js

import Layout from '../common/MyLayout.js'

export default () => (
    <Layout>
       <p>Hello Next.js</p>
    </Layout>
)

// pages/about.js 

import Layout from '../common/MyLayout.js'

export default () => (
    <Layout>
       <p>This is the about page</p>
    </Layout>
)

```

* 创建动态的pages
比如我们想动态的添加网页的内容，当点击link时，会出现的url时什么？代码示例如下：

```jsx

import Layout from '../common/MyLayout.js'
import Link from 'next/link'

const PostLink = (props) => (
  <li>
    <Link href={`/post?title=${props.title}`}> 
      <a>{props.title}</a>
    </Link>
  </li>
)

export default () => (
  <Layout>
    <h1>My Blog</h1>
    <ul>
      <PostLink title="Hello Next.js"/>
      <PostLink title="Learn Next.js is awesome"/>
      <PostLink title="Deploy apps with Zeit"/>
    </ul>
  </Layout>
)
```

添加pages/post.js

```jsx
import Layout from '../common/MyLayout.js'

export default (props) => (
    <Layout>
       <h1>{props.url.query.title}</h1>
       <p>This is the blog post content.</p>
    </Layout>
)

```

效果如下：

![](/images/4-2.gif)

* url的转换
在上述的示例中，我们看到当我点击link时，看到的是类似于这样的地址：`http://localhost:3000/post?title=Hello%20Next.js`
那么怎么将其变成http://localhost:3000/post/hello-nextjs呢？

修改pages/index.js如下：

```jsx
import Layout from '../common/MyLayout.js'
import Link from 'next/link'

const PostLink = (props) => (
  <li>
    <Link as={`/post/${props.id}`} href={`/post?title=${props.title}`}>
      <a>{props.title}</a>
    </Link>
  </li>
)

export default () => (
  <Layout>
    <h1>My Blog</h1>
    <ul>
      <PostLink id="hello-nextjs" title="Hello Next.js"/>
      <PostLink id="learn-nextjs" title="Learn Next.js is awesome"/>
      <PostLink id="deploy-nextjs" title="Deploy apps with Zeit"/>
    </ul>
  </Layout>
)
```
通过link as我们可以实现url的转换，但是点击link后，当我们再次刷新浏览器后，会发生什么呢？
下面我们将介绍NextJs的服务端渲染。

#### 服务端渲染

在项目的root目录下，添加server.js，并修改package.json。code如下：

```jsx 
// server.js
const express = require('express')
const next = require('next')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare()
.then(() => {
  const server = express()
  
  server.get('/post/:id', (req, res) => {
  	const actualPage = '/post'
  	const queryParams = { title: req.params.id } 
  	app.render(req, res, actualPage, queryParams)
  })

  server.get('*', (req, res) => {
    return handle(req, res)
  })

  server.listen(3000, (err) => {
    if (err) throw err
    console.log('> Ready on http://localhost:3000')
  })
})
.catch((ex) => {
  console.error(ex.stack)
  process.exit(1)
})

//package.json

{
  "scripts": {
    "dev": "node server.js"
  }
}
```
效果如下：

![](/images/4-3.gif)

问题是，当我们去刷新浏览器时，`const queryParams = { title: req.params.id }` 使得标题title会发生改变。
实际上，这时我们需要的是根据id来拿到对应的title。下面引入第三部分data fetch。

#### NextJs数据请求
在NextJs中，需要安装fetch依赖，npm install --save isomorphic-unfetch，并且此包可以在客户端和服务端环境下运行。
利用官网上提供的[TVMaze API](http://www.tvmaze.com/api)来实现data fetch。
修改pages/index.js中的代码
```jsx
import Layout from '../common/MyLayout.js'
import Link from 'next/link'
import fetch from 'isomorphic-unfetch'

const Index = (props) => (
  <Layout>
    <h1>Batman TV Shows</h1>
    <ul>
      {props.shows.map(({show}) => (
        <li key={show.id}>
          <Link as={`/p/${show.id}`} href={`/post?id=${show.id}`}>
            <a>{show.name}</a>
          </Link>
        </li>
      ))}
    </ul>
  </Layout>
)

Index.getInitialProps = async function() {
  const res = await fetch('https://api.tvmaze.com/search/shows?q=batman')
  const data = await res.json()

  console.log(`Show data fetched. Count: ${data.length}`)

  return {
    shows: data
  }
}

export default Index
```
代码中，NextJs通过getInitialProps function来实现数据的请求，最终的效果如下：

![](/images/4-6.png)

在server端中，可以看到每次刷新界面，都是服务端去请求数据并打印出console.log，也就是说当前是服务端在渲染。而在浏览器中并没有出现log信息。

![](/images/4-7.png)

修改pages/post.js
```jsx
import Layout from '../common/MyLayout.js'
import fetch from 'isomorphic-unfetch'

const Post =  (props) => (
    <Layout>
       <h1>{props.show.name}</h1>
       <p>{props.show.summary.replace(/<[/]?p>/g, '')}</p>
       <img src={props.show.image.medium}/>
    </Layout>
)

Post.getInitialProps = async function (context) {
  const { id } = context.query
  const res = await fetch(`https://api.tvmaze.com/shows/${id}`)
  const show = await res.json()

  console.log(`Fetched show: ${show.name}`)

  return { show }
}

export default Post
```
结果如下：
![](/images/4-8.png)
结果发现，当点击任意一个link时，客户端打印除了log信息，但server端没有log信息，此时是前端渲染，并且点击home或者返回按钮时，
仍然发现是前端渲染，也就是说客户端只有在第一次请求时或者刷新浏览器的情况下是服务端渲染。个人感觉NextJs的渲染策略类似同构了。
[data fetching](https://github.com/zeit/next.js#fetching-data-and-component-lifecycle)

#### 样式加载
在React中，通常我们是通过import css file的方式、postCss 以及 css in js的方式来加载style的。
在SSR中，通过这import css的方式来加载样式会造成很多问题。
在NextJs中，会通过[styled jsx](https://github.com/zeit/styled-jsx)方式及css preload的web标准来实现css的按需加载。
并且在NextJs中，css是有作用域的。代码示例如下：

```jsx
//修改pages/index.js
import Layout from '../common/MyLayout.js'
import Link from 'next/link'
import fetch from 'isomorphic-unfetch'

const Index = (props) => (
  <Layout>
    <h1>Batman TV Shows</h1>
    <ul>
      {props.shows.map(({show}) => (
        <li key={show.id}>
          <Link as={`/p/${show.id}`} href={`/post?id=${show.id}`}>
            <a>{show.name}</a>
          </Link>
        </li>
      ))}
    </ul>
    <style jsx>{`
      h1, a {
        font-family: "Arial";
      }

      ul {
        padding: 0;
      }

      li {
        list-style: none;
        margin: 5px 0;
      }

      a {
        text-decoration: none;
        color: blue;
      }

      a:hover {
        opacity: 0.6;
      }
    `}</style>
  </Layout>
)

Index.getInitialProps = async function() {
  const res = await fetch('https://api.tvmaze.com/search/shows?q=batman')
  const data = await res.json()

  console.log(`Show data fetched. Count: ${data.length}`)

  return {
    shows: data
  }
}

export default Index
```
NextJs中styled-jsx的工作方式类似于babel plugin，它在页面渲染的进程中就将css 转成es5的形式。

改变pages/index.js中的代码，如下：

```jsx
import Layout from '../common/MyLayout.js'
import Link from 'next/link'

function getPosts () {
  return [
    { id: 'hello-nextjs', title: 'Hello Next.js'},
    { id: 'learn-nextjs', title: 'Learn Next.js is awesome'},
    { id: 'deploy-nextjs', title: 'Deploy apps with ZEIT'},
  ]
}

const PostLink = ({ post }) => (
  <li>
    <Link as={`/p/${post.id}`} href={`/post?title=${post.title}`}>
      <a>{post.title}</a>
    </Link>
  </li>
)

export default () => (
  <Layout>
    <h1>My Blog</h1>
    <ul>
      {getPosts().map((post) => (
        <PostLink key={post.id} post={post}/>
      ))}
    </ul>
    <style jsx>{`
      h1, a {
        font-family: "Arial";
      }

      ul {
        padding: 0;
      }

      li {
        list-style: none;
        margin: 5px 0;
      }

      a {
        text-decoration: none;
        color: blue;
      }

      a:hover {
        opacity: 0.6;
      }
    `}</style>
  </Layout>
)
```

结果发现li 的样式在PostLink中并没有起作用，也就是说，当前的style并不能传递到组件内部的元素中。如下图：

![](/images/4-9.png)

修改PostLink中的代码：
```jsx
const PostLink = ({ post }) => (
  <li>
    <Link as={`/p/${post.id}`} href={`/post?title=${post.title}`}>
      <a>{post.title}</a>
    </Link>
    <style jsx>{`
      li {
        list-style: none;
        margin: 5px 0;
      }

      a {
        text-decoration: none;
        color: blue;
        font-family: "Arial";
      }

      a:hover {
        opacity: 0.6;
      }
    `}</style>
  </li>
)
```
结果发现样式生效，如下图。

![](/images/4-11.png)

如果我们需要给link加上样式时，直接给link添加样式是无效的，link在这里属于一个高级的过渡组件[higher-order-components](https://facebook.github.io/react/docs/higher-order-components.html)。code如下。

```jsx 
<Link href="/about">
  <a style={{ fontSize: 20 }}>About Page</a>
</Link>
---------------------
//无效样式
<Link href="/about" style={{ fontSize: 20 }}>
  <a>About Page</a>
</Link>

```

当然如果想让css样式在子组件中生效，需要在父组件的style 中添加 global 关键字。

`<style global jsx>...</style>`

如果还想用类似postCss的方式来加载，NextJs也提供了几种加载的方式
* [postcss](https://github.com/zeit/next.js/tree/master/examples/with-scoped-stylesheets-and-postcss)
* [sass less or css](https://github.com/zeit/next.js/tree/master/examples/with-external-scoped-css)

#### 一键式部署

利用NextJs可以实现一键build和发布，package.json修改如下：

```json
"scripts": {
  "build": "next build"
  "start": "next start" //"start": "next start -p $PORT"指定端口 。启动方式：PORT=8000 npm start
}
```
同时，NextJs可以结合zeit now来实现一键式部署、发布代码。这里不展开说明，详见链接：[now](https://zeit.co/now);

上述的代码[gitlab hello-nextJs](http://gitlab.wuxingdev.cn/bfe/WAPS/tree/hello-nextjs)

在此基础上，我将现有的应用发布系统的部分功能用NextJs实现了一下，引入了mobx、koa、ant design等。
参考工程见[gitlab NextJs-WAPS](http://gitlab.wuxingdev.cn/bfe/WAPS/tree/waps-nextjs)

### 4.使用体会
NextJs的优势在于使得开发者能专注与业务的开发，而不用去关心工程的构建、开发环境的配置等。
在NextJs中，可以实现SSR快速渲染，当首次去打开浏览器请求数据时是服务端渲染，但此时点击页面时又变成前端渲染了，刷新点击后的页面就又变成服务端渲染，感觉就是同构啊。
并且其打包的bundlejs commonjs等依赖非常小,促进了页面的加载时间。NextJs路由也可以通过接入Next-routes来实现自定义的动态路由。同时，支持接入的其他库有很多，koa、express、mobx、graphql、redux、markdown、jest、flow、SVG等。

![](/images/4-10.png)

展望： 在NextJs 3.0中会引入React未来即将发布的新特性 [React fiber](https://gist.github.com/duivvv/2ba00d413b8ff7bc1fa5a2e51c61ba43)，
以用来提高React复杂应用的可响应性及性能。
在以往的React渲染方式中，每次state的变化都会使得React重新计算，如果计算量较大时（比如动画），
浏览器的主线程来不及render，那么动画就会出现卡顿的现象。
而`React fiber`是一种新的调度算法，可以将大量的计算拆解，异步化，使得浏览器主线程得以释放，保证了渲染的帧率。从而提高响应性的速度。
简而言之，fiber是一种任务调度器（栈），可以暂停一个任务，一会再执行，根据任务的优先级来确定执行的顺利。原来Js也是有并发的。

>参考文档：
[How to use NextJs](https://learnnextjs.com/)

附：
* [Js的并发模型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)

