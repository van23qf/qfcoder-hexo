---
title: 最新的Let's Encrypt免费https证书配置
date: 2023-05-19 13:54:57
tags: ["https", "Let's Encrypt"]
categories: ["涨姿势"]
---

本次流程基于Ubuntu，首先安装certbot
```shell
apt install certbot
```
使用 openssl 工具生成 dhparams
```shell
openssl dhparam -out /etc/ssl/certs/dhparams.pem 2048
```
生成免费证书
```shell
certbot certonly --webroot --agree-tos -v -t --email xxx@xxx.com -w /path/to/your/web/root -d www.xxx.com
```
如果执行正常，将会显示生成的证书文件目录
配置nginx如下
```shell
server
{
    listen       80;
    server_name xxx.com;
    return 301 https://$server_name$request_uri;
}
server
{
    listen 443 ssl http2;

    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;
    ssl_session_timeout 5m;
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5";
    ssl_session_cache builtin:1000 shared:SSL:10m;
    # openssl dhparam -out /path/to/dhparams.pem 2048
    ssl_dhparam /path/to/dhparams.pem;

    server_name xxx.com;
    index index.html index.htm index.php default.html default.htm default.php;
    root  /path/to/wwwroot/web;
}
```

配置定时任务每个月自动续期(每隔七天临晨1点执行一次)
```shell
0 1 */7 * * certbot renew --renew-hook "systemctl restart nginx"
```



