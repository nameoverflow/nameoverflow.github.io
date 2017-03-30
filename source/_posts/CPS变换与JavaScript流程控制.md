---
title: CPS变换与JavaScript流程控制
date: 2016-02-21 22:26:38
tags:
  - javascript
---
正如[上一篇文章](https://hcyue.me/article/56a8aff28e0d54a562c73ca8)所说，利用CPS原理可以进行流程控制。那么在JavaScript编程中究竟如何消灭callback hell呢？

[co](https://github.com/tj/co) 正是一个解决JavaScript异步流程难题的库。它利用ES2015标准中的 `yield`，实现了与（本来被期望加入ES2016标准却因为大厂跳票没赶上deadline所以只能明年再说的）`async/await` 相似的功能而红透半边天。其原理就是利用 `yield` 的状态机特性，将（看起来是）同步的代码转换为CPS的形式。

co的源文件有两百多行，而其核心思想其实非常简洁——不断的将 `yield` 的 `next` 值追加到 `promise` 链中，达到“一步接一步”执行的效果。

这个核心思想用coffeescript写出来不到十行：

```coffeescript
fuck = (gen) ->
    it = do gen
    go = (res) =>
        p = it.next(res)
        if p.done
            return
        p.value.then go
    do go
```

<!--more-->
使用起来很简单，在异步语句前面都加上 `yield` 即可。比如下面的代码很简单的实现了每一毫秒按顺序 `log` 100个数

```javascript
let shit = i =>
    new Promise((res, rej) => setTimeout(() => res(i), 1))

fuck(function *() {
    for (let i = 0; i < 100; i++) {
        let holy = yield shit(i)
        console.log(holy)
    }
    console.log("done")
})
```
![运行结果](https://dn-hcyue.qbox.me/img/QQ%E6%88%AA%E5%9B%BE20160204173750.png)

（运行结果，so beautiful）

看出来了吧？其实它的效果就是把每个 `yield` 断开之后的代码都写到 `Promise::then` 里，其实就相当于

```js
~function () {
    // 第一次next()
    var p = shit(0).then(holy => console.log(holy))
    for (let i = 1; i < 100; i++) {
        p.then(shit(i).then(holy => console.log(holy)))
    }
    p.then(holy => console.log("done!"))
}()
```

由于 `Promise` 的修饰，这段代码看起来不是那么的CPS。如果能把 `Promise` 拿掉就好了。如何拿掉 `Promise` 呢？

从形式上来看，每次添加单个 `then` 的 `Promise` 和 `callback` 调用长得差不多，只是把表示 `continuation` 的参数移到了 `then` 中

```javascript
doSomeThingWithPromise().then(callback)

doSomeThingWithCb(callback)
```

……那么即使不进行深层次考虑，也很容易实现从 `Promise` 到 `callback` 的转化：

```coffeescript
fuck = (gen) ->
    it = do gen
    go = (res) =>
        p = it.next res
        if p.done
            return
        p.value go
    do go

```

不就是拿掉一个 `then` 嘛！

用起来也就是去掉了`Promise`
```javascript
let shit = i => callback =>
    setTimeout(() => callback(i), 1)

fuck(function *() {
    for (let i = 0; i < 100; i++) {
        let holy = yield shit(i)
        console.log(holy)
    }
    console.log("done")
})
```

它的等价调用便是典型的Continuation Passing Style
（其实`Promise` 本身便是一种CPS变换）

```javascript
~function () {
    let p = (next) => next()
    for (let i = 0; i < 100; i++) {
        let c = ((p) => // 此处的闭包是考虑到js的执行策略，避免陷入无限递归
            (next) => p(() => 
                shit(i)((holy) => {
                    console.log(holy)
                    next()
                })
            ))(p)
        p = c
    }
    p(holy => console.log("done!"))
}()
```

So beautiful! tj 真是个天才！
