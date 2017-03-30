---
title: 学习笔记：Continuation-passing Style
date: 2016-01-27 19:54:26
tags:
  - cps
  - computer science
---
说来惭愧，虽然对于 `CPS transform` 这个概念略有耳闻（当时的理解是一种将普通递归调用变化为尾递归的方法），但是对于 `CPS` 这个概念还是在看[轮子哥的文章](http://www.cppblog.com/vczh/archive/2014/01/19/205468.html) 时才想起去详细了解。终究还是CS基础不牢。

### Definition

> In <a href="https://en.wikipedia.org/wiki/Functional_programming" title="Functional programming">functional programming</a>, <b>continuation-passing style</b> (<b>CPS</b>) is a style of programming in which <a href="https://en.wikipedia.org/wiki/Control_flow" title="Control flow">control</a> is passed explicitly in the form of a <a href="https://en.wikipedia.org/wiki/Continuation" title="Continuation">continuation</a>. This is contrasted with <a href="https://en.wikipedia.org/wiki/Direct_style" title="Direct style">direct style</a>, which is the usual style of programming. <a href="https://en.wikipedia.org/wiki/Gerald_Jay_Sussman" title="Gerald Jay Sussman">Gerald Jay Sussman</a> and <a href="https://en.wikipedia.org/wiki/Guy_L._Steele,_Jr." title="Guy L. Steele, Jr.">Guy L. Steele, Jr.</a> coined the phrase in <a href="https://en.wikipedia.org/wiki/AI_Memo" title="AI Memo">AI Memo</a> 349 (1975), which sets out the first version of the <a href="https://en.wikipedia.org/wiki/Scheme_(programming_language)" title="Scheme (programming language)">Scheme</a> programming language.<sup id="cite_ref-1" class="reference"><a href="https://en.wikipedia.org/wiki/Continuation-passing_style#cite_note-1"><span>[</span>1<span>]</span></a></sup><sup id="cite_ref-2" class="reference"><a href="https://en.wikipedia.org/wiki/Continuation-passing_style#cite_note-2"><span>[</span>2<span>]</span></a></sup> <a href="https://en.wikipedia.org/wiki/John_C._Reynolds" title="John C. Reynolds">John C. Reynolds</a> gives a detailed account of the numerous discoveries of continuations.<sup id="cite_ref-3" class="reference"><a href="https://en.wikipedia.org/wiki/Continuation-passing_style#cite_note-3"><span>[</span>3<span>]</span></a></sup>

简单来说，就是一种程序调用方式，在每次函数调用时通过传参决定下一步执行的内容。

<!--more-->

[Wikipedia](https://en.wikipedia.org/wiki/Continuation-passing_style) 上有一个直观的例子显示 `CPS` 方式与 `Direct` 方式的区别

Direct:

```scheme
(define (pyth x y)
  (sqrt (+ (* x x) (* y y))))
```

CPS:

```scheme
(define (pyth& x y k)
  (*& x x (lambda (x2)
            (*& y y (lambda (y2)
                      (+& x2 y2 (lambda (x2py2)
                                  (sqrt& x2py2 k))))))))
```

其中 `+&` `*&` 等表示对应operator的CPS版本，即接受第三个参数。Wikipedia上亦提供了一段 transformer：

```scheme
(define (cps-prim f)
  (lambda args
    (let ((r (reverse args)))
      ((car r) (apply f
                 (reverse (cdr r)))))))
(define *& (cps-prim *))
(define +& (cps-prim +))
```

### Use and implementation

群众喜闻乐见的语言JavaScript中臭名昭著的 `callback hell` 其实就有点CPS的味道（当然不算是full CPS）。

```javascript
function fuckJson(url, callback) {
    makeAjax(url, (err, data) => err ? callback(err) : callback(null, data))
}

fuckJson("/holy-shit", (err, data) => {
    if(err) console.log("error!");
    addEntry(data);
})
```

CPS应用中最重要的地方当然不是引起开发者的怨念，而是在 functional programming 中进行控制流的优化（这个词是我乱说的，我也不知道准确的定义是啥……）

例如利用 CPS transform 实现将同步代码展开成回调地狱（2333333）[continuation](https://github.com/BYVoid/continuation)

例如 C# （包括现在的伪 EcmaScript 2016 实现）中的 `await`/`async` ，实质上也是一种形式的 CPS 变换（ES7是借助了Promise）

然后其它的黑魔法，以我的姿势水平也触及不到了……
