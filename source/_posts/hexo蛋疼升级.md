---
title: hexo蛋疼升级
date: 2015-03-06 01:16:53
tags:
  - hexo
---
蛋疼的升级。。

一开始是莫名的插件错误，让我好生看源码，看着看着发现不对啊这为什么报错只能是api问题啊

然后到hexo.io一看，果然。。升级了3.0，还是"the API has breaking changes"的节奏

按照文档里写的去掉版本号，`npm install hexo-cli -g` ，`npm install hexo --save` ，然后发现folder又要重新出屎化，出屎化完了还要重新安装全套插件，还要重新设置环境变量。。。

我说你一个本地跑的静态博客弄这么多api变化干嘛呢。。。