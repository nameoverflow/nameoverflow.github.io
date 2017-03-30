---
title: 博客迁移：使用hexo建站
date: 2015-03-01 04:24:19
tags:
  - hexo
  - website
---
神烦wordpress那一大堆奇怪的插件和功能，何况不懂后台优化的我看到wp页面加载时那几十条http请求就头大，于是毅然决然的始乱终弃（= =）了wp，转投静态博客。

一番考虑之后选择了hexo。

理由之一是易于部署，并且风评很好= =

> Blazing Fast
> 
Node.js brings you incredible generating speed. Hundreds of files take only seconds to build.

> Markdown Support
> 
All features of GitHub Flavored Markdown are supported. You can even use most Octopress plugins in Hexo.

> One-Command Deployment
> 
You only need one command to deploy your site to GitHub Pages, Heroku or other sites.

> Various Plugins
> 
Hexo has a powerful plugin system. You can install more plugins for Jade, CoffeeScript plugins.

这便是官网的介绍。听着就很带感= =

<!--more-->

##安装使用

于是动手安装。

```
npm install hexo-cli -g
```

之后在本地新建文件夹blog。

```bash
$ cd blog
$ hexo init
$ npm install
```

便完成了hexo的本地部署。

用hexo写博客，其实只要知道四个命令。

```bash
$ hexo new [post/page] <title> #新建文章或者页面，文章会在source/_post下新建<title>.md，页面会生成在source下。自动生成日期等。
$ hexo generate #生成静态网站，将整个网站生成在public文件夹下，部署站点时直接将该文件夹下的文件放置在站点目录下即可
$ hexo clean #清除生成的静态文件和缓存，重新生成时使用。
$ hexo server #在本地开启一个简单的http服务器，访问http://127.0.0.1:4000即可实时看到网站生成后的效果。
```

我是从wp上迁移过来，hexo也有导入wp文章和页面的插件。

```bash
$ npm install hexo-migrator-wordpress --save
```

把wordpress导出的xml文件放在目录下改名wp.xml，运行

```bash
$ hexo migrate wordpress wp.xml
```

即可将wp内容导入hexo。

至此完成内容迁移。

##主题

当然,对于网站更重要的是着手制作主题。

hexo自带一套主题landscape；github上也有丰富的主题资源。

不过作为有志前端的青年当然要自己动手！

大致浏览一下自带主题的文件结构

<pre>
├─layout
│  ├─_partial
│  │  └─post
│  └─_widget
├─scripts
└─source
    ├─css
    │  ├─fonts
    │  ├─images
    │  ├─_partial
    │  └─_util
    ├─fancybox
    │  └─helpers
    └─js

</pre>

layout文件夹中是ejs模板文件；scripts中是脚本；source中是资源文件（在生成后其内的文件会出现在根目录下），css文件夹中是stylus样式文件。

layout文件夹的结构如下

<pre>
│  archive.ejs
│  category.ejs
│  index.ejs
│  layout.ejs
│  page.ejs
│  post.ejs
│  tag.ejs
│  
├─_partial
│  │  after-footer.ejs
│  │  archive-post.ejs
│  │  archive.ejs
│  │  article.ejs
│  │  footer.ejs
│  │  google-analytics.ejs
│  │  head.ejs
│  │  header.ejs
│  │  mobile-nav.ejs
│  │  sidebar.ejs
│  │  
│  └─post
│          category.ejs
│          date.ejs
│          gallery.ejs
│          nav.ejs
│          tag.ejs
│          title.ejs
│          
└─_widget
</pre>

ejs语法参考[http://www.embeddedjs.com/](http://www.embeddedjs.com/)

stylus参考[http://www.zhangxinxu.com/jq/stylus/](http://www.zhangxinxu.com/jq/stylus/)（中文）
[https://github.com/LearnBoost/stylus](https://github.com/LearnBoost/stylus)（github官方）

hexo主题的渲染方式是:先寻找layout.ejs，再根据具体的页面需求找到index.ejs（首页），archive.ejs（文章列表），post.ejs（文章页）等文件替换layout中的<%- body %>标签。_partial文件夹下则是各个模块组件，在ejs模板中用<%- partial('_partial/filename') %>语句导入。

制作主题还需要知道博客程序提供的接口。hexo为主题模板提供了page、post对象，包含title、content等属性，代表文章或者页面的标题、内容等，具体参考[http://blog.gfdsa.net/2013/04/09/hexo/hexolessontwo/](http://blog.gfdsa.net/2013/04/09/hexo/hexolessontwo/)

知道这些后，便可以拿着自己做html模板参考着默认主题改改改，制作出一套完整的hexo主题了。

hexo全静态的特征让它很适合部署在github等静态空间下，hexo也提供了相应的命令可以直接部署（具体参考官方页面），不过本着不浪费的原则我还是把它放在了vps上。。计划等vps到期后就迁移到github上。


