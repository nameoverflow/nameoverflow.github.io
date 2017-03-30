---
title: 千年弦歌网络工作室javascript编码规范
date: 2015-04-12 22:48:36
tags:
---
## 1 前言

本文档的目标是使javascript代码风格保持一致，容易被理解和被维护。

虽然本文档是针对javascript设计的，但是在使用各种javascript的预编译器(如coffescript等)
时，适用的部分也应尽量遵循本文档的约定。

<!--more-->

## 2 代码风格

### 2.1 文件

##### [建议] `javascript` 文件使用无 `BOM` 的 `UTF-8` 编码。

解释：

UTF-8 编码具有更广泛的适应性。BOM 在使用程序或工具处理文件时可能造成不必要的干扰。

### 2.2 结构

#### 2.2.1 缩进

##### [强制] 使用 `4` 个空格做为一个缩进层级，不允许使用 `2` 个空格 或 `tab` 字符。

##### [强制] switch 下的 case 和 default 必须增加一个缩进层级。

示例：

```javascript
// good
switch (variable) {

    case '1':
        // do...
        break;

    case '2':
        // do...
        break;

    default:
        // do...

}

// bad
switch (variable) {

case '1':
    // do...
    break;

case '2':
    // do...
    break;

default:
    // do...

}

```

##### [强制] `链式调用`较长时采用`缩进`进行调整。

```javascript

$('#items')
    .find('.selected')
    .highlight()
    .end();

```

#### 2.2.2 分号

##### [强制] 所有javascript表达式一律以`分号`结束，不可以省略。

#### 2.2.3 换行

##### [强制] 每个`独立语句`结束后必须换行。

##### [强制] 每行不得超过 `120` 个字符。

解释：超长的不可分割的代码允许例外，比如复杂的正则表达式。长字符串不在例外之列。


##### [强制] `方法`之间添加空行。

##### [强制] `单行`或`多行注释`前添加空行。

##### [建议] `逻辑块`之间应添加空行增加可读性。

示例：

```javascript
// 仅为按逻辑换行的示例，不代表setStyle的最优实现
function setStyle(element, property, value) {
    if (element == null) {
        return;
    }

    element.style[property] = value;
}
```

##### [强制] 在表示`语句块`开始的`{`之前不换行，在表示`语句块`结束的`}`后换行。

解释：特别的，对于 `if...else...`、`try...catch...finally` 等语句，在 `}` 
之后不换行，而是添加一个空格。

示例：

```javascript
function funcName() {
    // some statements;
}

if (...) {
    // some statements;
}

if (condition) {
    // some statements;
} else {
    // some statements;
}

try {
    // some statements;
} catch (ex) {
    // some statements;
}

```


##### [强制] `运算符`处换行时，`运算符`必须在新行的`行首`。

示例：

```javascript
// good
if (user.isAuthenticated()
    && user.isInRole('admin')
    && user.hasAuthority('add-admin')
    || user.hasAuthority('delete-admin')
) {
    // Code
}

var result = number1 + number2 + number3
    + number4 + number5;


// bad
if (user.isAuthenticated() &&
    user.isInRole('admin') &&
    user.hasAuthority('add-admin') ||
    user.hasAuthority('delete-admin')) {
    // Code
}

var result = number1 + number2 + number3 +
    number4 + number5;
```

##### [强制] 在`函数声明`、`函数表达式`、`函数调用`、`对象创建`、`数组创建`、`for语句`等场景中，不允许在 `,` 或 `;` 前换行。

示例：

```javascript
// good
var obj = {
    a: 1,
    b: 2,
    c: 3
};

foo(
    aVeryVeryLongArgument,
    anotherVeryLongArgument,
    callback
);


// bad
var obj = {
    a: 1
    , b: 2
    , c: 3
};

foo(
    aVeryVeryLongArgument
    , anotherVeryLongArgument
    , callback
);
```

##### [建议] 对于`HTML片段`的拼接，通过`换行`和`缩进`，保持和HTML相同的结构。

示例

```javascript
var html = '' // 此处用一个空字符串，以便整个HTML片段都在新行严格对齐
    + '<article>'
    +     '<h1>Title here</h1>'
    +     '<p>This is a paragraph</p>'
    +     '<footer>Complete</footer>'
    + '</article>';
```


#### 2.2.5 空格

##### [强制] `二元运算符`两侧必须有`一个空格`，`一元运算符`与`操作对象`之间不允许有空格。

示例：

```javascript
var a = !arr.length;
a++;
a = b + c;
```

##### [强制] 用作`代码块`起始的左花括号 `{` 前必须有一个空格。

示例：

```javascript
// good
if (condition) {
}

while (condition) {
}

function funcName() {
}

// bad
if (condition){
}

while (condition){
}

function funcName(){
}
```

##### [强制] `if / else / for / while / function / switch / do / try / catch / finally` 关键字后，必须有一个空格。

示例：

```javascript
// good
if (condition) {
}

while (condition) {
}

(function () {
})();

// bad
if(condition) {
}

while(condition) {
}

(function() {
})();
```

##### [强制] 在对象创建时，属性中的 `:` 之后必须有空格，`:` 之前不允许有空格。

示例：

```javascript
// good
var obj = {
    a: 1,
    b: 2,
    c: 3
};

// bad
var obj = {
    a : 1,
    b:2,
    c :3
};
```

##### [强制] `函数声明`、`具名函数表达式`、`函数调用中`，`函数名`和 `(` 之间`不允许`有空格。

示例：

```javascript
// good
function funcName() {
}

funcName();

// bad
function funcName () {
}

funcName ();
```

##### [强制] 使用`函数表达式`声明函数或使用`匿名函数`时，关键词`function`之后应有空格。

示例：

```javascript
// good
var funcName = function () {
};
var value = (function () {
}());

// bad
var funcName = function() {
};
var value = (function() {
}());
```

##### [强制] `,` 和 `;` 前不允许有空格，`,`之后应有一个空格。

示例：

```javascript
// good
callFunc(a, b);

// bad
callFunc(a , b) ;

callFunc(a,b);
```

##### [强制] 在函数调用、函数声明、括号表达式、属性访问、`if / for / while / switch / catch` 等语句中，`()` 和 `[]` 内紧贴括号部分不允许有空格。

示例：

```javascript
// good

callFunc(param1, param2, param3);

save(this.list[this.indexes[i]]);

needIncream && (variable += increament);

if (num > list.length) {
}

while (len--) {
}


// bad

callFunc( param1, param2, param3 );

save( this.list[ this.indexes[ i ] ] );

needIncreament && ( variable += increament );

if ( num > list.length ) {
}

while ( len-- ) {
}
```

##### [强制] 单行声明的`数组`与`对象`，如果包含元素，`{}` 和 `[]` 内紧贴括号部分`不允许`包含空格。

解释：

声明包含元素的数组与对象，只有当内部元素的形式较为简单时，才允许写在一行。元素复杂的情况，还是应该换行书写。


示例：

```javascript
// good
var arr1 = [];
var arr2 = [1, 2, 3];
var obj1 = {};
var obj2 = {name: 'obj'};
var obj3 = {
    name: 'obj',
    age: 20,
    sex: 1
};

// bad
var arr1 = [ ];
var arr2 = [ 1, 2, 3 ];
var obj1 = { };
var obj2 = { name: 'obj' };
var obj3 = {name: 'obj', age: 20, sex: 1};
```

##### [强制] `行尾`不得有多余的空格。

### 2.3 命名

#### 2.3.1 命名风格

##### [强制] 变量使用`下划线`命名法。

示例：

```javascript
var value_name_like_this;
```
##### [强制] 函数名使用`Camel`命名法

```javascript
function funcNameLikeThis(){
    //code....
}
```

##### [强制] `常量命名`全部字母`大写`，单词间用`下划线`分隔。

示例：

```javascript
var CONSTANT_NAME_LIKE_THIS;
```

##### [强制] `函数`的`参数`使用`下划线`命名法。

示例：

```javascript
function funcName(param_name_like_this) {
    //code....
}
```


##### [强制] `类名`（构造函数）使用`Pascal`命名法。

示例：

```javascript
function ClassNameLikeThis(){
    //code....
}
```

##### [强制] `类的方法`（属性）使用`Camel`命名法。

示例：

```javascript
function ClassNameLikeThis(){
    this.valueNameLikeThis = value;
    this.functionInClass = function () {
    //code....
    }
}
```

##### [强制] `枚举变量`使用`Pascal`命名法，枚举的`属性`使用`全部字母大写`，单词间`下划线分隔`的命名方式。

示例：

```javascript
var TargetState = {
    READING: 1,
    READED: 2,
    APPLIED: 3,
    READY: 4
};
```

##### [强制] `命名空间`使用`Camel`命名法。

示例：

```javascript
var nameSpace = {};
```

##### [强制] 由多个单词组成的`缩写词`，在命名中，根据当前命名法和出现的位置，所有字母的大小写与首字母的大小写`保持一致`。

示例：

```javascript
function XMLParser() {
}

function insertHTML(element, html) {
}

var httpRequest = new HTTPRequest();
```

#### 2.3.2 命名语义

##### [强制] `类名`使用`名词`。

示例：

```javascript
function Engine(options) {
}
```

##### [建议] `函数名` 使用 `动宾短语`。

示例：

```javascript
function getStyle(element) {
}
```

##### [建议] `boolean` 类型的变量使用 `is` 或 `has` 开头。

示例：

```javascript
var isReady = false;
var hasMoreCommands = false;
```

##### [建议] `Promise对象` 用 `动宾短语的进行时` 表达。

示例：

```javascript
var loadingData = ajax.get('url');
loadingData.then(callback);
```

## 3 语言特性

### 3.1 变量

##### [强制] 变量在使用前必须通过`var`定义。

解释：

不通过 var 定义变量将导致变量污染全局环境。

示例：

```javascript
// good
var name = 'MyName';

// bad
name = 'MyName';
```

##### [强制] 声明多个变量时必须在`,`后换行，或使用多个`var`定义。

示例：

```javascript
// good
var hangModules = [];
var missModules = [];
var visited = {};
var number_one = 1,
    number_two = 2;

// bad
var hangModules = [], missModules = [], visited = {}, number_one = 1, number_two = 2;
```

##### [强制] 在同一个函数内部，`局部变量`的声明必须置于`顶端`。

解释：在顶部统一声明变量有助于增强可读性，即使放到中间，js解析器也会提升至顶部（hosting）

示例：

```javascript
// good
function kv2List(source) {
    var list = [];
    var key;
    var item;

    for (key in source) {
        if (source.hasOwnProperty(key)) {
            item = {
                k: key,
                v: source[key]
            };
            list.push(item);
        }
    }

    return list;
}
// bad
function kv2List(source) {
    var list = [];

    for (var key in source) {
        if (source.hasOwnProperty(key)) {
            var item = {
                k: key,
                v: source[key]
            };
            list.push(item);
        }
    }

    return list;
}

```

### 3.2 条件

##### [建议] 在 Equality Expression 中使用类型严格的 ===。

解释：

使用 === 可以避免等于判断中隐式的类型转换。

示例：

```javascript
// good
if (age === 30) {
    // ......
}

// bad
if (age == 30) {
    // ......
}
```

##### [建议] 尽可能使用简洁的表达式。

示例：

```javascript
// 字符串为空

// good
if (!name) {
    // ......
}

// bad
if (name === '') {
    // ......
}
// 字符串非空

// good
if (name) {
    // ......
}

// bad
if (name !== '') {
    // ......
}
// 数组非空

// good
if (collection.length) {
    // ......
}

// bad
if (collection.length > 0) {
    // ......
}
// 布尔不成立

// good
if (!notTrue) {
    // ......
}

// bad
if (notTrue === false) {
    // ......
}
// null 或 undefined

// good
if (noValue == null) {
  // ......
}

// bad
if (noValue === null || typeof noValue === 'undefined') {
  // ......
}
```

### 3.3 循环

##### [建议] 不要在循环体中包含函数表达式，事先将函数提取到循环体外。

解释：

循环体中的函数表达式，运行过程中会生成循环次数个函数对象。

示例：

```javascript
// good
function clicker() {
    // ......
}

for (var i = 0, len = elements.length; i < len; i++) {
    var element = elements[i];
    addListener(element, 'click', clicker);
}


// bad
for (var i = 0, len = elements.length; i < len; i++) {
    var element = elements[i];
    addListener(element, 'click', function () {});
}
```

##### [建议] 对循环内多次使用的不变值，在循环外用变量缓存。

示例：

```javascript
// good
var width = wrap.offsetWidth + 'px';
for (var i = 0, len = elements.length; i < len; i++) {
    var element = elements[i];
    element.style.width = width;
    // ......
}


// bad
for (var i = 0, len = elements.length; i < len; i++) {
    var element = elements[i];
    element.style.width = wrap.offsetWidth + 'px';
    // ......
}
```

### 3.4 类型

##### [建议] 转换成`string`时，使用`+ ''`。

示例：

```javascript
// good
num + '';

// bad
new String(num);
num.toString();
String(num);
```

##### [建议] 转换成`number`时，通常使用`+`。

示例：

```javascript
// good
+str;

// bad
Number(str);
```

[建议] 转换成`boolean`时，使用`!!`。

示例：

```javascript
var num = 3.14;
!!num;
```

### 3.5 对象

##### [强制] 使用对象字面量`{}`创建新`Object`。

示例：

```javascript
// good
var obj = {};

// bad
var obj = new Object();
```

##### [建议] 属性访问时，尽量使用`.`。

解释：

属性名符合 Identifier 的要求，就可以通过 . 来访问，否则就只能通过 [expr] 方式访问。

通常在 JavaScript 中声明的对象，属性命名是使用 Camel 命名法，用 . 来访问更清晰简洁。部分特殊的属性(比如来自后端的JSON)，可能采用不寻常的命名方式，可以通过 [expr] 方式访问。

示例：

```javascript
info.age;
info['more-info'];
```

##### [强制] 对象创建时，如果任何一个`属性`需要添加引号，则所有`属性`均应添加 '。

示例：

```javascript
// good
var info = {
    'name': 'someone',
    'age': 28,
    'more-info': '...'
};

// bad
var info = {
    name: 'someone',
    age: 28,
    'more-info': '...'
};
```

##### [强制] 不允许修改和扩展任何原生对象和宿主对象的原型。


### 3.6 数组

[强制] 使用数组字面量`[]`创建新数组，除非想要创建的是指定长度的数组。

示例：

```javascript
// good
var arr = [];

// bad
var arr = new Array();
```

##### [强制] 遍历数组不使用`for in`。

解释：

数组对象可能存在数字以外的属性, 这种情况下 for in 不会得到正确结果.

示例：

```javascript
var arr = ['a', 'b', 'c'];
arr.other = 'other things'; // 这里仅作演示, 实际中应使用Object类型

// 正确的遍历方式
for (var i = 0, len = arr.length; i < len; i++) {
    console.log(i);
}

// 错误的遍历方式
for (i in arr) {
    console.log(i);
}
```

### 3.7 函数

##### [建议] 一个函数的长度控制在`50`行以内。

解释：

将过多的逻辑单元混在一个大函数中，易导致难以维护。一个清晰易懂的函数应该完成单一的逻辑单元。复杂的操作应进一步抽取，通过函数的调用来体现流程。

特定算法等不可分割的逻辑允许例外。

示例：

```javascript
function syncViewStateOnUserAction() {
    if (x.checked) {
        y.checked = true;
        z.value = '';
    }
    else {
        y.checked = false;
    }

    if (!a.value) {
        warning.innerText = 'Please enter it';
        submitButton.disabled = true;
    }
    else {
        warning.innerText = '';
        submitButton.disabled = false;
    }
}

// 直接阅读该函数会难以明确其主线逻辑，因此下方是一种更合理的表达方式：

function syncViewStateOnUserAction() {
    syncXStateToView();
    checkAAvailability();
}

function syncXStateToView() {
    if (x.checked) {
        y.checked = true;
        z.value = '';
    }
    else {
        y.checked = false;
    }
}

function checkAAvailability() {
    if (!a.value) {
        displayWarningForAMissing();
    }
    else {
        clearWarnignForA();
    }
}
```

### 3.8 面向对象

##### [建议] 使用`function`进行类的定义，不推荐继承，如需继承采用成熟的类库实现。

##### [建议] 属性在构造函数中声明，方法在原型中声明。

示例：

```javascript
function TextNode(value, engine) {
    this.value = value;
    this.engine = engine;
}

TextNode.prototype.clone = function () {
    return this;
};
```

### 4 DOM

#### 4.1 元素获取

##### [建议] 对于单个元素，尽可能使用`document.getElementById`获取，避免使用`document.all`。

##### [建议] 对于多个元素的集合，尽可能使用`context.getElementsByTagName`获取。其中`context`可以为 `document`或其他元素。指定`tagName`参数为 * 可以获得所有子元素。

##### [建议] 遍历元素集合时，尽量缓存集合长度。如需多次操作同一集合，则应将集合转为数组。

#### 4.2 样式

##### [建议] 尽可能通过为元素添加预定义的`className`来改变元素样式，避免直接操作`style`设置。

##### [强制] 通过`style`对象设置元素样式时，对于带单位非`0`值的属性，不允许省略单位。

#### 4.3 DOM操作

##### [建议] 尽量减少`DOM`操作。

解释：

`DOM`操作是非常耗时的一种操作，减少`DOM`操作有助于提高性能。举一个简单的例子，构建一个列表。我们可以用两种方式：

1. 在循环体中 createElement 并 append 到父元素中。

2. 在循环体中拼接 HTML 字符串，循环结束后写父元素的 innerHTML。

第一种方法看起来比较标准，但是每次循环都会对 DOM 进行操作，性能极低。在这里推荐使用第二种方法。
