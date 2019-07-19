---
title: NoSql-memcached安装
abbrlink: bb47fb28
date: 2019-07-19 14:07:14
tags:
 - 后端
 - Nosql
categories:
 - 技术
---
yum install memcached
 /usr/bin/memcached -d -l 127.0.0.1 -p 11211 -m 150 -u root
-d守护进程 -l服务器地址 -p端口号 -m分配内存 -u已root用户启动



2--------------
Memcached客户端安装

a.安   Libmemcached
b.为PHP安装 Memcached扩展

wget https://launchpad.net/libmemcached/1.0/1.0.9/+download/libmemcached-1.0.9.tar.gz
ibmemcached
#/usr/lib/libmemcached 安装在这个目录
 ./configure --prefix=/usr/lib/libmemcached
make && make install


安装 Memcached扩展


wget http://pecl.php.net/get/memcached-2.2.0.tgz
tar zxvf memcached-2.2.0.tgz 
cd memcached-2.2.0
phpize
 ./configure --with-php-config=/usr/local/php/bin/php-config 

出现错误
checking for libmemcached location... configure: error: memcached support requires libmemcached. Use --with-libmemcached-dir=<DIR> to specify the prefix where libmemcached headers and library are located

#指定libmemcache安装目录
 ./configure --with-php-config=/usr/local/php/bin/php-config --with-libmemcached-dir=/usr/lib/libmemcached/

又出现一个错误
checking for sasl/sasl.h... no
configure: error: no, sasl.h is not available. Run configure with --disable-memcached-sasl to disable this check
[root@ansuns memcached-2.2.0]# 

解决
 ./configure --with-php-config=/usr/local/php/bin/php-config --with-libmemcached-dir=/usr/lib/libmemcached/ --disable-memcached-sasl



继续..
make

安装成功
If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,--rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------

Build complete.
Don't forget to run 'make test'.
-----------------------------------------


make install 


扩展安装完成

扩展目录
 /usr/local/php/lib/php/extensions/no-debug-non-zts-20121212/

接着把扩展加到PHP里边

vim /usr/local/php/etc/php.ini
extension=memcached.so

重启
查看有没有加载扩展
php -m | grep memcached



示例：
$memcached = new memcached();
$memcached->addServers('192.168.18.88',11211);
$memcached->add('key','value');
echo $memcached->get('key');

