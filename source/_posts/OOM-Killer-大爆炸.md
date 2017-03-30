---
title: OOM Killer 大爆炸
date: 2016-03-13 14:04:55
tags:
  - linux
---
讲道理把 OOM killer 设成 2 居然会把整个系统搞挂……

![enter image description here](https://dn-hcyue.qbox.me/img/QQ%E6%88%AA%E5%9B%BE20160313134747.png)

于是整个 OS 的内存管理全部挂掉，能用的命令只有 `echo` 

还好是可以以文件的形式强行修改服务进程的。

```bash
echo 0 > /proc/sys/vm/overcommit_memory
```

然后试了试 `ls` ，终于能够运行。

一激动之下多按了几个↑，然后……

![enter image description here](https://dn-hcyue.qbox.me/img/QQ%E6%88%AA%E5%9B%BE20160313135140.png)

……………………………………