---
title: 前端面试：将嵌套的js数组转化为一维数组的许多种办法
date: 2015-06-06 08:49:21
tags:
  - javascript
---
事件起源于一道饿了么的前端面试题。

>编写一个方法，将嵌套的多维数组转化为一维数组输出。
>示例：`var a = [1, [2, [3, 4]], 5, 6];` 输出 `[1, 2, 3, 4, 5, 6]`

这种嵌套的数据结构的处理，最常规的无疑就是递归。例如：

```javascript
function parseArr(arr) {
    var i = 0;
    var result = [];
    for (i = 0; i < arr.length; i++) {
        if (arr[i] instanceof Array) {
            parseArr(arr[i]);
        } else {
            result.push(a[i]);
        }
    }
    return result;
}
```

递归虽然好写，但是劣势也广为人知。而且作为一道面试题，递归这种常规的解法当然是不够吸引眼球的（大雾），于是便有大神提出了js特色的解法：使用`toString`

>`toString()` 方法可把数组转换为字符串，并返回结果。

例如`[1, [2, [3, 4]], 5, 6].toString()` 返回 `"1,2,3,4,5,6"`，毫不费力的便转化成了一位数组的形式。

什么，这是一个字符串，需要的是一个数组？

`[1, [2, [3, 4]], 5, 6].toString().spilt(',').map(function(v){return +v});` 就是这样。

`join`方法也是同样的效果。

利用es6的`reduce`方法也能写出漂亮的解。

```javascript
function parseArr(arr) {
    return arr.reduce(function (prev, cur, index) {
        var deeparr = [].concat(cur).some(Array.isArray);
        return prev.contact(deeparr ? flatarray(cur) : cur);
    }, []);
}
```

然而，一位大神默默写了一个利用正则的神奇的解

```javascript
JSON.stringify([1,[2,3,[4,5]],6]).replace(/[\[\]]/g,"").split(",");
```

直接转化为字符串，将方括号删除，然后用json解析之，牛逼闪闪。

然而我是看到递归总想转化成循环的人，于是便写了一个不那么js的非递归方法

```javascript
function parseArr(arr) {
    while (!!a[0]) {
        tmp = a.pop();
        if (tmp instanceof Array) {
            for (i = 0;i < tmp.length; i++) {
                a.push(tmp[i]);
            }
        } else {
            result.push(tmp);
        }
    }
    return result.reverse();
}
```

直接把原数组当做栈，如果取出的元素是数组则把数组内元素一个个压入栈中，最后把结果反转一下。
