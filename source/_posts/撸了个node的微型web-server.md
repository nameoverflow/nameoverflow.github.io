---
title: 撸了个node的微型web server
date: 2015-08-03 01:16:00
tags:
  - nodejs
---
[Github 链接在此](https://github.com/NameIsUseless/yue)

（明明是全是coffee文件强行叫.js，凑表碾）

-------------------

> yue.js is a tiny web server based on Node.js and CoffeeScript, supporting `route`, `session`, `database` (based on mongodb and is not completed yet..) and `static file`.

已经实现的功能有：路由(`Router`)，静态文件(`Static File`)， Session相关。

Router用了一种简单粗暴的实现，即把输入的uri解析之后拼成正则保存下来，接受请求的时候一个个正则去match。

主要的想法是做成松散的结构，支持各种中间件，虽然现有的核心功能都是写的比较死的，但是后续的扩展方式应该是各种中间件。

其中把对request和response的相关操作封装在了一个connection对象里，各种中间件可以给它添加属性方法之类。

这东西其实是为了我的新版博客写的，然后新版博客是想做成spa，于是还有一堆前端的东西要写=。=

（然而说好的函数式信徒结果写成了依赖副作用的架构）

（以后再重构吧）

（以后===说不定就忘记了）