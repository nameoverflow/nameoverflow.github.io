---
title: Is object-oriented programming both anti-modular and anti-parallel by its very nature? Why or why not?
date: 2015-03-06 23:00:45
tags:
  - object-oriented programming
---
(ref: [http://www.quora.com/Is-object-oriented-programming-both-anti-modular-and-anti-parallel-by-its-very-nature-Why-or-why-not](http://www.quora.com/Is-object-oriented-programming-both-anti-modular-and-anti-parallel-by-its-very-nature-Why-or-why-not))



>According to a blog post from Professor Robert Harper, the CMU Computer Science department is removing the required study of O-O from the Freshman curriculum for this reason. But why?



Jon Moter's answer:



While I can't climb into Professor Harper's brain, I figure what he's pointing to is that OOP is a programming paradigm that's based on state changes. You create a User object, you call methods on it, the state of the user changes, etc.



In contrast, (pure) functional programming has no side-effects, so calling a function doesn't cause any state change.  It just returns information based on the parameters passed in.



So if you assume the world is heading for a direction where computer code is written to run on massively parallel distributed systems, it's far easier to design code written in a functional manner, since there is no shared state to manage.  If you have hundreds or thousands of processors running on GPUs or in hardware clusters, you can distribute computations across all of them, leveraging a programming paradigm that's better suited for that.



I don't think procedural and OO programming is going to magically go away any time soon, but it's interesting that he's starting his students thinking in terms of functional and imperative programming, rather than procedural or OO.