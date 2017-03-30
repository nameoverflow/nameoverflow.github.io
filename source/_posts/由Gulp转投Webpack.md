---
title: 由Gulp转投Webpack
date: 2016-01-25 21:56:00
tags:
  - webpack
  - react
---
本站原本使用 `gulp` 进行构建，今天终于下定决心迁移到了 `Webpack` 并用上了高大上的 `react-hot-loader`。

<!--more-->

### Webpack Config

Webpack的理念总结起来就是“啥都能给你打包”，突破了传统的脚本、资源、样式分别发布的观念。一个明显的表现就是它的 `loader` 体系，理论上任何资源都有对应的 `loader` 进行处理，打包发布。

Webpack的文档是出名的烂，于是我放弃了从文档入手，直接找到了 `react-hot-loader` 的一个示例[react-hot-boilerplate](https://github.com/gaearon/react-hot-boilerplate)。

它的webpack.config.js长这样：

```js
var path = require('path');
var webpack = require('webpack');

module.exports = {
  devtool: 'eval',
  entry: [
    'webpack-dev-server/client?http://localhost:3000',
    'webpack/hot/only-dev-server',
    './src/index'
  ],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: '/static/'
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ],
  module: {
    loaders: [{
      test: /\.js$/,
      loaders: ['react-hot', 'babel'],
      include: path.join(__dirname, 'src')
    }]
  }
};
```

可以看出，webpack对于文件入口的配置形式有点类似grunt，直接用键值对定义输入、输出路径。`module` 中的明显是大名鼎鼎的 `loader` 配置对于不同文件类型采用不同的处理模块，`react-hot` 赫然在其中。而那个 `plugins` ，当然就是与 `hot loader` 配合的插件了。

依葫芦画瓢很容易写出用于自己工程的配置文件。

```js
var path = require('path');
var webpack = require('webpack');

var script_src = path.resolve(__dirname, 'src', 'script');
var style_src = path.resolve(__dirname, 'src', 'style');

module.exports = {
  devtool: 'eval',
  entry: {
    main: [
    'webpack-dev-server/client?http://localhost:3000',
    'webpack/hot/only-dev-server',
    './src/script/main'
    ],
    admin: './src/script/admin'
  },
  output: {
    path: path.join(__dirname, 'public', 'script'),
    filename: '[name].js',
    publicPath: '/static/script/'
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ],
  module: {
    loaders: [{
      test: /\.jsx?$/,
      loaders: ['react-hot', 'babel?presets[]=react,presets[]=es2015'],
      include: script_src
    }, {
      test: /\.sass$/,
      loaders: ['style', 'css', 'sass?indentedSyntax'],
      include: style_src
    }, {
      test: /\.s?css$/,
      loaders: ['style', 'css', 'sass'],
      include: style_src
    }]
  },
  resolve: {
    extensions: ['', '.js', '.jsx'],
    root: [
      style_src,
      script_src
    ]
  }
};
```

这个过程中发现自己的目录设计很不科学，还是使用的传统的脚本与样式分别放置，导致不得不把脚本和样式的根目录都设为 `root` 来方便使用。

另外，`node-sass` 有个暗坑——直接使用 `sass-loader` 默认是采用scss语法，导致有洁癖的我写的sass文件统统报错，需要特别设置为 `indentedSyntax` 模式。

### Dev Server


示例的目录中还有一个server.js

```js
var webpack = require('webpack');
var WebpackDevServer = require('webpack-dev-server');
var config = require('./webpack.config');

new WebpackDevServer(webpack(config), {
  publicPath: config.output.publicPath,
  hot: true,
  historyApiFallback: true
}).listen(3000, 'localhost', function (err, result) {
  if (err) {
    console.log(err);
  }

  console.log('Listening at localhost:3000');
});
```

看到这里，即使不看源码也能推测出 `react-hot-loader` 的工作模式：利用webpack提供的dev server这个东西启动一个本地服务器，运行过程中监听文件变化通过服务器将模块的变化通知浏览器，浏览器运行替换后的代码实现热更新。看到webpack配置中的那个eval，就可以猜到是直接eval代码了——看运行中的console也确实如此。

然而我的博客出于调用后台接口的需要，开发时也会在本地运行一个 `yue.js` 的服务器，所以肯定不能直接跑 `WebpackDevServer` 来调试。如果有个对特定url进行代理的功能就太好了。

查阅文档，只需在配置参数中加入一个 `proxy` 字段即可。

```js
var webpack = require('webpack');
var WebpackDevServer = require('webpack-dev-server');
var config = require('./webpack.config');

new WebpackDevServer(webpack(config), {
  publicPath: config.output.publicPath,
  hot: true,
  historyApiFallback: true,
  proxy: {
      "/": "http://127.0.0.1:4000",
      // Proxies...
  }
}).listen(3000, 'localhost', function (err, result) {
  if (err) {
    console.log(err);
  }

  console.log('Listening at localhost:3000');
});
```

此处有坑：`publicPath` 的作用。

一开始我直接将其设置为生产环境下的静态文件目录，运行报错，脚本文件404。

查阅文档得知它应该设置为发布文件的目录——我的主文件目录是 `./src/script/main.js` ，故设成 `/static/script/` 搞定。

那么，来测试一下

![我要开始装逼了](https://dn-hcyue.qbox.me/img%2F00.gif)



![看](https://dn-hcyue.qbox.me/img/QQ%E6%88%AA%E5%9B%BE20160125214545.png)

这样

![加一行代码](https://dn-hcyue.qbox.me/img/QQ%E6%88%AA%E5%9B%BE20160125214647.png)

加一行文字，保存

![YOOOOOOOAMAZING](https://dn-hcyue.qbox.me/img/QQ%E6%88%AA%E5%9B%BE20160125214658.png)

YOOOOOOOAMAZING!!!!!（参加 `react-hot-loader` 发布演示时现场欢呼）

![enter image description here](https://dn-hcyue.qbox.me/img/QQ%E6%88%AA%E5%9B%BE20160125215306.png)

（可见`Websocket` 对生产力发展的重要性）