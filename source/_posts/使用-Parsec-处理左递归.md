---
title: 使用 Parsec 处理左递归
date: 2017-03-03 20:41:36
tags:
  - haskell
  - parsec
  - compile
---


在给之前写的 Lisp 解释器之前套上表达式语法时，遇到这样几条文法




{% math %}
\begin{aligned}
Expr & \rightarrow Factor ... \\\\
Exprs & \rightarrow Expr  ,  Exprs\\\\
 Factor & \rightarrow Integer|Apply|Identify|...|{(}  {Expr} {)} \\\\
  Integer & \rightarrow... \\\\
...\\\\
 Apply & \rightarrow  Factor  (  Exprs  )
\end{aligned}
{% endmath %}

显然，non-terminal {% math %}Apply{% endmath %} 的派生最左端会进入  {% math %}Factor{% endmath %} ，之后又会回到 {% math %}Apply{% endmath %} 。教科书式的左递归。

<!--more-->

龙书中告诉我们，左递归可以通过提取出一个产生式来消除。



对于

{% math %}

A \rightarrow A \alpha_1|A \alpha_2|...|\beta..

{% endmath %}

在所有不以 A 开头的 A 的派生 {% math %} \beta {% endmath %} 后加入非终结符 {% math %} A' {% endmath %} ，然后 {% math %} A' {% endmath %} 派生出 {% math %} \alpha {% endmath %} ，即可消除左递归。

{% math %}
\begin{aligned}
A &\rightarrow \beta A' | ... \\\\
A' &\rightarrow \alpha_1 A'|\alpha_2 A' |...|\epsilon
\end{aligned}
{% endmath %}

道理都懂，但是怎么写代码呢？

`parsec` 的代码的一个特点是可以很直观的将文法和返回的结果对应。我们按照最开始的有左递归的文法可以很容易的写出这样的组合子

```haskell
parseFactor = spaces >> value >>= \v -> return v
    where parenExpr = between (char '(') (char ')') parseExpr
          value     = choice [ parseNumber, try parseFunCall, 
                               -- lots of parsers...
                               parenExpr ]

parseExpr = -- based on `Text.Parsec.Expr`

theComma = spaces >> char ','

parseApply :: Parser Expr
parseApply = do
    fun <- parseExpr
    spaces
    args <- between (char '(') (char ')') $ sepBy parseExpr theComma
    return $ Apply fun args
```

原文法中的每一个非终结符都可以对应到相应的 `parser` ，相应的 `parser` 进行适当的处理返回 AST 的节点。一切都那么尽然有序，除了它会陷入无穷的递归然后电脑被不会爆栈的 Haskell 撑爆内存。

然而进行消除左递归后的文法中，{% math %} A' {% endmath %} 的意义是不明的，不能对应到 AST 中的某个节点；甚至 A 本身意义也是不明的。这种情况下，我们就需要进行一些特殊的间接处理让 `parser` 能返回正确的结果。

把原来的文法的左递归消除。

{% math %}
\begin{aligned}
Factor & \rightarrow Integer| Identify|...|{(}  {Expr} {)} \\\\
Factor' & \rightarrow Factor   Apply'\\\\
Apply' & \rightarrow   (  Exprs  )  Apply' | \epsilon \\\\
\end{aligned}
{% endmath %}

第一条产生式很好表达，和之前只是删掉了直接派生 {% math %}  Apply {% endmath %} 

```haskell
parseFactor = spaces >> noLeft <|> parenExpr
  where split = many splitter
        parenExpr = between (char '(') (char ')') parseExpr
        noLeft = choice [ parseString, parseNumber, parenExpr,
                          -- lots of parsers...
        				]
```

接下来需要处理第二条和第三条。显然，这两条规则对应的 AST 节点有两种，普通的 `Factor` 或 `Apply` 。我们可以把这两条规则视为一个整体的过程：在 `parse` 出一个 `Factor` 之后，尝试 `char '('` ，满足则继续，不满足则直接返回之前的 `Factor` 。

```haskell
parseFactorOrApply = do
    factor <- parseFactor
    rest fun
    where rest x = (do args <- between (char '(') (char ')') $ sepBy parseExpr theComma
                           rest pos' . Expr pos $ Apply x args) <|> return x
```

成功完成任务。

`parsec` 自带的组合子中的 `chainr1` 也是使用的类似方法处理左递归，它的功能是构造右结合的双目操作符的表达式的解析。

```haskell
chainr1 :: (Stream s m t) => ParsecT s u m a -> ParsecT s u m (a -> a -> a) -> ParsecT s u m a
chainr1 p op        = scan
                    where
                      scan      = do{ x <- p; rest x }
                      rest x    = do{ f <- op
                                    ; y <- scan
                                    ; return (f x y)
                                    }
                                <|> return x
```



