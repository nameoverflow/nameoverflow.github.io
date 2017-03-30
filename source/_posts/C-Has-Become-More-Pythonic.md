---
title: C++ Has Become More Pythonic
date: 2015-01-18 21:20:53
tags:
  - become
  - c/cpp
  - python
---

原文：[http://preshing.com/20141202/cpp-has-become-more-pythonic/](http://preshing.com/20141202/cpp-has-become-more-pythonic/)

![](http://preshing.com/images/pythonic_cpp.png)

&nbsp;

C++ has changed a lot in recent years. The last two revisions, C++11 and C++14, introduce so many new features that, in the words of Bjarne Stroustrup, “It feels like a new language.”

It’s true. Modern C++ lends itself to a whole new style of programming – and I couldn’t help noticing it has more of a Python flavor. Ranged-based for loops, type deduction, vector and map initializers, lambda expressions. The more you explore modern C++, the more you find Python’s fingerprints all over it.
<!--more-->

Was Python a direct influence on modern C++? Or did Python simply adopt a few useful constructs before C++ got around to it? You be the judge.

&nbsp;

# Literals

&nbsp;

Python introduced binary literals in 2008\. Now C++14 has them. [Update: Thiago Macieira points out in the comments that GCC actually supported them back in 2007.]

static const int primes = 0b10100000100010100010100010101100;
Python also introduced raw string literals back in 1998\. They’re convenient when hardcoding a regular expression or a Windows path. C++11 added the same idea with a slightly different syntax:
    const char* path = R"(c:thisstringhasbackslashes)";

Range-Based For Loops

In Python, a for loop always iterates over a Python object:

```python
for x in myList:
    print(x)
```

Meanwhile, for nearly three decades, C++ supported only C-style for loops. Finally, in C++11, range-based for loops were added:

```cpp
for (int x : myList)
    std::cout << x;
```


You can iterate over a std::vector or any class which implements the begin and end member functions – not unlike Python’s iterator protocol. With range-based for loops, I often find myself wishing C++ had Python’s xrange function built-in.

&nbsp;

**Auto**

&nbsp;

Python has always been a dynamically typed language. You don’t need to declare variable types anywhere, since types are a property of the objects themselves.

```python
x = "Hello world!"
print(x)
```

C++, on the other hand, is not dynamically typed. It’s statically typed. But since C++11 repurposed the auto keyword for type deduction, you can write code that looks a lot like dynamic typing:

```cpp
auto x = "Hello world!";
std::cout << x;
```

When you call functions that are overloaded for several types, such as std::ostream::operator<< or a template function, C++ resembles a dynamically typed language even more. C++14 further fleshes out support for the auto keyword, adding support for auto return values and auto arguments to lambda functions.

&nbsp;

**Tuples**

&nbsp;

Python has had tuples pretty much since the beginning. They’re nice when you need to package several values together, but don’t feel like naming a class.

```python
triple = (5, 6, 7)
print(triple[0])
```


C++ added tuples to the standard library in C++11\. The proposal even mentions Python as an inspiration:

```cpp
auto triple = std::make_tuple(5, 6, 7);
std::cout << std::get(triple);
```

Python lets you unpack a tuple into separate variables:

```python
x, y, z = triple
```

You can do the same thing in C++ using std::tie:

```cpp
std::tie(x, y, z) = triple;
```


&nbsp;

**Uniform Initialization**

&nbsp;

In Python, lists are a built-in type. As such, you can create a Python list using a single expression:

```python
myList = [6, 3, 7, 8]
myList.append(5);
```

C++’s std::vector is the closest analog to a Python list. Uniform initialization, new in C++11, now lets us create them using a single expression as well:

```cpp
auto myList = std::vector{ 6, 3, 7, 8 };
myList.push_back(5);
```

In Python, you can also create a dictionary with a single expression:

```python
myDict = {5: "foo", 6: "bar"}
print(myDict[5])
```

Similarly, uniform initialization also works on C++’s std::map and unordered_map:

```cpp
auto myDict = std::unordered_map < int, const char* > { { 5, "foo" }, { 6, "bar" } };
std::cout << myDict[5];
```

&nbsp;

**Lambda Expressions**

&nbsp;

Python has supported lambda functions since 1994:

```python
myList.sort(key = lambda x: abs(x))
```

Lambda expressions were added in C++11:

```cpp
std::sort(myList.begin(), myList.end(), [](int x, int y){ return std::abs(x) &lt; std::abs(y); });
```

In 2001, Python added statically nested scopes, which allow lambda functions to capture variables defined in enclosing functions:

```python
def adder(amount):
    return lambda x: x + amount
...
print(adder(5)(5))
```

Likewise, C++ lambda expressions support a flexible set of capture rules, allowing you to do similar things:

```cpp
auto adder(int amount) {
    return [=](int x){ return x + amount; };
}
...
std::cout << adder(5)(5);
```

&nbsp;

**Standard Algorithms**

&nbsp;

Python’s built-in filter function lets you selectively copy elements from a list (though list comprehensions are preferred):

```python
result = filter(lambda x: x &gt;= 0, myList)
```

C++11 introduces std::copy_if, which lets us use a similar, almost-functional style:

```cpp
auto result = std::vector{};
std::copy_if(myList.begin(), myList.end(), std::back_inserter(result), [](int x){ return x &gt;= 0; });
```

Other C++ algorithms that mimic Python built-ins include transform, any_of, all_of, min and max. The upcoming ranges proposal has the potential to simplify such expressions further.

&nbsp;

**Parameter Packs**

&nbsp;

Python began supporting arbitrary argument lists in 1998\. You can define a function taking a variable number of arguments, exposed as a tuple, and expand a tuple when passing arguments to another function:

```python
def foo(*args):
    return tuple(*args)
...
triple = foo(5, 6, 7)
```

C++11 adds support for parameter packs. Unlike C-style variable arguments, but like Python’s arbitrary argument lists, the parameter pack has a name which represents the entire sequence of arguments. One important difference: C++ parameter packs are not exposed as a single object at runtime. You can only manipulate them through template metaprogramming at compile time.

```cpp
template  auto foo(T&amp;&amp;... args) {
    return std::make_tuple(args...);
}
...
auto triple = foo(5, 6, 7);
```

Not all of the new C++11 and C++14 features mimic Python functionality, but it seems a lot of them do. Python is recognized as a friendly, approachable programming language. Perhaps some of its charisma has rubbed off?

What do you think? Do the new features succeed in making C++ simpler, more approachable or more expressive?