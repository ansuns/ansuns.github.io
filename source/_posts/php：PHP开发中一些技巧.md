---
title: php：PHP开发中一些技巧
abbrlink: 6d5007a7
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