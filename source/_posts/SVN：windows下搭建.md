---
title: windows下SVN服务搭建
abbrlink: e7812f0a
date: 2015-12-25 00:44:22
tags:
  - svn
categories:
  - 后端
  - 版本控制
---
#### 1. 基础 #### 
##### 1.1. 安装服务端，安装客户端（小乌龟。。。） ##### 
##### 1.2. 建立仓库：svnadmin create f:\web\shop ##### 
##### 1.3. 启动仓库 svnserve -d -r f:\web\shop ##### 
##### 1.4. shop\conf\svnserve.conf 约 12行 ##### 
#anon-access = read 修改为 anon-access = write （前面不能含空格）意思是允许匿名提交 ##### 
##### 1.5.checkout... ##### 
 
#### 2.设置账户密码 #### 
#### 2.1 引进配置文件**shop\conf\svnserve.conf**修改 #### 
#### 约20行 **password-db = passwd** 用户账户配置 #### 
#### 约27行 **authz-db = authz** 权限配置 #### 
 
#### 2.2 添加用户->修改 #### 
/conf/passwd
#添加用户
```bash
[users]
# harry = harryssecret
# sally = sallyssecret
a1 = 123
a2 = 123
```
 
 
 
##### 2.3. 设置权限 ##### 
修改->**conf/authz** 
```bash
# 给svn仓库的用户设置权限
#[/] 仓库的主机名地址 svn://localhost
#   运行仓库服务： svnserve -d -r f:/web/shop
 
#[shop:/] 多个。。仓库的主机名地址 svn://localhost/shop
#   运行仓库服务： svnserve -d -r f:/web/
#具体权限 r：read 读 w：write 写
[shop:/]
a1 = rw
a2 = r
* = 
```
 
 
##### 2.4. svnserve -d -r f:\web ##### 
>这里启动多个仓库，所以web，而不是web\shop
 
##### 2.5. commit or checkout ，就要输入用户名和密码了额 ##### 
 
##### 2.6. 多个用户时进行组别设置 ##### 
 
约21-24行
``bash
myusers = a1,a2,a3,a4,a5
``
 
#@符表示组，而不是普通用户例如 a1
#给组myusers 读写权限
@myusers = rw
 

日志不正常。
>关掉匿名用户：shop\conf\svnserve.conf 约 12行
``bash
anon-access = none
``
 
然后就正常了。
 
#### 3.设置某个目录权限，只允许访问该目录。。 #### 
#设置某个目录权限
修改authz
```bash
[shop:/waibao]
w1 = rw
w2 = rw
w3 = rw
* = 
```

保存后在用a1提交：

用w1提交：

 
#### 4. 设置SVN开机启动 #### 
```bash
sc create svnd binPath= "C:/Program Files (x86)/Subversion/bin/svnserve.exe -r f:/web/ --service" start= auto 
binPath=空格 start=空格
```
 
删除服务 **sc delete svnd**
 
 
 
 
 