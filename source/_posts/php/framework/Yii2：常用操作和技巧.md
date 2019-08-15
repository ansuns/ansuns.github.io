---
title: 常用操作和技巧
tags:
  - Yii2
  - PHP
  - 后端
categories:
  - 后端
  - PHP
abbrlink: e550b6d2
date: 2016-11-01 19:13:00
---

####  1. 关联操作 #### 
```php
/**
 * 返回订单总额
 * @return Ambigous <\yii\db\mixed, string, NULL, \yii\db\false, mixed>
 */
public function getOrdertotalprice()
{
    // 第一个参数为要关联的子表模型类名，
    // 第二个参数指定 通过子表的order_id(左边)，关联主表的order_id字段(右边)
    return $this->hasMany(OrderGoodsOrdinary::className(), ['order_id' => 'order_id'])->sum('subtotal');

}
```
####  2. 获取 YII2 AR 执行的 SQL 语句 #### 
```php
$query = User::find()
    ->where(['id' => [1, 2, 3, 4])
    ->select(['username'])

// get the AR raw sql in YII2
$commandQuery = clone $query;
echo $commandQuery->createCommand()->getRawSql();

$users = $query->all();
```

####  3 . post搜索/多个字段like #### 
```php

$keywords = Yii::$app->request->post('keywords') ? Yii::$app->request->post('keywords') : Yii::$app->request->get('keywords');
$type = Yii::$app->request->get('type', 'all');

if ($type == 'all') {
    $where = ['>', 'actived', -1];
} else {
    $where = ['=', 'actived', $type];
}

$data = Members::find()->andWhere($where)->orderBy('member_id desc');

//多个字段Like
if (!empty($keywords)) {
    $data->andWhere(
        [
            'or',
            ['like', 'member_name', $keywords],
            ['like', 'name', $keywords],
            ['like', 'mobile', $keywords],
        ]
    );;
}

$pages = new Pagination(['totalCount' => $data->count(), 'pageSize' => 10]);

$result = $data->offset($pages->offset)->limit($pages->limit)->all();

//设置分页查询的关键字，点击分页是带回该查询关键字
$pages->params = ['keywords' => $keywords];
$this->assign("type", $type);
$this->assign("result", $result);
$this->assign("pages", LinkPager::widget(['pagination' => $pages]));

```

#### 4. 配置并连接redis #### 
>安装扩展包
```bash
composer require --prefer-dist yiisoft / yii2 - redis
```
>配置components组件
```php

'components' => [
    'cache' => [
        // 'class' => 'yii\caching\FileCache',
        'class' => 'yii\redis\Cache',
    ],
    'redis' => [
        'class' => 'yii\redis\Connection',
        'hostname' => '192.168.186.233',
        'port' => 6379,
        'database' => 0,
    ],
]
```
>使用
```php
Yii::$app->cache->set('test', 'hehe..');
echo Yii::$app->cache->get('test'), "\n";
Yii::$app->cache->set('test1', 'haha..', 5);
```

#### 5. COMPOSER安装Yii2错误 ####
```bash
[Symfony\Component\Process\Exception\RuntimeException]                                   
  The Process class relies on proc_open, which is not available on your PHP installation. 
``` 

>需要在PHP.INI开启 disable_functions = proc_open 去掉。

```bash
Failed to decode response: zlib_decode(): data error
Retrying with degraded mode, check https://getcomposer.org/doc/articles/troubleshooting.md#degraded-mode for moYour 
requirements could not be resolved to an installable set of packages.
```

#### 6. Yii2中设置多个应用雨荨跨域 ####
>在BaseController基类中添加一下代码
```php
$origin = isset($_SERVER['HTTP_ORIGIN']) ? $_SERVER['HTTP_ORIGIN'] : '';
if (!empty($origin)) {
    if (strpos($origin, ".zbc.com")) {
        header('Access-Control-Allow-Origin:' . $origin);
    }
}
//允许跨域
header("Access-Control-Allow-Credentials:true");
header('Access-Control-Allow-Methods: GET, POST');
header('Access-Control-Max-Age: 3628800');
header('Access-Control-Allow-Headers:x-requested-with,content-type');
header('Content-Type:application/json;charset=utf-8');
```

#### 7. 构造复杂的价格查询语句 ####
```php
//起始价格
$min_price = Yii::$app->request->post('min_price') ? Yii::$app->request->post('min_price') : Yii::$app->request->get('min_price');
//结束价格
$max_price = Yii::$app->request->post('max_price') ? Yii::$app->request->post('max_price') : Yii::$app->request->get('max_price');


if (!empty($min_price) && empty($max_price)) {
    $pricestr = ['>=', Product::tableName() . '.min_price', $min_price];
    $pricestr2 = ['>=', FxMember::tableName() . '.min_price', $min_price];
} elseif (empty($min_price) && !empty($max_price)) {
    $pricestr = ['<=', Product::tableName() . '.min_price', $max_price];
    $pricestr2 = ['<=', FxMember::tableName() . '.min_price', $max_price];

} elseif (!empty($max_price) && !empty($min_price)) {
    $pricestr = [
        'and',
        ['>=', Product::tableName() . '.min_price', $min_price],
        ['<=', Product::tableName() . '.min_price', $max_price],
    ];

    $pricestr2 = [
        'and',
        ['>=', FxMember::tableName() . '.min_price', $min_price],
        ['<=', FxMember::tableName() . '.min_price', $max_price],
    ];
}

$Model->where([

    'or',
    [
        'and',
        ['=', Product::tableName() . '.shop_id', $this->member_shop_id],
        ['=', Product::tableName() . '.is_delete', 0],
        ['=', Product::tableName() . '.on_sale', 1],
        $br,
        $cr,
        $pricestr
    ],
    [
        'and',
        ['in', FxMember::tableName() . '.goods_id', $fxGoods],
        ['=', FxMember::tableName() . '.is_delete', 0],
        ['=', FxMember::tableName() . '.on_sale', 1],
        $pricestr2
    ]

]);
```

#### 9. 集成smarty，便于模版分离####
>首先安装扩展
```bash
php composer.phar require --prefer-dist yiisoft/yii2-smarty
```
>配置组件
```php
'components' => [
    'view' => [
        'renderers' => [
            'html' => [
                'class' => 'yii\smarty\ViewRenderer',
                //'cachePath' => '@runtime/Smarty/cache',
                'cachePath' => '@runtime/Smarty/cache',
                'options' => [
                    'left_delimiter' => '{{',
                    'right_delimiter' => '}}',
                ],
            ],
        ],
    ],
] 
```
>使用
```php
//控制器里边
return $this->render('display.html',['name'=>'ansuns']);

//模板里边
{$name}
```