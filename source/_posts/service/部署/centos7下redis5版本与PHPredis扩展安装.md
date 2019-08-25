---
title: centos7下redis5编译安装
abbrlink: 977b1b01
date: 2019-07-01 23:50:05
tags:
  - 部署
  - PHP
  - centos
  - 服务器
categories:
  - 服务
  - 部署
---
### 下载 redis 5.0.2
```bash
wget -c http://download.redis.io/releases/redis-5.0.2.tar.gz
```
### 解压到目录
```bash
tar -zxvf redis-5.0.2.tar.gz -C /usr/local/
cd /usr/local/redis-5.0.2/
```

### 编译
```bash
make
```
### 安装到指定位置
```bash
make  PREFIX=/usr/local/redis-5.0.2 install
```
### 编辑配置文件
```bash
vim /usr/local/redis-5.0.2/redis.conf
```

在配置文件里面修改一下项目:
```text
# 修改一下配置
# redis以守护进程的方式运行
# no表示不以守护进程的方式运行(会占用一个终端)  
daemonize yes

# 客户端闲置多长时间后断开连接，默认为0关闭此功能                                      
timeout 300

# 设置redis日志级别，默认级别：notice                     
loglevel verbose

# 设置日志文件的输出方式,如果以守护进程的方式运行redis 默认:""  如果要写入文件可以直接指定文件路径。
# 并且日志输出设置为stdout,那么日志信息就输出到/dev/null里面去了 
logfile stdout
# 设置密码授权
requirepass <设置密码>
# 监听ip
bind 127.0.0.1 
#指定监听端口 默认为6379
port 6379
#持久化数据存放目录 默认为./表示当前目录
dir /data/redis
#RBD持久化数据文件 保持默认
dbfilename dump.rdb
```

### 添加系统服务与开机启动:
```bash
vim /usr/lib/systemd/system/redis.service
```
内容如下:
```text
[Unit]
Description=Redis
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/redis_6379.pid
ExecStart=/usr/local/redis-5.0.2/bin/redis-server /usr/local/redis-5.0.2/redis.conf
ExecReload=/bin/kill -s HUP $MAINPID 
ExecStop=/bin/kill -s QUIT $MAINPID 
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
保存退出

添加环境变量:
```bash
vim /etc/profile
```
添加内容如下:
```bash
export PATH=$PATH:/usr/local/redis-5.0.2/bin
```
使环境变量生效:
```bash
source /etc/profile
```
重载服务配置:
```bash
systemctl daemon-reload
```
启动redis:
````bash
systemctl start redis
````
添加开机启动:
```bash
systemctl enable redis
```
第六步测试redis数据库
链接redis数据库:
```bash
redis-cli
```
验证密码:
```bash
AUTH <password>
```
<password>为你自己设置的密码
写入数据:
```bash
set name 888
```
查看数据:
```bash
get name
``` 
删除数据:
```bash
DEL name 
```

### 安装PHP扩展
>下载扩展包
```bash
wget -c https://pecl.php.net/get/redis-5.0.2.tgz
```

```bash
tar xzf redis-5.0.2.tgz # 解压
``` 
### 进入文件夹并编译php扩展
```bash
cd redis-5.0.2/
phpize
```
### 配置环境
```bash
./configure --with-php-config=/usr/local/php/bin/php-config
```
> 这里的php-config路径 根据自己实际情况而定

### 编译安装
```bash
make && make install
```
```text
#出现#Installing shared extensions:/usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/#即为安装成功。
#打开php.in文件，并添加 extension=redis.so,
保存退出即可
vim /usr/local/php/etc/php.ini 
#文件末尾添加 extension=redis.so  #保存退出，
```
重启php-fpm
```bash
systemctl restart php-fpm
```
### 安装完成，查看phpInfo()：
{% asset_img 1.png  %}
