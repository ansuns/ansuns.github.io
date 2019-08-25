---
title: centos7部署lnmp环境
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
abbrlink: 2293fa6a
date: 2019-07-13 20:05:07
---
#1. 准备编译环境

- 关闭防火墙
运行
`systemctl status firewalld`命令查看当前防火墙的状态。
{% asset_img 1.png  %}
i. 
- 如果防火墙的状态参数是inactive，则防火墙为关闭状态。
- 如果防火墙的状态参数是active，则防火墙为开启状态。本示例中防火墙为开启状态，因此需要关闭防火墙。
ii. 关闭防火墙。如果防火墙为关闭状态可以忽略此步骤。
- 如果您想临时关闭防火墙，运行命令`systemctl stop firewalld`。
>说明 这只是暂时关闭防火墙，下次重启Linux后，防火墙还会开启。

- 如果您想永久关闭防火墙，运行命令`systemctl disable firewalld`。
>说明 如果您想重新开启防火墙，请参见**firewalld**官网信息。

关闭SELinux。
运行`getenforce`命令查看SELinux的当前状态。
{% asset_img 2.png  %}

ii. 
- 如果SELinux状态参数是Disabled， 则SELinux为关闭状态。
- 如果SELinux状态参数是Enforcing，则SELinux为开启状态。本示例中SELinux为开启状态，因此需要关闭SELinux。

iii. 关闭SELinux。如果SELinux为关闭状态可以忽略此步骤。
- 如果您想临时关闭SELinux，运行命令setenforce 0。

> 说明 这只是暂时关闭SELinux，下次重启Linux后，SELinux还会开启。

- 如果您想永久关闭SELinux，运行命令`vi /etc/selinux/config`编辑SELinux配置文件。回车后，把光标移动到SELINUX=enforcing这
一行，按i键进入编辑模式，修改为`SELINUX=disabled`， 按Esc键，然后输入:wq并按Enter键以保存并关闭SELinux配置文件。
> 说明 如果您想重新开启SELinux，请参见SELinux的官方文档。

- 查看关闭状态
{% asset_img 3.png  %}

- 重启系统使设置生效。


- 下载软件：
```bash
wget -c https://www.php.net/distributions/php-7.3.8.tar.gz
wget -c http://nginx.org/download/nginx-1.16.1.tar.gz
wget -c https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz
wget -c https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-5.7/mysql-5.7.25.tar.gz
wget -c http://mirrors.163.com/mysql/Downloads/MySQL-5.7/mysql-boost-5.7.27.tar.gz
```

#2. 安装Nginx
安装编译环境
```bash
yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

```bash
groupadd www # 添加用户组
```

```bash
useradd www -g www -s /sbin/nologin -M #-M 不要自动建立用户的登入目录 -g指定用户组
```

```bash
tar -zxvf pcre.gz
```

```bash 
mv pcre-8.42 /usr/src/pcre-8.42
```

```bash
cd /usr/lcaol/src/nginx-1.16.1
```

```bash
./configure \
--prefix=/usr/local/nginx \
--sbin-path=/usr/local/nginx/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--user=www \
--group=www \
--with-http_ssl_module \
```
>--with-pcre=/usr/src/pcre-8.42 # 不手动指定

编译安装
```bash
make && make install
```

启动
```bash
/usr/local/nginx/nginx
```
{% asset_img 4.png  %}

>参考资料
>- [centos7.2源码编译安装LNMP](https://www.bfshu.com/jcaz/123)
>- [centos 7.x编写开机启动服务](http://www.dohooe.com/2016/03/03/352.html)
>- [CentOS7设置nginx开机自启动](https://www.jianshu.com/p/ca5ee5f7075c)

1、在系统服务目录里创建nginx.service文件
```bash
vim /usr/lib/systemd/system/nginx.service
```
2、写入内容如下：

```bash
#nginx服务配置到该文件中
#服务描述性的配置
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network.target remote-fs.target nss-lookup.target
#服务关键配置
[Service]
Type=forking
#pid文件位置 
#要与nginx配置文件中的pid配置路径一致，这个很重要，否则会服务启动失败
PIDFile=/usr/local/nginx/nginx.pid
#启动前检测 nginx配置文件 是否正确
ExecStartPre=/usr/local/nginx/nginx -t -c /usr/local/nginx/nginx.conf
#启动
ExecStart=/usr/local/nginx/nginx -c /usr/local/nginx/nginx.conf
#重启
ExecReload=/bin/kill -s HUP $MAINPID
#关闭
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```
保存退出。


[Unit]:服务的说明
- **Description**：描述服务
**After**描述服务类别
- [Service]服务运行参数的设置
- **Type=forking**：是后台运行的形式
- **ExecStart**：为服务的具体运行命令
- **ExecReload**：为重启命令
- **ExecStop**：为停止命令
- **PrivateTmp=True**：表示给服务分配独立的临时空间
- 注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
[Install]运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3


2.设置开机启动
-	`systemctl enable nginx.service`

3.其它命令
-	`systemctl start nginx.service` 　#启动nginx服务
-	`systemctl enable nginx.service`　#设置开机自启动
-	`systemctl disable nginx.service`　#停止开机自启动
-	`systemctl status nginx.service`　#查看服务当前状态
-	`systemctl restart nginx.service`　#重新启动服务
-	systemctl list-units --type=service`　#查看所有已启动的服务


停止nginx
```bash
/usr/local/nginx/nginx -s stop
```

创建软链接
```bash
ln -s /usr/local/nginx/nginx /usr/bin/nginx
```



#3. MySQL安装
1、检查是否安装mariadb如安装则卸载
```bash
yum list installed | grep mariadb 
```
{% asset_img 5.png  %}

>删除
```bash
yum -y remove mariadb* boost-*
```
{% asset_img 6.png  %}

```bash
cd /usr/local/src/mysql-5.7.27/
```

2、安装依赖包
```bash
yum install -y cmake make gcc gcc-c++ bison ncurses ncurses-devel
```

新建MySQL用户和用户组
```bash
groupadd -r mysql && useradd -r -g mysql -s /sbin/nologin -M mysql 
```

创建目录
```bash
[root@centosGome ~]# mkdir -vp /usr/mysql/{data,log,temp} #或分别依次创建
mkdir: 已创建目录 "/usr/mysql"
mkdir: 已创建目录 "/usr/mysql/data" #数据存储目录
mkdir: 已创建目录 "/usr/mysql/log" #日志目录
mkdir: 已创建目录 "/usr/mysql/temp"
```

```bash
chown -R mysql:mysql /usr/mysql # root用户将mysql目录归mysql用户所有
```

如果不带Boots的mysql包，下载
```bash
wget http://downloads.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz
```

路径DWITH_BOOST修改为boots包的解压路径
接下来 预编译

```bash
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=boost \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DENABLE_DTRACE=0 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DMYSQL_USER=mysql
```

配置解释：
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql    //设置安装目录
-DMYSQL_DATADIR=/home/mysql/data     //设置数据库存放目录             
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock   //设置UNIX socket目录
-DDEFAULT_CHARSET=utf8mb4     //设置默认字符集
-DDEFAULT_COLLATION=utf8mb4_general_ci     //设置默认校对规则
-DWITH_INNOBASE_STORAGE_ENGINE=1    //添加InnoDB引擎支持
-DSYSCONFDIR=/etc    //设置my.cnf配置文件的所在目录，默认为安装目录
编译安装
```bash
make && make install
```
尴尬了，虚拟机空间不足：
{% asset_img 7.png  %}
拷贝vdi到新目录执行命令：
VBoxManage internalcommands sethduuid "D:\VirtualBox VMs\centos7Gome\centos7Gome.vdi"

解决后继续：

cmake执行完的结果
{% asset_img 8.png  %}
使用make命令进行编译，接下来你将经历漫长的等待

安装完成
{% asset_img 9.png  %}

##### 拷贝可执行文件到指定的目录下，并修改名字为mysqld 
```bash
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld #授予可执行的权限
```

```bash
chmod +x /etc/init.d/mysqld #设置为开机启动 systemctl enable mysqld
```

添加环境变量
```bash
vi /etc/profile#末尾添加以下内容#mysql env
```

```bash
export PATH=$PATH:/usr/local/mysql/bin:/usr/local/mysql/lib
```

```bash
source /etc/profile
```

```bash
#参考，具体里面的参数说明，请自行网上搜索
[mysqld]
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci

skip-external-locking
skip-name-resolve

user = mysql
port = 3306

basedir = /usr/local/mysql
datadir = /usr/mysql/data
tmpdir = /usr/mysql/temp
# server_id = .....
socket = /usr/local/mysql/mysql.sock
log-error = /usr/mysql/log/mysql_error.log
pid-file = /usr/mysql/data/mysql.pid

open_files_limit = 10240

back_log = 600
max_connections=500
max_connect_errors = 6000
wait_timeout=605800

#open_tables = 600
#table_cache = 650
#opened_tables = 630

max_allowed_packet = 32M

sort_buffer_size = 4M
join_buffer_size = 4M
thread_cache_size = 300
query_cache_type = 1
query_cache_size = 256M
query_cache_limit = 2M
query_cache_min_res_unit = 16k

tmp_table_size = 256M
max_heap_table_size = 256M

key_buffer_size = 256M
read_buffer_size = 1M
read_rnd_buffer_size = 16M
bulk_insert_buffer_size = 64M

lower_case_table_names=1

default-storage-engine = INNODB

innodb_buffer_pool_size = 1G
innodb_log_buffer_size = 32M
innodb_log_file_size = 128M
innodb_flush_method = O_DIRECT

#####################
long_query_time= 2
slow-query-log = on
slow-query-log-file = /usr/mysql/log/mysql-slow.log

[mysqldump]
quick
max_allowed_packet = 32M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

>参考资料
>[在CentOS7上编译安装MySQL 5.7](https://my.oschina.net/u/1429136/blog/738772)
>[在CentOS7上编译安装MySQL 5.7.13步骤详解](https://www.jianshu.com/p/95a103add722)
>[centos7.2源码编译安装LNMP](https://www.bfshu.com/jcaz/123)

初始化数据库 
```bash
mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/usr/mysql/data
```

启动数据库：
```bash
systemctl start mysqld
```

查看mysqld服务状态：
```bash
systemctl status mysqld
```
{% asset_img 10.png  %}

{% asset_img 11.png  %}

```
设置数据库root用户密码
　　MySQL和Oracle数据库一样，数据库也默认自带了一个 root 用户（这个和当前Linux主机上的root用户是完全不搭边的），我们在设置好
MySQL数据库的安全配置后初始化root用户的密码。配制过程中，一路输入 y 就行了。这里只说明下MySQL5.7.14版本中，用户密码策略分成
低级 LOW 、中等 MEDIUM 和超强 STRONG 三种，推荐使用中等 MEDIUM 级别！
mysql_secure_installation*
设置中等的话，一般是
至少长度为8包涵大小写字母和数字和特殊字符的密码
Qwerty.1234
```

#4. 安装PHP
```
安装php-fpm
Nginx本身不能处理PHP，作为web服务器，当它接收到请求后，不支持对外部程序的直接调用或者解析，必须通过FastCGI进行调用。如果是
PHP请求，则交给PHP解释器处理，并把结果返回给客户端。PHP-FPM是支持解析php的一个FastCGI进程管理器。提供了更好管理PHP进程的
方式，可以有效控制内存和进程、可以平滑重载PHP配置。
```

1、安装依赖包
```bash
yum -y install php-mcrypt libmcrypt libmcrypt-devel  autoconf  freetype gd libmcrypt libpng libpng-devel libjpeg libxml2 libxml2-devel zlib curl curl-devel re2c net-snmp-devel libjpeg-devel php-ldap openldap-devel openldap-servers openldap-clients freetype-devel gmp-devel
```

```bash
cd /usr/local/src/php-7.3.8/
```

可能错误1:
```bash
error：configure: error: Cannot find ldap libraries in /usr/lib
```
>解决方案：可能的原因是安装了64位的系统，在lib64下面有这个文件，可能在lib这文件夹里面没有，所以强制复制一次。
> ```cp -frp /usr/lib64/libldap* /usr/lib/```
可能错误2:
```bash
configure: error: Please reinstall the libzip distribution
```
{% asset_img 12.png  %}

> 解决方案：
```bash
#先删除旧版本
yum remove -y libzip
#下载编译安装
wget https://nih.at/libzip/libzip-1.2.0.tar.gz
tar -zxvf libzip-1.2.0.tar.gz
cd libzip-1.2.0
./configure
make && make install
```
可能错误3
```bash
configure: error: Please reinstall the libzip distribution
```
> 解决方案：
```bash
wget https://nih.at/libzip/libzip-1.2.0.tar.gz
tar -zxvf libzip-1.2.0.tar.gz
cd libzip-1.2.0
./configure
make && make install
```
可能错误4
```bash
configure: error: off_t undefined; check your library configuration
```
> 解决方案：
>>添加搜索路径到配置文件
```bash
echo '/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64'>>/etc/ld.so.conf
```
>>更新配置
```bash
ldconfig -v
```
可能错误5
```bash
/usr/local/include/zip.h:59:21: fatal error: zipconf.h: No such file or dire
```
>解决方案：手动复制过去
```bash
cp /usr/local/lib/libzip/include/zipconf.h /usr/local/include/zipconf.h
```

预编译
```bash
./configure --prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--with-mysqli \
--with-pdo-mysql \
--with-mysql-sock=/usr/local/mysql/mysql.sock \
--with-iconv-dir \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-curl \
--with-gd \
--with-gmp \
--with-zlib \
--with-xmlrpc \
--with-openssl \
--without-pear \
--with-snmp \
--with-gettext \
--with-mhash \
--with-libxml-dir=/usr \
--with-ldap \
--with-ldap-sasl \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-xml \
--enable-fpm  \
--enable-ftp \
--enable-bcmath \
--enable-soap \
--enable-shmop \
--enable-sysvsem \
--enable-sockets \
--enable-inline-optimization \
--enable-maintainer-zts \
--enable-mbregex \
--enable-mbstring \
--enable-pcntl \
--enable-zip \
--disable-fileinfo \
--disable-rpath \
--enable-libxml \
--enable-opcache \
--enable-mysqlnd
```

预编译结果：


编译安装：
```bash
make && make install
```
安装错误
```bash
/usr/bin/ld: ext/ldap/.libs/ldap.o: undefined reference to symbol 'ber_strdup'
//usr/lib64/liblber-2.4.so.2: error adding symbols: DSO missing from command line
collect2: error: ld returned 1 exit status
make: *** [sapi/cli/php] 错误 1
```
解决办法：在PHP源码目录下 vim Makefile 找到 EXTRA_LIBS 行（带有很多参数的行），在行末添加 ‘ -llber ‘ 保存退出再次make即可。
解决办法：遇到这种类似的情况，说明「./configure 」沒抓好一些环境变数值。解决方法，来自老外的一篇文章：
在PHP源码目录下 vi Makefile 找到 EXTRA_LIBS 行，在行末添加 ‘ -llber ‘ 保存退出再次make即可


安装成功
```bash
[root@centos7Gome php-7.3.8]# make install
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-zts-20180731/
Installing PHP CLI binary:        /usr/local/php/bin/
Installing PHP CLI man page:      /usr/local/php/php/man/man1/
Installing PHP FPM binary:        /usr/local/php/sbin/
Installing PHP FPM defconfig:     /usr/local/php/etc/
Installing PHP FPM man page:      /usr/local/php/php/man/man8/
Installing PHP FPM status page:   /usr/local/php/php/php/fpm/
Installing phpdbg binary:         /usr/local/php/bin/
Installing phpdbg man page:       /usr/local/php/php/man/man1/
Installing PHP CGI binary:        /usr/local/php/bin/
Installing PHP CGI man page:      /usr/local/php/php/man/man1/
Installing build environment:     /usr/local/php/lib/php/build/
Installing header files:          /usr/local/php/include/php/
Installing helper programs:       /usr/local/php/bin/
  program: phpize
  program: php-config
Installing man pages:             /usr/local/php/php/man/man1/
  page: phpize.1
  page: php-config.1
/usr/local/src/php-7.3.8/build/shtool install -c ext/phar/phar.phar /usr/local/php/bin
ln -s -f phar.phar /usr/local/php/bin/phar
Installing PDO headers:           /usr/local/php/include/php/ext/pdo/
```


配置
移动php配置文件的位置，并修改名
php-fpm.conf是 php-fpm 进程服务的配置文件
www.conf是 php-fpm 进程服务的扩展配置文件
```bash
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
```
```bash
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
```
复制php.ini文件
```bash
cp /usr/local/src/php-7.3.8/php.ini-production /usr/local/php/etc/php.ini
```

启动php-fpm
```bash
[root@centos7Gome php-7.3.8]# ps -ef | grep php-fpm
root      3201     1  0 17:55 ?        00:00:00 php-fpm: master process (/usr/local/php/etc/php-fpm.conf)
www       3202  3201  0 17:55 ?        00:00:00 php-fpm: pool www
www       3203  3201  0 17:55 ?        00:00:00 php-fpm: pool www
root      3205 12571  0 17:55 pts/0    00:00:00 grep --color=auto php-fpm
```

编辑vim /usr/local/nginx/nginx.conf
```bash
vim /usr/local/nginx/nginx.conf
```

```bash
        location / {
            root   html;
            index  index.php index.html index.htm;
        }

        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            include        fastcgi_params;
I			nclude fastcgi.conf;  #安装的时候没有，加上去
        }
```
```bash
systemctl restart php-fpm #重启php-fpm
```
```bash
systemctl restart nginx #重启ngingx
```

- 搭建完成
{% asset_img 13.png  %}