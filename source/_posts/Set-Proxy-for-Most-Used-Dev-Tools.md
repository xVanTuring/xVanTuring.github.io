---
title: Set Proxy for Most Used Dev Tools | 常用软件代理设置
date: 2019-05-03 01:51:26
tags: [Tips,Software]
---
## 1. git
``` bash
# config
$ git config --global https.proxy socks5://127.0.0.1:1080
$ git config --global http.proxy socks5://127.0.0.1:1080

# unset
git config --global --unset https.proxy
git config --global --unset http.proxy
```
<!-- More -->
## 2. npm & yarn
``` bash
# config
$ npm config set proxy socks5://127.0.0.1:1080
$ npm config set https-proxy socks5://127.0.0.1:1080

# unset
$ npm config delete proxy
$ npm config delete https-proxy

$ yarn config set proxy socks5://127.0.0.1:1080
$ yarn config set https-proxy socks5://127.0.0.1:1080
```