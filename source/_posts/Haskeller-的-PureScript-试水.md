---
title: Haskeller 的 PureScript 试水
date: 2016-05-08 00:21:33
tags:
  - haskell
  - purescript
  - functional programming
---
-- 讲道理虽然自称 Haskeller ，但是其实我根本不会 Haskell（

> 为什么我要尝试 PureScript ？
> 
> 因为它是 Haskell 和 JavaScript 生的娃啊。

Facebook 的 React.js 可以说是给 JavaScript 社区带来了一次 functional programming 的热潮。然而 JavaScript 毕竟不是真正的函数式编程语言——虽然有着 closure 等特性让它可以支持这种独特的范式，但是整体而言它还是基于顺序指令和事件回调的语言。

PureScript 是一门年轻的语言——现在在 github 上可以查到的最早提交来自于 30 Sep 2013 。它致力于创造一种“类似 haskell 的编程体验”和“生成可读的 JavaScript 代码”。于是，有了它，我们终于可以在项目前端部分中愉快的写“ monoid in the category of endofunctors of X, with product × replaced by composition of endofunctors and unit set by the identity endofunctor ”了（

<!--more-->
## 上手

如果“这门语言的目的是创造类似 haskell 的编程体验”这句话所言不虚的话，那么，它确实做到了。熟悉 Haskell 的初学者是完全可以平滑迁移的。

在项目主页上有这样一个栗子：

```haskell
import Control.Monad.Aff
import Control.Monad.Aff.Par

data Model = Model (List Product)

loadModel = do
    popular <- get "/products/popular"
    products <- runPar $ for_ popular \product -> Par (get product.uri)
    return $ Model products
```

和 Haskell 的相似程度不言而喻。

## 差异

PureScript 的 [wiki](https://github.com/purescript/purescript/wiki/Differences-from-Haskell) 上列出了几点和 Haskell 的不同之处。

除了 List 、 Tuple 这些语法糖的不同之外，最能体现 PureScript 与 JavaScript 亲密关系的就是 PureScript 对 Record 的加强。

Haskell 中的 Record 在创建类型时会自动创建各字段对应的函数

```haskell
data Point = Point { x :: Number, y :: Number }
```

这样的声明会创建三个东西：类型构造器`Point`和 `x` 、 `y` 字段对应的两个函数

```haskell
Point :: Number -> Number -> Point
x :: Point -> Number
y :: Point -> Number
```

而 PureScript 直接给了一个方便的 `.` 操作符，直接可以像操作 JavaScript 对象一样操作 Record

```haskell
originX :: Number
originX  = origin.x
```

不过这样就导致 Haskell 中常用的用来构建复合函数的 `.` 被占用了，于是 PureScript 给了它另外的名字：`<<<`

然后，在语法上， PureScript 有一个很蛋疼的点：不支持在 `let` 或者 `where` 中的 type destructure 。

在 Haskell 中，如果你想拿到一个类型里面的值，除了在函数参数里面写模式匹配外，还可以在函数里面写模式匹配的绑定

```haskell
data SomeType a = SomeType a

fuck (SomeType a) = a

fuckU theAWrapper =
    let SomeType a = theAWrapper in
        a
```

然而在 PureScript 中并不可以这样，想在 `let` 里面拆装只能用 `case of`

```haskell
fuckIt theAWrapper =
    let SomeThing a = theAWrapper in
        a -- BOOM!!!!

shit theAWrapper =
    let a = case theAWrapper of SomeThing b -> b in
        a -- 妈的智障

```

据[作者说](https://github.com/purescript/purescript/issues/764)不加这个糖的原因是没有可以生成优雅的 js 的脱糖模式……

（于是我们用 case of 写的话生成的代码就很优雅吗……）


[随地挖坑有空填]
