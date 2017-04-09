---
title: The Mind Language
date: 2016-03-18 13:31:50
tags:
  - computer science
---

最近开的一个坑，预想中是一门 Lisp 方言。

另外，手撸状态机真是天坑，一辈子没写过这么多 `switch`

## Example Code

```text
# Definition of cons, car, cdr

def cons fst, snd =
    sel -> sel fst, snd

def car pair =
    pair (fst, snd) -> fst

def cdr pair =
    pair (fst, snd) -> snd

# Definition of boolean

def true x, y =
    x
def false x, y =
    y

def fakeIf condi, fst, snd =
    condi fst, snd


# Codes above are not the real definitions
# In fact they are implemented in interpreter

def fib n =
    if n == 0 || n == 1,
        n,
        fib (n - 1) + fib (n - 2)

def fizzBuss =
    def buildList start, end =
        if start == end then end
        else cons start, (buildList (start + 1), end)
    def helper n =
        if n % 3 == 0 && n % 5 == 0 then "Fizz Buzz"
        else if n % 3 == 0 then "Fizz"
        else if n % 5 == 0 then "Buzz"
        else n
    print map (buildList 1, 100), helper


```

## Grammar

```text
<MindProgram> := <Declarations>

<Declarations> := <Definition> [ "\n+" <Declarations> ]

<Definition> := "def" "(" <ValueDefinition> | <FunctionDefinition> )

<ValueDefinition> := <DefinitionHead> <DefinitionTail>

<FunctionDefinition> := <DefinitionHead> <ParamList> <DefinitionTail>

<DefinitionHead> := <Identifier>

<DefinitionTail> := <DeclarOperator> [ <Declarations> "\n"]  [ <LetBlock> ] <Expression>

<ParamList> := <ParamDeclaration> ["," <ParamList> ]

<ParamDeclaration> := <Identifier> 

<LetBlock> := "let" <Declarations> "in"

<LetDeclarations> := <Assignment> ["\n" <LetDeclarations>]

<Assignment> := <ValueDefinition> | <FunctionDefinition>

<Expression> := <Value> | <Literal> | <FunctionCall> | <Expression> <Operator> <Expression> 

<Value> := <Identifier>

<Literal> := <StringLiteral> | <CharLiteral> | <NumberLiteral> | <LambdaFunction> | <ListLiteral>

<StringLiteral> := ""* [^"]* ""

<CharLiteral> := "' [^'] '"

<NumberLiteral> := "[0-9]+("."[0-9]+)?"

<LambdaFunction> := <Identifier> "->" <Expression>

<ListLiteral> := "[" <ListElements> "]"

<ListElements> := <ListElement> ["," <ListElement>]

<ListElement> := <Identifier> | <Literal>

<Identifier> := "[_a-zA-Z][_0-9a-zA-z]"*
```
