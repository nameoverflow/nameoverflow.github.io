---
title: 使用git部署网站
date: 2015-03-03 01:54:05
tags:
  - git
---
(Someone said a coder who's blog is full of "using some tools to do sth" is totally unreliable....)

大致方法是创建一个bare repository，然后利用hook脚本在每次push时更新网站内容。

##服务端

创建git用户和，禁用shell登陆

```
$ sudo useradd git && sudo groupadd git
#创建并修改密码
$ passwd git 
$ sudo vi /etc/passwd
将git一行设置为:/home/git:/usr/bin/git-shell
```

选一个喜欢的目录建立bare repository

```
$ cd <path>
$ mkdir website.git
$ sudo git init --bare website.git
#赋予给git用户
$ sudo chown -R git:git website.git
#配置hook
$ sudo vi website.git/hooks/post-receive

#!/bin/sh
GIT_WORK_TREE=path/to/websiteroot  git checkout -f

$ chmod +x post-receive
$ sudo chown -R git:git path/to/websiteroot
```

之后便获取本地公钥，设置authorized_keys，之后网站更新时只要git push到repository即可。