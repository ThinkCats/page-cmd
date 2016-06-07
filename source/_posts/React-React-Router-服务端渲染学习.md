---
title: React & React Router 服务端渲染学习
date: 2016-06-07 21:52:18
categories: JavaScript
tags: ReactJS, React-Router
---

### 吐槽下

[《徒手撸一个Photo Hub（ReactJS + AltJS + Express ）》](http://thinkcat.me/2016/05/22/Photo-Hub/) 这玩意当初已经把界面撸好了，用 React 已经把 UI 搞完了，但是之后停滞了。因为发现 NodeJs 用 Es6 来写东西，是个大坑啊 ！！ 我死活不知道是哪里出问题了，NodeJs 愣是解析不到 JSX 了。

已经纠结了两个晚上了。

来babel 的世界一看， 我好像掉入了一片大海，左右看不到岸，任由风浪推着我到处飘摇。。。 记得上次出现这感觉，还是刚进入大学，学习 C 的时候，直到我遇到了Java。对啊，我是Java程序员啊，怎么混入JavaScript的世界了，赶紧写几行JavaScript压压惊 。。。

好了，现在从最简单的程序开始写起，一步一步来，看看到底哪一步出什么问题了，看看各种babel工具是干嘛的。

。。。（省略各种纠结的解决方案 ，ORZ。。。）

经过好久的纠结排查中，终于发现问题在 route的引用上。

`
React Router 配置的 route 是 jsx 的语法，在 node 环境中，引入该语法的文件，如果直接引用过来，会报语法异常，所以需要加上 babel-core 里面的 register 这个 hook 进行转换。
`

阮一峰的博客里面，有提及到这个 register ， 但是他用的babel-register，而我使用 babel-core里面的register ，也可以成功跑起来。暂未知两个具体区别是什么。待之后探究。

目前最简单的一个 React Router 的服务端框架已经可以展现出来了。

### 基础的环境
packge.json:
```
{{
  "name": "react-router-startup",
  "description": "A Startup Devlopment Kit for React Router",
  "version": "1.0.0",
  "devDependencies": {
    "babel-cli": "^6.9.0",
    "babel-core": "^6.9.1",
    "babel-preset-es2015": "^6.9.0",
    "babel-preset-react": "^6.5.0",
    "babel-watch": "^2.0.2"
  },
  "dependencies": {
    "express": "^4.13.4",
    "react": "^15.1.0",
    "react-dom": "^15.1.0",
    "react-router": "^2.4.1",
    "swig": "^1.4.2"
  },
  "scripts": {
    "start": "babel-node app.js",
    "dev": "babel-watch app.js"
  }
}


```
* babel-cli 里面有个 babel-node ,比较重要的工具。
* babel-watch 动态监视文件变化
* babel-preset-es2015 es2015转换
* babel-preset-react react转换

app.js:
```
// Babel 6 Compile
require('babel-core/register')({
   presets: ['es2015', 'react']
});

import express from 'express';
import React from 'react';
import ReactDOM from 'react-dom/server';
import {match ,RouterContext} from 'react-router';
import swig from 'swig';
var route = require('./view/route'); //注意这一段怪怪的引用

const app = express();

//服务端渲染
app.use((req,res) => {
   match({routes:route.default , location:req.url},(error, redirectLocation, renderProps) => {
      if (error) {
         res.status(500).send(error.message)
      } else if (redirectLocation) {
         res.redirect(302, redirectLocation.pathname + redirectLocation.search)
      } else if (renderProps) {
         var html = ReactDOM.renderToString(React.createElement(RouterContext, renderProps));
         var page = swig.renderFile('template/index.html',{html:html});
         res.status(200).send(page);
      } else {
         res.status(404).send('Not found')
      }
   });
});

app.listen(3000,()=>{
   console.log('server running');
});
```

route.js:
```
import React from 'react';
import {Route} from 'react-router';
import App from './component/App';
import Home from './component/Home';
import Content from './component/Content';

export default (
    <Route component={App}>
        <Route path="/" component={Home}/>
        <Route path="/content" component={Content}/>
    </Route>
)

```

基础环境这样，用新语法去写，一切ok了。

当初试验写新语法，在撸PhotoHub时候，按照阿里前端的那篇文章，搬来了很多babel插件。现在测试了下，用这个方式，babel的插件一个没有用到，已经可以支持node使用新语法，以及服务端的渲染了。

全部代码已放入github，[传送门](https://github.com/ThinkCats/ReactRouterServerRender)，希望能给同样在大海中漂泊的小白们带来点亮光。。。

一天晚上，整理了两篇文章，业已很困，酸奶和煤球都已经睡得呼呼了。
