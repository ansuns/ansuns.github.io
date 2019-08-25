---
title: Docker环境下jenkins结合gitlab自动化部署
abbrlink: 93441cba
date: 2018-11-16 11:55:56
tags:
  - 部署
  - PHP
  - mysql
  - nginx
  - lnmp
  - centos
  - 服务器
categories:
  - 服务
  - 部署
---
# Windows下使用docker toolbox 或 win10使用Dockerinstaller 安装docekr
- 安装 jenkins
docker run -p 8010:8080 -p 50000:50000 --name jenkins --volume /volume/jenkins:/var/jenkins_home jenkins/jenkins:lts

WIN10
docker run -p 8010:8080 -p 50000:50000 --name jenkins --volume f:/vol/jenkins:/var/jenkins_home jenkins/jenkins:lts

在浏览器端访问http://192.168.99.1:8010，看到如下界面就说明启动成功
{% asset_img j1.png  %}

查看输入密码，登录进入，会看到如下页面
{% asset_img j2.png  %}

选择安装推荐插件即可，等待安装完成
{% asset_img j3.png  %}

填写要创建的管理用户这里使用Admin 密码为12345678，点击保存并完成即可
{% asset_img j3.png  %}