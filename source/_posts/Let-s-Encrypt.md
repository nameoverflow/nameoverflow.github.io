---
title: Let's Encrypt!
date: 2016-05-14 13:40:54
tags:
---
> 世界因自由软件而美好。

昨天（5月13日）发现 SSL 证书到期了。想起之前谭总说过的 Let's Encrypt，决定体验一下。

相比起[半年前](https://tms.im/tms/letsencrypt-beta)，这玩意完善了很多、傻瓜了很多。

全程需要做的只需要 clone 一个[傻瓜工具包](https://github.com/certbot/certbot)，然后运行一行命令

```bash
./certbot-auto certonly --standalone --email admin@hcyue.me -d hcyue.me
```

之后坐等几秒，SSL 验证所需要的全部证书就都被放在了 `/etc/letsencrypt/live/hcyue.me/`

然后把 nginx 配置对应的项目修改为生成的 pem 即可。

![](https://dn-hcyue.qbox.me/img%2FS60514-133728.jpg)