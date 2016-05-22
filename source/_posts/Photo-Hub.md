---
title: 徒手撸一个Photo Hub（ReactJS + AltJS + Express ）(一)
date: 2016-05-22 10:51:34
categories: JavaScript
tags: ReactJS
---

昨天和今天，拿react和express撸了一个网站，现在只是把前端搞成react的了，各个组件已拆分好，后端也已经集成好。

之所以这么搞，是因为最近在公司做后台的时候，使用ReactJS + AltJS，深深感受到了这种方式的好处，代码简直太好维护了。

现在还没有加上AltJS，不过之前也已经搞过这么多了，这个不是什么大问题。

说说这个站点吧。PhotoHub是一个模板，关于图片站点的。先把站开发好，权当练手的，万一以后真能开起来了呢，对吧。

先YY下，如果能开发好，那图片呢，我自己上传肯定不够的。还要撸一个爬虫去爬，然后存到我这这里，然后在OSS啥的。爬虫之前也撸过，Node版本的爬cnblog的文字的，Golang版本的爬的某H站图片(纯碎来玩的...)，效果还良好...

总结下技术情况。

后端，Node还是没有完全支持 ES6，于是，使用了babel-node在后端转化了一下。参考淘宝的文章：《[找回 Node.js 里面那些遗失的 ES6 特性](http://taobaofed.org/blog/2016/01/07/find-back-the-lost-es6-features-in-nodejs/)》

前端，使用的是 ant-design 的命令行工具，antd-init，除了jquery在界面引用，其他的所有均作为js模块引入，包括css 和 自定义的js操作ui的片段。

其他的东西，之后慢慢总结。
