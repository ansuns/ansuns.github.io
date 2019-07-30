---
title: 接口封装
abbrlink: 6dd28e9b
date: 2015-12-18 16:06:14
tags:
  - PHP
  - 后端
categories:
  - 技术
  - PHP
---
``` php
<?php

class Response
{
    CONST TYPE = "json";
    
    /**
     * @param $code 状态码
     * @param string $message 提示信息
     * @param array $data 数据
     * @param string $type 数据库类型
     * @return string
     */
    public static function show($code, $message = '', $data = array(), $type = self::TYPE)
    {
        if (!is_numeric($code)) {
            return '';
        }

        $result = array(
            'code' => $code,
            'message' => $message,
            'data' => $data
        );

        if ($type == 'json') {
            self::json($code, $message, $data);
        } elseif ($type == 'array') {
            echo "<pre>" . print_r($result, 1) . "</pre>";
        } elseif ($type == 'xml') {
            self::xml($code, $message, $data);
        } else {
            exit();
        }
    }

    /**
     * @param $code
     * @param string $message
     * @param array $data
     */
    public static function json($code, $message = '', $data = array())
    {
        if (!is_numeric($code)) {
            return;
        }
        $result = array(
            'code' => $code,
            'message' => $message,
            'data' => $data
        );

        echo json_encode($result);
    }

    /**
     *
     * @param int $code
     * @param string $message
     * @param array $data
     * return string
     */
    public static function xml($code, $message = '', $data = array())
    {
        if (!is_numeric($code)) {
            return '';
        }

        $result = array(
            'code' => $code,
            'message' => $message,
            'data' => $data
        );

        header('Content-Type:text/xml');
        $xml = "<?xml version='1.0' encoding='UTF-8'?>\n";
        $xml .= "<root>";
        $xml .= self::xmlToEncode($result);
        $xml .= "</root>";
        echo $xml;
    }

    /**
     * @param $result
     * @return string
     */
    public static function xmlToEncode($result)
    {
        $xml = $attr = '';
        foreach ($result as $key => $value) {
            if (is_numeric($key)) {
                $attr = " id='" . $key . "'";
                $key = "item";
            }
            $xml .= "<{$key}{$attr}>";
            $xml .= is_array($value) ? self::xmlToEncode($value) : $value;
            $xml .= "</{$key}>\n";
        }
        return $xml;
    }
}
```
