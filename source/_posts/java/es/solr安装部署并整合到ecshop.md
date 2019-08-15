---
title: solr安装部署并整合到ecshop
tags:
  - solr
  - ecshop
  - 全文检索
categories:
  - PHP
  - ThinkPHP
abbrlink: 51c1b2a2
date: 2019-08-15 16:54:42
---
---
#### linux环境部署 ####
>先安装java环境 ，需要Java Runtime Environment(JRE) 1.7或更高版本
>下载5.5版本的solr zip包 : [链接](http://mirror.bit.edu.cn/apache/lucene/solr/5.5.0/)
>解压到当前目录
```bash
unzip -r solr-5.5.0.zip ./
```
>创建应用程序目录
```bash
mkdir -p /usr/local/solr
```
>svn 代码solr_project/ check out 至 /data目录下
>创建运行solr的用户并赋权
```bash
# groupadd solr
# useradd -g solr solr
# chown -R solr.solr /data/solr_project/solr_cores /usr/local/solr
```
安装solr服务
```bash
solr-5.5.0/bin/install_solr_service.sh solr-5.5.0.zip -d /data/solr_project/solr_cores -i /usr/local/solr
```
检查服务
```bash
service solr status
```
/data/solr_project/README.txt里有增加core，删除core重启
**core**介绍
**solr_cores**是**solr**存放**core**的目录
- 新增core配置参考solr_cores/demo_core配置，修改conf里的managed-schema最后的字段信息
- 新增core方法：复制demo_core，重命名为目标target_core，修改配置字段信息后，执行脚本./solr_s/add_core.sh target_core
- 修改配置后需要重启solr，执行脚本./solr_cores/restart_solr.sh即可
- import_scripts存放导入core数据的脚本，对应core命名
- configs放置一些配置信息
- hnlog_文件夹存放索引更新的日志
- includes存放一些公用类
- 扩展类添加到（如ik分词）/usr/local/solr/solr/server/solr-webapp/webapp/WEB-INF/lib


#### 更新core的scheme字段配置与导数据 ####
- solr_project已加入版本控制:svn://192.168.2.231/solr_project
- 配置文件，solr_project/solr_cores/demo_core/conf/managed-schema，对应demo_core
- 修改字段配置后重启solr脚本./restart_solr.sh
- 导数据脚本solr_project/import_scripts/demo_core.php,对应core名称

#### 添加ik中文分词 ####
- 将solr_project/inluces里的ik jar包放进/usr/local/solr/solr/server/solr-webapp/webapp/WEB-INF/lib
- 将jar包中ext.dic IKAnalyzer.cfg.xml stopword.dic复制进/usr/local/solr/solr/server/solr-webapp/webapp/WEB-INF/classes
- solr_project/solr_cores/restart_solr.sh会将ext.dic、stopword.dic复制到上面的classes目录，自动更新扩展词库

#### 整合**solarium** ####
```bash
composer require solarium/solarium
```
#### 在要使用检索的位置调用 ####
```php
define('IN_ECS', true);
require dirname(FILE) . '/includes/lib_search.php';
$Search = new Search();
$Search→query_params("goods_name:欧式 shop_price:[100 TO *]");
```
搜索条件：字段名:条件; * 如果多个查询条件，AND 或者 OR 等运算符 * 具体也可参考includes\search\vendor\solarium\solarium\examples下的例子