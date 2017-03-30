---
title: React Redux Isomorphic 踩坑记——数据同步
date: 2016-03-08 16:25:34
tags:
  - react
  - redux
  - javascript
  - node
---
同构渲染的好处大家都有目共睹，SEO 优化、首屏速度之类的。这次重写 blog，基于试水的考虑使用了 react + redux 实现同构渲染，然后果不其然踩到了点小坑。

### `state` 带来的难题
起初我只是想感受一下 react 的服务端渲染，并没有考虑把 redux 这种应对高复杂度应用的框架弄来。然而很快便发现，同构渲染要解决的首要问题——数据同步问题，在传统 react 的 props + state 模式下很难做到。

一般纯客户端使用 react 时，习惯将数据保存在 state 中，然而在服务端渲染时很难控制组件的 `state` （至少我在文档中根本没找到解决办法），于是难以将客户端的组件数据与服务端渲染时的数据保持一致。当时想到的一个办法就是将所有的状态全部抽出来由最顶层组件的state提供，然后以props的形式分发到下级组件。然后开始写之后发现，这™不就是 Redux ？于是便高高兴兴的 `npm install redux --save` 

### 在服务端分发数据

Readux 除了能解决数据同步的问题外，还顺带解决了服务端对不同组件数据分发的问题。比如之前对于加载文章的解决方式是：当组件挂载后，ajax拉下组件对应的文章数据保存在state中，然后渲染。

```js
export default class SingleArticle extends Component {
    constructor(props) {
        super(props);
        this.state = {/* Init */}
    }
    componentDidMount() {
        ajax.get(/*load*/)
            .then(/*handle*/)
    }
    render() {
        // ....
    }
}

```
然而在服务端是不会触发组件生命周期更不能ajax的。

那么应该如何在前后端公用一套代码的情况下为每个组件加载不同的数据呢？Redux 的另一个好处就体现出来了——它的 `action -> reducer` 逻辑是独立的，不与组件相关的。于是便可以单独为每种情况抽取“加载”的逻辑，对应不同的 `action` ，然后 `dispatch` ，世界和平。

顺着这种思路下去，又一个问题出现了——如何实现对于不同的 URL 请求，在服务端分发不同的“加载”逻辑，这一套“加载”逻辑还要™和客户端公用一套代码。幸好 `react-router` 已经给我们提供了用于服务端渲染的 API ，可以方便的获取当前应该渲染的组件。于是解决方法也浮出水面——将数据加载作为组件类的 `static` 方法（如 `static fetchData()` ，然后利用 router 的 api 获取当前组件类，调用 `fetchData()` 触发 `action` 。

这样加载数据的上层逻辑就统一了，但是服务端和客户端对于 `fetchData()` 的实现肯定不能一样——客户端使用ajax，服务端从model读取数据。

为了保证调用形式的统一，我们可以使用 `redux` 提供的 `middleware` 方案抹平差异——使用 `action` 触发 `middleware` ，而服务端和客户端分别各自实现。

最后的服务端形式便化为了这样

```js
// Component.jsx
@connect(/** Connect functions **/)
export default class Page extends Component {
	// ...other methods
    render() {
        const { data } = this.props
        return (
            <div>
                <ContentView>{ data }</ContentView>
                <Comment />
            </div>
        )
    }
    static fetchData(store, props) {
        const id = props.params.id
        return store.dispatch(loadAction(id))
    }
}

// server render

const matchRouter = (location, routes) => {
    return new Promise((res, rej) => {
        match({
            location,
            routes
        }, (error, redirectLocation, renderProps) =>
            res({error, redirectLocation, renderProps}))
    })
}
export default function* () {
    /** react-router 匹配 **/
    const {
        error,
        redirectLocation,
        renderProps
    } = yield matchRouter(this.url.path, routes)
    
    if (error) {
        /** 错误处理 **/
    } else if (redirectLocation) {
        /** 重定向 **/
    } else {
        const
            midd = applyMiddleware(apiFactory(makeRequest)),
            store = createStore(reducer, midd)
        // 获取将要渲染的组件
        const components = renderProps.components.filter(c => c && c.fetchData)
        // 分别触发各组件的加载方法
        yield Promise.all(components.map(c =>
                            c.fetchData(store, renderProps)))
        /** 渲染，发送response **/
    }
}
```

其中 `apiFactory` 的实现为

```js
export const CALL_API = Symbol('CALL_API')
export const apiFactory = makeRequest => store => next => action => {
    if (!action[CALL_API]) {
        return next(action)
    }
    const req = action[CALL_API]
    const { method, url, success, fail, extra } = req
    return makeRequest(url, { method })
        .then(/** handle data **/)
}
```

服务端环境和客户端环境各自传入不同的 `makeRequest` 实现，实现抹平上层差异。

为了偷懒，我在服务端是用 supertest 向自身发送 http 请求调用 api 实现。

### 将数据同步到客户端

解决服务端获取数据的问题后，终于到了最后一步——将数据同步到客户端。

解决办法非常的简单粗暴——渲染一个模板，把初始状态转为 json 字面量挂载到客户端的 `window` 对象属性下，客户端脚本读取初始状态生成store。

```js
/** 服务端 **/
export default function* () {
    const {
        error,
        redirectLocation,
        renderProps
    } = yield matchRouter(this.url.path, routes)
    if (error) {
    } else if (redirectLocation) {
    } else {
        const
            rendered = getRendered(store, renderProps),
            initial_state = JSON.stringify(store.getState())
        /** 渲染模板，发送数据 **/
        this.render('client', { rendered, initial_state })
    }
}

/** 客户端 **/
const
    initial_state = window.__INITIAL_STATE__,
    middleware = applyMiddleware(apiFactory(makeRequest)),
    store = createStore(reducer, initial_state, middleware),
    history = syncHistoryWithStore(browserHistory, store)
```

模板文件：

```jade
doctype html
html(lang="zh-cn")
    head

    body
        div#client!= rendered
        script!= 'window.__INITIAL_STATE__ = ' + initial_state
        script(src = static_path + "client.js")

```

于是，终于可以舒心的写真正的“客户端与服务端公用”的代码了……