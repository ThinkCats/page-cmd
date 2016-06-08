---
title: NodeJs & React 服务器端渲染开发环境
date: 2016-06-09 07:22:11
categories: JavaScript
tags: ReactJS, nodemon
---
昨天晚上，我对樱泡说：“完了,我已经对 Nodejs + Reactjs服务端渲染的开发环境丧失信心了，我已经放弃了。” 然后自暴自弃，破罐破摔，莫名其妙地去看Emacs去了。。。 找虐！

刚刚觉得心有不甘，去搞了一下，突然间想起了 nodemon ，顿时豁然开朗，有解决方案了。

如果用传统的babel-node 去跑的话，node是可以良好运行的，但是不利于调试。每次修改掉组件jsx之后，必须手动重启应用，才可以加载到修改后的东西。 nodemon 类似supervisor ，可以监控修改，自动重启,但是功能貌似更强，官方说可以支持任何语言。

解决方案很简单：
* 安装命令： `npm install nodemon --save-dev`
* package.json的script配置一下: `"dev" :"nodemon src/app.js --exec babel-node --presets es2015",`

然后就放手去搞吧， npm run dev ！

现成栗子已准备好，github[传送门](https://github.com/ThinkCats/ReactRouterServerRender)
