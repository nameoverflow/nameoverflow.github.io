---
title: 把多说整合进 React
date: 2015-11-01 18:30:29
tags:
  - react
  - javacript
---
由于使用了 React 构建 Single Page Application，多说那种通过每次页面加载完成后在header中插入script标签来实现评论框的插件无疑是不能直接用了，我的博客便有了长达数个月的无法评论状态。



这种情况终于在10月的最后一天结束。



<!--more-->



### 多说代码分析



多说提供的傻瓜式插件代码的作用，是创建一个script标签，设置一个全居的duoshuoQuery变量，然后将script插入页面的head中，之后的事情完全交给加载的脚本文件完成。



这种方法简单却实简单，但是对于spa来说会造成重复插入script标签。虽然一两个并没有什么太大的影响，但毕竟看着不舒服，而且一旦页面跳转次数多了就日狗了。



### 评论框API



在多说的[开发者文档](http://dev.duoshuo.com/docs/50b344447f32d30066000147)中，有关于“动态载入评论框”的说明。从所给解决办法中可以看出，多说在head中引入的embed.js脚本在找不到“傻瓜式”解决方案中的特定id元素的时候，会只在全局引入一个DUOSHUO对象供API调用。



`DUOSHUO.EmbedThread` 方法接收一个`HTML DOM` 元素作为参数，读取其中的几个特定属性然后在这个元素中渲染多说评论框。



那么接下来的事情就简单了。在HTML head中引入多说脚本，构建一个 `DuoshuoComment` 组件，在 `DOM` 属性中提供规定的属性值，在其生命周期中的 `componentDidMount` 时期调用 `DUOSHUO.EmbedThread` 方法即可。



### Talk is Cheap Show Me the Code



```javascript

export default class Comment extends React.Component {

    constructor(props) {

        super(props);

    }

    componentDidMount() {

        try {

            DUOSHUO.EmbedThread(React.findDOMNode(this));

        } catch (e) {

            console.log(e);

            DUOSHUO.EmbedThread(React.findDOMNode(this));

        }

    }

    render() {

        return (

            <div

                data-thread-key={this.props.thread}

                data-url={this.props.url}

            />

        );

    }

}



```