---
title: PHP开发中一些技巧
abbrlink: 1ad5be0d
date: 2015-08-27 17:56:44
tags:
  - PHP
  - 后端
categories:
  - 技术
  - PHP
---
1. 让json_encode不转义斜杠

``` php
echo str_replace("\\/", "/", json_encode("2013/4/21"));
```
若是php版本是5.4的话
``` php
echo json_encode("2011/7/11", JSON_UNESCAPED_SLASHES);
```
2. openssl生成证书server.key server.crt
Key是私用秘钥，通常是RSA算法
Csr是证书请求文件，用于申请证书。在制作csr文件时，必须使用自己的私钥来签署申，还可以设定一个密钥。
crt是CA认证后的证书文，签署人用自己的key给你签署凭证。
key的生成
``` bash
openssl genrsa -out server.key 2048
```
这样是生成RSA密钥，openssl格式，2048位强度。server.key是密钥文件名。
csr的生成
``` bash
openssl req -new -key server.key -out server.csr
```
需要依次输入国家，地区，组织，email。最重要的是有一个common name，可以写你的名字或者域名。如果为了https申请，这个
必须和域名吻合，否则会引发浏览器警报。生成的csr文件讲给CA签名后形成服务端自己的证书。
crt的生成
``` bash
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
生成的server.crt文件
3. 获取程序开始执行的时间
``` php
$stime = microtime(true); #获取程序开始执行的时间
#你写的php代码
$etime = microtime(true); #获取程序执行结束的时间
$total = $etime - $stime;   #计算差值
echo "<br />{$total} times";
```
