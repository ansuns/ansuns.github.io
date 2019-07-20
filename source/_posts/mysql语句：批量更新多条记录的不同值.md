---
title: mysql语句：批量更新多条记录的不同值
abbrlink: 1125adb3
date: 2019-07-20 11:44:41
tags:
---
mysql更新语句很简单，更新一条数据的某个字段，一般这样写：
``` sql
UPDATE mytable SET myfield = 'value' WHERE other_field = 'other_value';
```
如果更新同一字段为同一个值，mysql也很简单，修改下where即可：
``` sql
UPDATE mytable SET myfield = 'value' WHERE other_field in ('other_values');
```
这里注意 ‘other_values' 是一个逗号（，）分隔的字符串，如：1,2,3
那如果更新多条数据为不同的值，可能很多人会这样写：
``` php
foreach ($display_order as $id => $ordinal) { 
    $sql = "UPDATE categories SET display_order = $ordinal WHERE id = $id"; 
    mysql_query($sql); 
}
```
即是循环一条一条的更新记录。一条记录update一次，这样性能很差，也很容易造成阻塞。
那么能不能一条sql语句实现批量更新呢？mysql并没有提供直接的方法来实现批量更新，但是可以用点小技巧来实现。

``` sql
UPDATE mytable 
    SET myfield = CASE id 
        WHEN 1 THEN 'value'
        WHEN 2 THEN 'value'
        WHEN 3 THEN 'value'
    END
WHERE id IN (1,2,3)
```

这里使用了case when 这个小技巧来实现批量更新。
举个例子：

``` sql
UPDATE categories 
    SET display_order = CASE id 
        WHEN 1 THEN 3 
        WHEN 2 THEN 4 
        WHEN 3 THEN 5 
    END
WHERE id IN (1,2,3)
```

这句sql的意思是，更新display_order 字段，如果id=1 则display_order 的值为3，如果id=2 则 display_order 的值为4，如果id=3 则 display_order 的值为5。
即是将条件语句写在了一起。
这里的where部分不影响代码的执行，但是会提高sql执行的效率。确保sql语句仅执行需要修改的行数，这里只有3条数据进行更新，而where子句确保只有3行数据执行。
如果更新多个值的话，只需要稍加修改：

``` sql
UPDATE categories 
    SET display_order = CASE id 
        WHEN 1 THEN 3 
        WHEN 2 THEN 4 
        WHEN 3 THEN 5 
    END, 
    title = CASE id 
        WHEN 1 THEN 'New Title 1'
        WHEN 2 THEN 'New Title 2'
        WHEN 3 THEN 'New Title 3'
    END
WHERE id IN (1,2,3)
```

到这里，已经完成一条mysql语句更新多条记录了。
但是要在业务中运用，需要结合服务端语言，这里以php为例，构造这条mysql语句：
``` php
$display_order = array( 
    1 => 4, 
    2 => 1, 
    3 => 2, 
    4 => 3, 
    5 => 9, 
    6 => 5, 
    7 => 8, 
    8 => 9 
); 
$ids = implode(',', array_keys($display_order)); 
$sql = "UPDATE categories SET display_order = CASE id "; 
foreach ($display_order as $id => $ordinal) { 
    $sql .= sprintf("WHEN %d THEN %d ", $id, $ordinal); 
} 
$sql .= "END WHERE id IN ($ids)"; 
echo $sql;
```

这个例子，有8条记录进行更新。代码也很容易理解，你学会了吗
性能分析
当我使用上万条记录利用mysql批量更新，发现使用最原始的批量update发现性能很差，将网上看到的总结一下一共有以下三种办法：
1.批量update，一条记录update一次，性能很差
```` sql
update test_tbl set dr='2' where id=1;
````
2.replace into 或者insert into ...on duplicate key update
``` sql
replace into test_tbl (id,dr) values (1,'2'),(2,'3'),...(x,'y');
```
或者使用
```` sql
insert into test_tbl (id,dr) values  (1,'2'),(2,'3'),...(x,'y') on duplicate key update dr=values(dr);
````
3.创建临时表，先更新临时表，然后从临时表中update
 代码如下
``` sql
create temporary table tmp(id int(4) primary key,dr varchar(50));
insert into tmp values  (0,'gone'), (1,'xx'),...(m,'yy');
update test_tbl, tmp set test_tbl.dr=tmp.dr where test_tbl.id=tmp.id;
``` 
注意：这种方法需要用户有temporary 表的create 权限。
下面是上述方法update 100000条数据的性能测试结果：

逐条update
``` sql
real    0m15.557s
user    0m1.684s
sys    0m1.372s

replace into
real    0m1.394s
user    0m0.060s
sys    0m0.012s

insert into on duplicate key update
real    0m1.474s
user    0m0.052s
sys    0m0.008s

create temporary table and update:
real    0m0.643s
user    0m0.064s
sys    0m0.004s
```
就测试结果来看，测试当时使用replace into性能较好。
replace into  和insert into on duplicate key update的不同在于：
replace into　操作本质是对重复的记录先delete 后insert，如果更新的字段不全会将缺失的字段置为缺省值
insert into 则是只update重复记录，不会改变其它字段。
 
``` php
$iii = implode(',', array_keys($arr));       

        $sql2 = "UPDATE ecs_wechat_user SET tag_id = CASE uid ";
        foreach ($arr as $key => $val) {
            //WHEN %d THEN %d 字符串用s替代
            //$sql2 .= sprintf("WHEN %s THEN %s ", $key,$val); 这里是字符串 ，格式 1,3 会报错
            $sql2 .= sprintf("WHEN %s THEN %s ", $key,"'$val'");
        }
        $sql2 .= "END WHERE uid IN ($iii)";

echo $sql2;

``` php
<?php

/**
 * 批量更新多字段
 * @return string
 * @throws \think\db\exception\BindParamException
 * @throws \think\exception\PDOException
 */
public
function updateAll()
{
    BaseModel::instance()->startTrans();
    $sql = "UPDATE lc_users ";
    $sql .= ' SET name = CASE id ';
    $ageData = [];
    $sexData = [];
    $data = BaseModel::instance()->table('lc_users')->select()->toArray();
    foreach ($data as $key => $val) {
        $name = $val['id'] . ":" . date('Y-m-d H:i:s');
        $age = $val['id'] + rand(1, 9);
        $sex = rand(0, 2);
        $sql .= sprintf(" WHEN %s THEN %s ", $val['id'], "'$name'");
        $ageData[] = sprintf(" WHEN %s THEN %s ", $val['id'], "'$age'");
        $sexData[] = sprintf(" WHEN %s THEN %s ", $val['id'], "'$sex'");
    }

    $sql .= " END, age = CASE id " . implode(' ', $ageData);
    $sql .= " END, sex = CASE id " . implode(' ', $sexData);
    $sql .= " END ";

    if ($counts = BaseModel::instance()->execute($sql) > 0) {
        BaseModel::instance()->commit();
        return $sql;
    }

    BaseModel::instance()->rollback();
}
```

