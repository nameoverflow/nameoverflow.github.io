---
title: 〔吐槽向〕Windows 输入法的 metro 应用兼容性改造
date: 2018-01-13 21:51:55
tags:
  - input method
  - windows
  - named pipe
  - interprocess
---

## TL;DR

微软真是坑爹不允许在 UWP 里用共享内存结果小狼毫就挂了，改成用 Windows 的 `named pipe` 来做跨进程交互后就可以用了。

## 缘由

重新捡起五笔后，一直苦恼没有一个合用的五笔输入法。身为半个初学者，很看中的一个功能就是五笔反查——总不能每个不会的字都专门打开一个反查工具；而 Windows 自带的五笔只有一个并不好用的五笔拼音混输功能。各种老牌的五笔输入法要么面界太丑要么久不更新，更有甚者会摆出一幅流氓作派。

几年之前试水过 RIME 输入法，知道它是一个很强大的输入法，可以满足各种定制需求。但是当初的我并没有开发能力，RIME 的定制又稍显复杂，于是并没有继续下去。现在有了需求后回头看，发现 RIME 无疑是最适合现在的我的。

但是现在要在 Windows 下使用 RIME 却有了个当初没有的新问题——兼容性。作为第一批 Windows 10 用户，无疑需要一个能够在 UWP 应用下正常使用的输入法。但是 RIME 最初的 Windows 前端「小狼毫」却对从 Windows 8 开始的 Metro 应用——包括 Windows 10 的 UWP 都不兼容。原作者也因为无暇分身而[放弃了「小狼毫」的维护](https://github.com/rime/home/issues/25)；其它开发者实现的其它前端又存在各种不稳定的问题——比如 RIME 吧的用户将 RIME 移植到了一个叫作 [PIME](https://github.com/EasyIME/PIME) 的输入法框架下，这个框架的界面实现非常的搓，除此之外它的输入服务实现也很蛋疼，经常崩溃，也没有可靠的异常处理，很多时候需要手动重启输入服务。

开源界的一大准则就是「你行你上」和「show me the code」。当年使用小狼毫的时候没有经历过什么异常崩溃，说明它在这方面的设计是十分优秀的；现在的主要问题也就是不兼容 Metro 应用而已。既然现在自己有了开发能力，不如自已维护一下，方便自己，也方便他人。

<!--more-->

## Text Services Framework

说是决定自己维护，一开始心里也是没什么底的，毕竟自己没有接触过 Windows 开发那一套，也不知道「小狼毫」的兼容性问题究竟出在哪。

把源码 clone 下来后，一边翻 MSDN 一边浏览代码，基本摸清了 Windows 下输入法的原理。

现在的 Windows 下的输入法一般有两种实现方式—— Input Method Editor 和 Text Services Framework，都是 Windows 提供给输入法应用的接口。前者算是传统方法，功能强大，但是在 Metro 应用中禁用。后者算是现在的「正统」方法，在现在的 Windows 系统中通用。（什么，Windows XP？那是什么？）

小狼毫输入法在这方面并没有什么毛病——它同时实现了两种方法（膜一下公子）。不过既然我是想要兼容 Metro，那么就懒得管 IME 实现了。

所谓 TSF（Text Services Framework），说白了就是 Windows 以 COM 形式（又是微软特色）提供的一套 interface，实现了 interface 要求的 methods 后就可以实现一般意义上的输入法的各种功能。

以 TSF 方式实现的输入法其实就是一个 DLL。Windows 在应用需要用户输入的时候加载注册过的输入法对应的 DLL，然后调用 DLL 中导出的接口的 method。输入法应用在这些 method 中对用户输入的事件进行处理后喂给系统一个叫作 composition 的东西，里面包括了对用户输入行为的响应（如要在输入框里显示什么东西）；系统拿到 composition 后进行相应处理返馈给用户。这样看来，这个东西设计的相是挺优雅的。

## Windows Runtime

了解了 TSF 的工作机制后，小狼毫问题出在哪就很明显了。

TSF 输入法是作为 DLL 注入进程中的，那么这个进程如果受了什么限制，输入法也会同样受限。而从 Windows 8 开始引入的 Windows Store App，或者叫作 Metro 应用，或者在 Windows 10 里叫 UWP，它们的进程统统是运行在一个叫 Windows Runtime 的特殊运行时里的。由于微软对 Metro APP 的定位（安全性和受限），在 Windows Runtime 里有一大堆事情不能干。小狼毫肯定是在 TSF 中试图调用 Windows Runtime 不允许的 API 导致了崩溃。

用 VS 注入到 Edge 进程里单步一下，不出所料找到了小狼毫 DLL 崩溃之源。

## 小狼毫输入服务

小狼毫输入法的结构分为「前」「后」两部分。

「前」，就是指的 TSF 或者 IME 实现部分，用于接受用户在应用中的输入，以及将输入法的处理结果反馈给系统。

「后」，也是核心部分，是与 RIME 的对接。「前」部分接受输入后将需要处理的用户输入发送给「后」部分，「后」部分调用 RIME 进行相应的处理。输入法的 UI，也就是选字框，小狼毫也是放在了这一部分（虽然 TSF 有专门的「候选词」接口可以用来做这事）。

「前」部分，对于 TSF 来说，就是那个 DLL。而「后」部分，小狼毫将其作为了一个独立的进程「WeaselServer.exe」运行。

「前」和「后」之间交互的这个过程就是问题所在。小狼毫使用 `window message` 和 `shared memory` 来实现前后的跨进程交互——然而 Windows Runtime 里这两个哪个者不许用。跟踪小狼毫的 DLL 执行就可以发现，DLL 在初始化输入法时，试图打开共享内存来获取后端进程的运行状态，然后被系统无情拒绝了，输入法很伤心，只能原地静默崩溃。

## Named Pipe

知道了问题出在哪就好解决了。只要换一个能在 Windows RT 里用的跨进程交互方式就行……了吗？

然而 MSDN 告诉我：吔屎啦，我们就是不想让 Windows RT 能做 IPC。

然而办法肯定是有的，因为主流的输入法，包括 Windows 自带的，很多都是前后端进程分离的模式，他们也能在 UWP 里正常使用。

`Named pipe` 是经典的跨进程通信方式 `pipe` 的加强版，相当于 UNIX 下的 `Unix domain socket`。在应用中以文件读写的形式调用。Windows Runtime 并没有完全禁止文件读写，前提是这个文件的权限设置允许。

关于 [`named pipe`](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365590.aspx) 及其[权限设置](https://msdn.microsoft.com/en-us/library/windows/desktop/aa379560.aspx)在 MSDN 上有详细的说明。

几乎[重写了一半的 IPC 部分代码](https://github.com/nameoverflow/weasel/commit/c045b2c0e23e82700541973b25e6ec1f8e9f65cb)后，终于让小狼毫能在 Windows 10 下无痛使用了……

