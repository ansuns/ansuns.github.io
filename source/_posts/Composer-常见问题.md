---
title: Composer-常见问题
abbrlink: 59db2d48
date: 2019-07-19 13:58:53
tags:
 - 后端
 - Composer
categories:
 - 技术
---
1. git-bash下composer命令无法使用的问题
已经安装composer，写好composer.bat，并且设置好了path，在cmd下可以正常使用，但是在git-bash里面不行，显示如下提示：
``` bash```
bash: composer: command not found
```
原因很可能是composer文件没有可执行权限，git-bash是以linux shell方式运行的，linux和windows文件权限管理方式不太一样。
切换到composer文件所在目录，执行如下命令修复权限：
``` bash
chmod 755 composer.bat
```
可是我发现上面的命令没有效果，这就尴尬了……
其实真正的原因是，git-bash 不识别 composer.bat 文件，需要新建一个 composer 文件，输入如下内容：
```angular2html
#!/usr/bin/env sh
# php /path/to/composer.phar $*
php `dirname $0`/composer.phar $*
```
#!/usr/bin/env sh 有个这一行，git-bash 会自动识别为可执行文件，不需要也不能使用 chmod 命令修改权限。

2. Installation failed, reverting ./composer.json to its original content.
解决
``` bash
composer update nothing
```
