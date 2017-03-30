---
title: Fedora 22 新内核下 BCMA 43142网卡无WiFi的解决
date: 2015-08-26 09:36:19
tags:
---
先说解决办法：

```bash
sudo dnf install akmod-wl
```

Fedora的更新一向很快，带来的问题之一就是兼容性。

比如4.1.5的kernel下，并没有兼容bcma43142的内置driver。

第一反应是去下载内核编译，然并卵。

编译之后可以搜寻wifi，但是不能连接wpa2加密的wifi。

最后灵机一动想到了那个闭源驱动的rpm源，安装之，解决。

睡觉。
