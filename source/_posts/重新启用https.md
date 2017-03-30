---
title: 重新启用https
date: 2015-10-16 13:58:10
tags:
  - server
---
之前因为误操作把老版站点的服务器给挂了【手残把iptables drop all了orz】，新版博客弄到了另一台服务器，于是便没有开启https。

今天把证书弄了回来，于是重新开启了https访问。

然而个人小站开https存粹装逼罢了= =

开启https需要把证书放在指定的位置，然后配置nginx开启ssl

```nginx
server {
  listen 80;
  server_name hcyue.me www.hcyue.me;
  rewrite ^ https://$server_name$request_uri? permanent;
}
server {
    listen 443 ssl http2;
    server_name www.hcyue.me hcyue.me;
    include none.conf;
    location / {
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        proxy_set_header Host $http_host;  
        proxy_set_header X-NginX-Proxy true;  
        proxy_pass http://127.0.0.1:4000/;  
        proxy_redirect off;  
    }
    location /static/ {
        alias [打码];
        expires 30d;
        autoindex on;
    }
    access_log  [打码]  access;
    ssl on;
    ssl_certificate [打码]/hcyue_me.crt;
    ssl_certificate_key [打码]/hcyue_me.key;
}
```

然后打开iptables的443端口

```bash
iptables -A INPUT -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
```

重启nginx，小绿锁就回来咯