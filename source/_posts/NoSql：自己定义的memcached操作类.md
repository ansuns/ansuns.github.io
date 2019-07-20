---
title: NoSql-自己定义的memcached操作类
abbrlink: cc768ab3
date: 2019-07-19 14:08:51
tags:
 - 后端
 - NoSql
categories:
 - 技术
---
``` php
<?php

class Memcached
{
    private $type = 'Memcached';
    private $m;
    private $time = 0;
    private $error;
    private $debug = false;

    public function __construct()
    {
        if (!class_exists($this->type)) {
            $this->error = 'No ' . $this->type;
            return false;
        } else {
            $this->m = new $this->type;
        }

    }

    public function addServers($arr)
    {
        $this->m->addServers($arr);
    }

    public function s($key, $value = '', $time = 0)
    {
        $num = func_num_args();
        if ($num == 1) {
            return $this->get($key);
        } elseif ($num >= 2) {
            if ($value === NULL) {
                $this->delete($key);
            } else {
                $this->set($key, $value, $time);
            }
        }
    }

    public function set($key, $value, $time = NULL)
    {
        if ($time === NULL) {
            $time = $this->time;
        }
        $this->m->set($key, $value, $time);
        if ($this->m->getResultCode() != 0) {
            return false;
        }
    }

    public function delete($key)
    {
        $this->m->delete($key);
    }

    public function get($key)
    {
        $return = $this->m->get($key);
        if ($this->m->getResultCode() != 0) {
            return false;
        }
        return $return;
    }

    public function addServer($arr)
    {
        $this->m->addServers($arr);
    }

    public function getError()
    {
        if ($this->error) {
            return $this->error;
        } else {
            return $this->m->getResultMessage();
        }
    }
}
```