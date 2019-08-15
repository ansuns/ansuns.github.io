---
title: mysql：连接PHP等基本操作
abbrlink: 5144b647
date: 2015-11-29 15:05:36
tags:
 - mysql
 - 数据库
 
categories:
 - 后端
 - mysql
---
基本流程：
```php
// 1. 连接数据库： 
$mylink  =  mysql_connect(“localhost",  'root',  '123');

//2. 设定连接编码（通常是utf8）
mysql_set_charset(“utf8");  //也可以使用：mysql_query(“set names utf8");
             
// 3. 选择数据库（如有需要）
mysql_select_db("数据库名"); //也可以使用：mysql_query("use  数据库名");
         
// 4. 执行sql命令。 
$result  =  mysql_query( "几乎任何sql语句");
```
返回的结果通常需要分两种情形进行处理：
##### 如果是无返回数据的语句： #####
##### 如果$result为true，表示执行成功 #####
##### 如果$result为false，表示执行失败 #####
##### 如果是有返回数据的语句： #####
##### 如果$result为false，表示执行失败 #####
##### 否则，执行成功，需要继续从结果中取出数据并显示出来。 #####
 
补充sql语句：
set names  gbk；
use 数据库名；
show  databases：
desc  表名：        显示一个表的“结构信息"，返回的其实也是结果集（类似select语句）
补充php操作mysql的函数：
```php
$record  =  mysql_fetch_array( 结果集$result );
$n1 = mysql_num_rows(  结果集$result ):  获取结果集的行数
$n2 = mysql_num_fields(  结果集$result ):  获取结果集的列数
mysql_field_name( 结果集$result， $i  ): 获取结果集中的第i个字段名（i从0开始）
```
 
细看今天的代码文件：《3php_mysql_base.php》
 
数据定义语句
数据库定义
语法形式
create  database  [if  not  exists ] 数据库名  [charset  字符集]  [collate  字符排序规则]；
说明：
>1，if  not  exists：用于判断是否存在该数据库名，如果存在则不执行该语句
>2，字符集： 意图数据存储到本数据库中的时候所使用的字符编码名称，通常utf8，也可以gbk。
>3，字符排序规则通常不设置，而是使用所设定的字符集的默认规则（每个字符集都有一个默认的排序规则）；
>什么叫排序规则：设定一个字符集中的所有字符怎么排列先后顺序的规则。
>“中"，“国"，“人"：

举例：

 
显示mysql中的所有可用字符集：

 
显示mysql中的所有可用排序规则：

 
修改数据库：
alter database 数据库名 character set=新字符集 collate=新校对集;
删除数据库：
drop  database  数据库名；
 
其他数据库相关语句
选择（进入）某数据库：　　use  数据库名；
通常，要进行数据中的数据表和数据的操作，都必须先“进入"该数据库。

 
显示所有数据库：      show  databases;
 
显示某个数据库的“创建语句"：
show  create  database  数据库名；