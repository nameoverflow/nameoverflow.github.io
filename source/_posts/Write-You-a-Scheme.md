---
title: Write You a Scheme
date: 2016-08-01 09:33:57
tags:
  - scheme
  - haskell
  - interpreter
  - lambda calculus
---
撸了个 [Scheme 解释器](https://github.com/nameoverflow/LittleScheme)，也算是拿 Haskell 做过东西了（虽然只是个玩具

最大的体会就是，既熟悉了 Haskell，也巩固了 Scheme （虽然看过 SICP 但是并不是很明白它的 quosiquote 和 call/cc 之类的鬼东西

本来是打算自己定义一门语言（像[这个](https://hcyue.me/article/56eb92c6d09a19dd0956c469)），但是发现挺麻烦的（大雾），而且我比较关心的也是解释执行的过程，于是还是决定把 Scheme 实现一下。大体上是跟着 [Write Yourself a Scheme in 48 Hours](https://en.wikibooks.org/wiki/Write_Yourself_a_Scheme_in_48_Hours) 来的，在它的基础上增加了 Continuation 之类的玩意儿

### 0. Parser

虽然 Lisp 的语法接近没有，但是还是总还是需要一个 Parser。Haskell 写Parser 的利器便是 `parsec` 。其实一开始有点想自己实现一个简单的 parser combiner ，但是最后还是懒（

Lisp 语法——所谓 S-expressions ，其实就是嵌套的列表，语法定义非常的简单

```text
List → Expr (spaces) List | Expr
Expr → '(' List ')' | Atom
```

当然其实还有 quote 、dotted 之类的语法糖，也就是多加几个分支而已。

接下来便开始愉快的写代码了。写 Haskell 第一步当然是考虑类型。Scheme 语言内的数据类型这个解释器暂且实现四种：`Number` 整型、`Float` 浮点、`String` 字符串以及 `Bool` 。再加上语法中的 `List` 和 `DottedList` 以及通用的标识符 `Atom` ，便可以定义一个有 7 个值（ constructor ）的类型 `LispVal`

```haskell
data LispVal = Atom String
             | List [LispVal]
             | DottedList [LispVal] LispVal
             | Number Integer
             | Float Double
             | String String
             | Bool Bool
```

当然，顺便让他们能够被 `show` 一下

```haskell
instance Show LispVal where
    show (Atom name) = name
    show (List contents) = "(" ++ unwordsList contents ++ ")"
    show (DottedList h t) = "(" ++ unwordsList h ++ " . " ++ show t ++ ")"
    show (String ctns) = "\"" ++ ctns ++ "\""
    show (Number num) = show num
    show (Float num) = show num
    show (Bool True) = "#t"
    show (Bool False) = "#f"

unwordsList :: [LispVal] -> String
unwordsList = unwords . map show

```

然后便是拼 parsec 的组合子，将语法对应解析到七个 constructor 上

```haskell
type LispParser = Parser LispVal

parseExpr :: LispParser
parseExpr = parseString <|> parseNumber <|> parseListSugar <|> parseAtom <|> do
    _ <- char '('
    x <- try parseList <|> parseDottedList
    _ <- char ')'
    return x
```

分别按照对应的语法规则实现 `parseString` `parseNumber` `parseAtom` `parseList` `parseDottedList` 即可（[参考](https://github.com/nameoverflow/LittleScheme/blob/master/src/Hscheme/Parser.hs)），其中 `parseListSugar` 用于解析 quote 之类的语法糖。

## 1. 执行、规约

如何将一个 S 表达式变成一个值？

最普通的情况，考虑一个 S 表达式表达函数调用。我们都知道 lambda calculus 的三条规则：

> α-conversion: changing bound variables (alpha);
> 
> β-reduction: applying functions to their arguments (beta);
> 
> η-conversion: which captures a notion of extensionality (eta).

实现执行 Lisp 语句，实际上就是实现 β 规约——代换求值的过程。将一个 List 看作为 `((λV1.λV2.λV3.(...).E) ...) E3') E2') E1')` ，作代换 `E [V:=E']` 直到得到一个不能再规约的 β 范式。

得到了 β 范式之后呢？在实际的编程中表达式总会有一个值。经过上述过程之后得到的 β 范式，在实际中可能有两种形式：只包含一个自由变量的表达式 M，或者一个 lambda 表达式 λV.E 。

显然，如果最后的结果是只包含一个自由变量的表达式，我们的程序运行结果就是这个自由变量所对应的值。那么这个值到底是多少呢？这就涉及到一个很常见的 `闭包` 概念。如果规约结果是个函数，类似的，程序的运行结果可能是一个函数，而在这里还有另外一种可能—— Lisp 是一种不支持默认的函数部分应用的语言，如果规约出的 β 范式结果是一个函数，也有可能是程序出错——参数数目不一致。

这里暂且不讨论如何得到最后的结果，先实现对一个 Lisp 表达式进行 β 规约的 `eval` 函数。

由于存在抛错（参数数目不匹配）的情况，首先需要定义一个异常类型（利用 `Control.Monad.Error` 包），并实现从 `Either` 类型中获取值

```haskell
-- import
import Control.Monad.Error

-- type
data LispError = NumArgs Integer [LispVal]

extractValue :: ThrowsError a -> a
extractValue (Right val) = val
extractValue _           = undefined

instance Show LispError where
    show (NumArgs expr found) = "Expected " ++ show expr ++ " args; found values " ++ unwordsList found

instance Error LispError where
    noMsg = Default "An error has occurred"
    strMsg = Default

type ThrowsError = Either LispError
```

目前为止，我们的 `eval` 过程只是对表达式进行规约，接受的参数类型为 `LispVal` ，返回值也是一个 `LispVal` ，当然也可能抛异常。

```haskell
eval :: LispVal -> ThrowsError LispVal
```

接着我们考虑如何表达一个函数。

从 λ 表达式和 Lisp 函数的形式来看，有两个要素——形参和函数体。我们可以用一个包含两个字段的 `record` 来表示。

```haskell
data LispFunc = LispFunc {
    paramsList :: [String],
    body :: [LispVal]
}
```

并且函数应该作为语言中的一个类型

```haskell
data LispVal = Atom String
             | List [LispVal]
             | DottedList [LispVal] LispVal
             | Number Integer
             | Float Double
             | String String
             | Bool Bool
             | Lambda LispFunc
```

然后 `show` 一下

```haskell
    show (Lambda LispFunc { paramsList = params }) =
      "(lambda (" ++ unwords (map show params) ++ ") ...)"
```

在 Scheme 中，表示一个函数的语法为 `(lambda (params) (body))` 。如果在执行过程中碰到这种形式的表达式，我们会把它规约为一个函数。用模式匹配很容易做到。

```haskell
eval (List (Atom "lambda" : List params : funcBody)) =
     makeFunction params funcBody

-- makeFunction
makeFunction :: [LispVal] -> [LispVal] -> LispVal
makeFunction args funcBody = Lambda $ LispFunc (map show args) funcBody
```

然后考虑通用的函数调用语句 `(function params...)`

{- 妈呀写的好累 -}

{- TODO -}
