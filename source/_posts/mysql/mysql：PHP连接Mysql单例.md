---
title: mysql：PHP连接Mysql单例
abbrlink: 264386d1
date: 2015-12-29 15:05:42
tags:
 - mysql
 - 数据库
 
categories:
 - 后端
 - mysql
---
```php
<?php
class Db
{
    static private $_instance;
    static private $_connectSource;
    private $_dbConfig = array(
        'host' => '127.0.0.1',
        'user' => 'root',
        'password' => 'root',
        'database' => 'thinkphp',
    );

    private function __construct()
    {

    }

    static public function getInstance()
    {
        self::$_instance = new self();
        if (self::$_instance instanceof self) {
            self::$_instance = new self();
        }

        return self::$_instance;
    }

    public function connect()
    {
        if (!self::$_connectSource) {

            self::$_connectSource = mysql_connect($this->_dbConfig['host'], $this->_dbConfig['user'], $this->_dbConfig['password']);
            if (!self::$_connectSource) {
                die('mysql error: ' . mysql_error());
            }

            mysql_select_db($this->_dbConfig['database'], self::$_connectSource);
            mysql_query("set names utf8", self::$_connectSource);

        }

        return self::$_connectSource;
    }
}

$connect = Db::getInstance()->connect();
$sql = "select * from tp_users";
$res = mysql_query($sql, $connect);
$arr = array(); //空数组
while ($rec = mysql_fetch_assoc($res)) {
    $arr[] = $rec;//这样就形成二维数组
}

echo "<pre>" . print_r($arr, 1) . "</pre>";


```