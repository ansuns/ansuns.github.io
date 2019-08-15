---
title: 时间操作
abbrlink: f459561d
date: 2019-01-07 16:06:06
tags:
  - PHP
  - 后端
categories:
  - 后端
  - PHP
---
##### 1. 列出日期范围内的所有日期 #####
```php
<?php

function date_range($first, $last, $step = '+1 day', $format = 'Y-m-d')
{

    $dates = array();
    $current = strtotime($first);
    $last = strtotime($last);

    while ($current <= $last) {

        $dates[] = date($format, $current);

        $current = strtotime($step, $current);

    }
    return $dates;
}

// test: print_r(date_range('2014-06-22', '2014-07-02'));

//PHP > 5.3 ，用此版本
function date_range2($first, $last)
{

    $period = new DatePeriod(
        new DateTime($first),
        new DateInterval('P1D'),
        new DateTime($last)
    );
    $dates = [];
    foreach ($period as $date) {

        $dates[] = $date->format('Y-m-d');
    }
    return $dates;

}

// test: print_r(date_range('2014-06-22', '2014-07-02'));
```
##### 2. 获取自定义时间范围 #####
```php
/**
 * 获取自定义时间范围
 * @param $type 1:每天 2:每周 3:每月 4:每季 5:每年 6：上季度
 * @return array
 */
function get_diy_date($type)
{
    switch ($type) {
        case 1:
            //每天
            $start = mktime(0, 0, 0, date('m'), date('d'), date('Y'));
            $end = mktime(0, 0, 0, date('m'), date('d') + 1, date('Y')) - 1;
            break;
        case 2:
            //每周
            $start = mktime(0, 0, 0, date('m'), date('d') - date('w') + 1 - 7, date('Y'));
            $end = mktime(23, 59, 59, date('m'), date('d') - date('w') + 7 - 7, date('Y'));
            break;
        case 3:
            //3:每月
            $start = mktime(0, 0, 0, date('m'), 1, date('Y'));
            $end = mktime(23, 59, 59, date('m'), date('t'), date('Y'));
            break;
        case 4:
            //本季度
            $season = ceil((date('n')) / 3);//当月是第几季度
            $start = mktime(0, 0, 0, $season * 3 - 3 + 1, 1, date('Y'));
            $end = mktime(23, 59, 59, $season * 3, date('t', mktime(0, 0, 0, $season * 3, 1, date("Y"))), date('Y'));
            break;
        case 5:
            //每年
            $start = mktime(0, 0, 0, 1, 1, date("Y", time()));//今年开始
            $end = mktime(23, 59, 59, 12, 31, date("Y", time()));//今年结束
            break;
        case 6:
            //上季度
            $season = ceil((date('n')) / 3) - 1;//上季度是第几季度
            $start = mktime(0, 0, 0, $season * 3 - 3 + 1, 1, date('Y'));
            $end = mktime(23, 59, 59, $season * 3, date('t', mktime(0, 0, 0, $season * 3, 1, date("Y"))), date('Y'));
            break;
        default:
            $start = mktime(0, 0, 0, date('m'), date('d'), date('Y'));
            $end = mktime(0, 0, 0, date('m'), date('d') + 1, date('Y')) - 1;
    }

    return [date('Y-m-d H:i:s', $start), date('Y-m-d H:i:s', $end)];
}
```
