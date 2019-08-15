---
title: SolrCloud5.5部署和使用
abbrlink: d1504f96
date: 2017-11-15 01:32:06
tags:
  - solr
  - SolrCloud
  - 全文检索
categories:
  - PHP
  - 全文检索
---
# SolrCloud5.5 部署

#1.SolrCloud基本概念

摘自：[http://www.it165.net/pro/html/201506/45483.html](http://www.it165.net/pro/html/201506/45483.html)

## 1.1  SolrCloud相关概念

 SolrCloud中有四个关键名词： **core** 、 **collection** 、 **shard** 、 **node** 。

**core** ：在Solr单机环境中，core本质上就是单个index。若需有多个index，那必须创建多个core。在SolrCloud环境中，单个index可以横跨多个Solr实例，这意味着单个index是由不同机器上的多个cores组成。

**collection** ：由core组成的逻辑index叫做 **collection** ，一个collection是跨越多个cores的index，这使index可扩展并冗余备份。

**shard** ：在SolrCloud中可以有多个collections。Collections可被分片，每个分片可有多个副本（Replica），同一副本下的相同分片称为 **shards** 。每个shards下的有一个分片为leader，该leader通过选举策略产生。

**node** ：SolrCloud中，node是运行Solr的Java虚拟机实例，也就是Server（例如Tomcat、Jetty）。

理解core和collection的区别非常重要。在传统的单node solr中，core和collection的概念等同，都代表一个逻辑index。在SolrCloud中，多个nodes下的cores形成一个collection。

## 1.2 SolrCloud路由

SolrCloud中，提供了两种路由算法：

**compositeIdimplicit**  在创建Collection时，需要通过router.name指定路由策略，默认为compositeId路由。

#### **compositeId**

该路由为一致性哈希路由，shards的哈希范围从80000000~7fffffff。初始创建collection是必须指定numShards，compositeId路由算法根据numShards的个数，计算出每个shard的哈希范围，因此路由策略不可以扩展shard。

#### **implicit**

该路由方式指定索引具体落在路由到哪个Shard，这与compositeId路由方式索引可均匀分布在每个shard上不同。同时 **只有在implicit路由策略下才可**

1.
# 2.Solr5.x和以前版本的区别

摘自：[http://www.tuicool.com/articles/aiYRJjv](http://www.tuicool.com/articles/aiYRJjv)

solr5发布有一段时间了，5系列跟4的最大区别是5被发布成一个独立的应用，而不再需要Tomcat等容器，在其内部集成了一个jetty容器，现在它可以通过bin目录的脚本直接启动。

solr5的应用方式有两种：独立模式（Standalone）和云模式（SolrCloud）。

solr5的启动方式有两种，相应地其对索引的管理也有两种方式，在独立模式以是以core来管理，在云模式下是collection来管理。

1.
# 3.部署流程【失败了】

  1. 3.1启动ZK集群

## 3.2 启动Solr

摘自：[http://wenku.baidu.com/link?url=wRfVyDLuS15uoAruz9-HDgQ3w5MqJMCD52AYKaPWdNDHg\_WJ\_ciboXyfsedn6SrqERuy\_dg4gstFYR8vj07dd\_TkS6ais03Dsmqq-6\_M8NW](http://wenku.baidu.com/link?url=wRfVyDLuS15uoAruz9-HDgQ3w5MqJMCD52AYKaPWdNDHg_WJ_ciboXyfsedn6SrqERuy_dg4gstFYR8vj07dd_TkS6ais03Dsmqq-6_M8NW)

执行命令：

```bash
[root@node1 solr]# ./bin/solr start -c -z node1:2181,node2:2181,node3:2181

[root@node2 solr]# ./bin/solr start -c -z node1:2181,node2:2181,node3:2181

[root@node1 solr]# ./bin/solr start -c -z node1:2181,node2:2181,node3:2181
```

## 3.3 创建Collection
```bash
[root@node1 solr]# ./bin/solr create\_collection -c example -d example/example-DIH/solr/solr/conf/ -shards 3 -replicationFactor 2

```
如果出现如下错误：
{% asset_img 1.png 错误 %}

请参考：

[https://cwiki.apache.org/confluence/display/solr/Using+ZooKeeper+to+Manage+Configuration+Files](https://cwiki.apache.org/confluence/display/solr/Using+ZooKeeper+to+Manage+Configuration+Files)，

在node1执行如下命令：为gettingstarted提交缺失的配置文件。

| ./server/scripts/cloud-scripts/zkcli.sh -cmd upconfig -confdir ./server/solr/configsets/data\_driven\_schema\_configs/conf -confname myconf -z node1:2181,node2:2181,node3:2181./server/scripts/cloud-scripts/zkcli.sh -cmd linkconfig -collection gettingstarted -confname myconf -z node1:2181,node2:2181,node3:2181 |
| --- |

再次创建collection

{% asset_img 2.png 再次创建collection %}
命令参数说明：

| -c : collection名称-d : 配置文件的路径，可以使用上面提供的实例配置-n : 配置名称可以和collection名称不同，默认这个参数不填的话，会使用collection名称作为config名称-shards : 创建的shard个数，建议和集群节点数量一致。-replicationFactor : 每个shard的副本数，综合考虑为了保证集群的稳定性，建议配置为 最少2个，最多集群节点数量/shard数量 \* 2 |
| --- |

## 3.4 使用含DIH配置的Collection
```bash
[root@node3 solr]# ./bin/solr create\_collection -c example3 -d ./server/solr/configsets/fj\_collection/conf2/ -shards 3 -replicationFactor 2

```
启动ｃｌｏｕｄ样例后创建collection

```bash
[root@node3 solr]# ./bin/solr create\_collection -c example -d ./server/solr/configsets/fj\_collection/conf/ -shards 2 -replicationFactor 2
```

```bash
./bin/solr create\_collection -c example -d ./server/solr/configsets/fj\_collection/conf/ -shards 2 -replicationFactor 2
```

错误：

```bash
| ERROR: Failed to create collection &#39;example&#39; due to: org.apache.solr.client.solrj.impl.HttpSolrClient$RemoteSolrException:Error from server at http://192.168.232.129:7574/solr: Error CREATEing SolrCore &#39;example\_shard2\_replica1&#39;: Unable to create core [example\_shard2\_replica1] Caused by: Can&#39;t find resource &#39;lang/stopwords\_ar.txt&#39; in classpath or &#39;/configs/example&#39;, cwd=/usr/local/src/solr-5.5.0/server |
| --- |
```

## 3.5 验证

查看collection创建结果如下：
{% asset_img 3.png 错误 %}

查看zookeeper中新增记录的example的状态：
{% asset_img 4.png 错误 %}
在三个节点中都新增了配置文件

{% asset_img 5.png 配置1 %}
{% asset_img 6.png 配置2 %}
{% asset_img 7.png 配置3 %}

1.
# 4. 使用方法

## 4.1 向任意一台服务器发送请求

[http://node1:8983/solr/example/select?q=\*%3A\*&amp;wt=json&amp;indent=true](http://node3:8983/solr/example/select?q=*%3A*&amp;wt=json&amp;indent=true)

[http://node2:8983/solr/example/select?q=\*%3A\*&amp;wt=json&amp;indent=true](http://node4:8983/solr/example/select?q=*%3A*&amp;wt=json&amp;indent=true)

[http://node3:8983/solr/example/select?q=\*%3A\*&amp;wt=json&amp;indent=true](http://node5:8983/solr/example/select?q=*%3A*&amp;wt=json&amp;indent=true)

其中：example=Collection名称
{% asset_img 8.png 错误 %}
 疑问：集群应该有统一入口地址，不应该区分IP地址。

## 4.2 访问SolrCloud的过程
{% asset_img 9.jpg 错误 %}
 在SolrCloud模式下，同一个集群里所有Core的配置都是统一的，Core有leader和replication两种角色，每个core一定属于一个shard，Core在Shard中扮演leader还是replication有Solr使用的Zookeeper自动协调。

 访问SolrCloud的过程是：Solr Client向Zookeeper咨询Collection的地址，Zookeeper返回存活的节点地址供访问，插入数据的时候有SolrCloud内部协调数据分发（内部使用一致性哈希）。

 SolrCloud中使用Zookeeper主要实现以下三点功能：（1）集中配置存储以及管理，（2）集群状态改变时进行监控以及通知，（3）shard leader的选举。

## 4.3 发送http请求的方式

## 4.4 客户端API查询方式

摘自：[http://www.656463.com/article/IvMvQv.htm](http://www.656463.com/article/IvMvQv.htm)

## 4.5 SolrCloud无法使用DIH和MySQL集成

 使用福建测试环境下&quot;Solr单实例集成MySQL的配置文件&quot;做模板重新创建新的collection对象example2。

 福建配置文件上传到node1的目录下/usr/local/src/solr-5.4.0/server/solr/configsets，作为example2的模板如下图：
{% asset_img 10.png 错误 %}

在node1，node2，node3所有节点执行如下操作：

| (1)将solr-dataimporthandler-5.4.0.jar从solr/dist/文件夹下copy到solr/server/solr-webapp/webapp/WEB-INF/lib当中，此java包是导入数据用的。(2)从mysql官网中下载一个mysql-connector-java-5.1.34.zip压缩包，解压出一个mysql-connector-java-5.1.35-bin.jar包，将它copy到solr/server/lib下。 |
| --- |

在node1上创建example2：

```bash
[root@node1 solr]# ./bin/solr create\_collection -c example2 -d server/solr/configsets/fj\_collection/conf/  -shards 3 -replicationFactor 2

```
页面验证：
{% asset_img 11.png 错误 %}

为MySQL数据创建索引

>solrtest1
{% asset_img 12.png 错误 %}

>solrtest2 
{% asset_img 13.png 错误 %}


创建索引有问题

{% asset_img 14.png 错误 %}
 试验结果不成功，只能查到部分数据。

## 4.6 SolrCloud提供的URl命令

摘自：[http://www.wtoutiao.com/p/15fprvf.html](http://www.wtoutiao.com/p/15fprvf.html)

1. //查看zk里面的有多少个collection

2. http://node1:8983/solr/admin/configs?action=LIST&amp;wt=json

返回：
```json
{"responseHeader":{"status":0,"QTime":45},"configSets":["example2","collection","example","test","myconf"]}
```
1. //
2. 
```bash
curl http://localhost:8983/solr/admin/configs?action=DELETE\&amp;name=big\_search
```

摘自：[https://cwiki.apache.org/confluence/display/solr/Collections+API](https://cwiki.apache.org/confluence/display/solr/Collections+API)

solrCloud提供了collection管理的url命令。

## 4.7 SolrCloud提供Client API

Solr提供了多种Client API，摘自：[https://cwiki.apache.org/confluence/display/solr/Client+APIs](https://cwiki.apache.org/confluence/display/solr/Client+APIs)

{% asset_img 15.png 错误 %}
有人推荐：spring-data-solr

## 4.8 SolrJ-5.4.0 客户端库

 摘自：[https://cwiki.apache.org/confluence/display/solr/Using+SolrJ](https://cwiki.apache.org/confluence/display/solr/Using+SolrJ)

maven配置

| \&lt;dependency\&gt;  \&lt;groupId\&gt;org.apache.solr\&lt;/groupId\&gt;  \&lt;artifactId\&gt;solr-solrj\&lt;/artifactId\&gt;  \&lt;version\&gt;x.y.z\&lt;/version\&gt;\&lt;/dependency\&gt; |
| --- |

在solr安装文件夹中自带了solrj库的包，如下图：
{% asset_img 16.png 错误 %}
### 4.8.1 SolrJ连接SolrCloud进行查询

 逻辑：SolrCloud部署三个节点上（node1，node2，node3），SolrJ通过zookeeper集群选举出SolrCloud中的一个leader节点，然后向该leader节点查询关键字。

```java
 **public**** class **SolrCloudTest {       ** private **CloudSolrClient cloudSolrServer =** null **;       ** public**CloudSolrClient getCloudSolrServer(**final**String zkHost) {                cloudSolrServer =**new**CloudSolrClient(zkHost);               **return **cloudSolrServer;        }        // 根据关键字查询索引结果       ** public ****void** search(CloudSolrClient solrServer, String String) {                SolrQuery query = **new** SolrQuery();                query.setQuery(String);                **try** {                        QueryResponse response = solrServer.query(query);                        SolrDocumentList docs = response.getResults();                         System._out_.println(&quot;文档个数：&quot; + docs.getNumFound());                        System._out_.println(&quot;查询时间：&quot; + response.getQTime());                         **for** (SolrDocument doc : docs) {                                /\*\*                                \* 每个doc的结构 \&lt;str name=&quot;id&quot;\&gt;solrtest1\_3\&lt;/str\&gt;                                \* \&lt;str name=&quot;detail&quot;\&gt;李四睡觉\&lt;/str\&gt;                                \* \&lt;str name=&quot;tabletag&quot;\&gt;solrtest1\&lt;/str\&gt;                                \* \&lt;str name=&quot;position&quot;\&gt;25.09,116.2\&lt;/str\&gt;                                \* \&lt;long name=&quot;\_version\_&quot;\&gt;1528509412933632000\&lt;/long\&gt;                                \*/                                String id = (String) doc.getFieldValue(&quot;id&quot;);                                String detail = (String) doc.getFieldValue(&quot;detail&quot;);                                String tabletag = (String) doc.getFieldValue(&quot;tabletag&quot;);                                String position = (String) doc.getFieldValue(&quot;position&quot;);                                System._out_.println(&quot;id=&quot; + id + &quot;, detail=&quot; + detail                                                + &quot;, tabletag=&quot; + tabletag + &quot;, position=&quot; + position);                        }                } **catch** (SolrServerException e) {                        e.printStackTrace();                } **catch** (Exception e) {                        System._out_.println(&quot;Unknowned Exception!!!!&quot;);                        e.printStackTrace();                }        }        /\*\*        \* **@param** args        \*/        **public**** static ****void** main(String[] args) {                 String zkHost = &quot;172.16.2.100:2181,172.16.2.101:2181,172.16.2.102:2181&quot;;                **final** String defaultCollection = &quot;example2&quot;;                **final**** int **zkClientTimeout = 20000;               ** final ****int** zkConnectTimeout = 1000;                 **try** {                        SolrCloudTest test = **new** SolrCloudTest();                        CloudSolrClient cloudSolrServer = test.getCloudSolrServer(zkHost);                         System._out_.println(&quot;The Cloud SolrServer Instance has benn created!&quot;);                        cloudSolrServer.setDefaultCollection(defaultCollection);                        cloudSolrServer.setZkClientTimeout(zkClientTimeout);                        cloudSolrServer.setZkConnectTimeout(zkConnectTimeout);                        cloudSolrServer.connect();                        System._out_.println(&quot;The cloud Server has been connected !!!!&quot;);                        test.search(cloudSolrServer, &quot;\*:\*&quot;);                        cloudSolrServer.shutdown();                        System._out_.println(&quot;测试结束&quot;);                } **catch** (Exception e) {                        e.printStackTrace();                }        }}
```
| --- |

**查询结果:**

| 文档个数：2查询时间：70id=solrtest1\_3, detail=李四睡觉, tabletag=solrtest1, position=25.09,116.2id=solrtest3\_4, detail=赵六, tabletag=solrtest3, position=25.1,116.11测试结束 |
| --- |

### 4.8.2 SolrJ连接SolrCloud创建索引

{% asset_img 17.png 错误 %}
1.
# 5.异常

## 5.1 This IndexSchema is not mutable

{% asset_img 18.png 错误 %}
{% asset_img 19.png 错误 %}

摘自：[http://stackoverflow.com/questions/31719955/solr-error-this-indexschema-is-not-mutable](http://stackoverflow.com/questions/31719955/solr-error-this-indexschema-is-not-mutable)

在  **solrConfig.xml中删除如下配置：原因是不允许手动添加ｆｉｅｌｄ**

| \&lt;!--     \&lt;processor class=&quot;solr.AddSchemaFieldsUpdateProcessorFactory&quot;\&gt;      \&lt;str name=&quot;defaultFieldType&quot;\&gt;strings\&lt;/str\&gt;      \&lt;lst name=&quot;typeMapping&quot;\&gt;        \&lt;str name=&quot;valueClass&quot;\&gt;java.lang.Boolean\&lt;/str\&gt;        \&lt;str name=&quot;fieldType&quot;\&gt;booleans\&lt;/str\&gt;      \&lt;/lst\&gt;      \&lt;lst name=&quot;typeMapping&quot;\&gt;        \&lt;str name=&quot;valueClass&quot;\&gt;java.util.Date\&lt;/str\&gt;        \&lt;str name=&quot;fieldType&quot;\&gt;tdates\&lt;/str\&gt;      \&lt;/lst\&gt;      \&lt;lst name=&quot;typeMapping&quot;\&gt;        \&lt;str name=&quot;valueClass&quot;\&gt;java.lang.Long\&lt;/str\&gt;        \&lt;str name=&quot;valueClass&quot;\&gt;java.lang.Integer\&lt;/str\&gt;        \&lt;str name=&quot;fieldType&quot;\&gt;tlongs\&lt;/str\&gt;      \&lt;/lst\&gt;      \&lt;lst name=&quot;typeMapping&quot;\&gt;        \&lt;str name=&quot;valueClass&quot;\&gt;java.lang.Number\&lt;/str\&gt;        \&lt;str name=&quot;fieldType&quot;\&gt;tdoubles\&lt;/str\&gt;      \&lt;/lst\&gt;    \&lt;/processor\&gt;     --\&gt; |
| --- |

## 5.2 PeerSync: core=example\_shard3\_replica2 url=http://192.168.232.129:8983/solr ERROR,​ update log not in ACTIVE or REPLAY state. FSUpdateLog{state=BUFFERING,​ tlog=null}

{% asset_img 20.png 错误 %}

## 5.3 配置文件缺失admin-extra.menu-bottom.html、admin-extra.menu-top.html、admin-extra.html

{% asset_img 21.png 错误 %}
解决方法：在conf文件夹中创建以上三个空文件即可，这些文件是Solr admin管理页面需要的。摘自：[http://stackoverflow.com/questions/26597813/how-can-i-get-rid-of-can-not-find-admin-extra-html-error](http://stackoverflow.com/questions/26597813/how-can-i-get-rid-of-can-not-find-admin-extra-html-error)

## 5.4 bad request

{% asset_img 22.png 错误 %}
1.
# 6. 单机SolrCloud部署流程【成功了】

## 6.1环境

Solr5.4升级到新版本Solr5.5

内嵌Zookeeper

借助内嵌ZK实现单机上部署多节点SolrCloud。

## 6.2配置文件

在configsets\data\_driven\_schema\_configs\conf文件基础上添加DIH的配置。

新建文件夹

{% asset_img 24.png 错误 %}
## 6.3 运行命令

注意不需要执行./bin/solr start –c，启动SolrCloud就可以直接执行如下步骤。

{% asset_img 25.png 步骤 %}
{% asset_img 26.png 步骤 %}
{% asset_img 27.png 步骤 %}
{% asset_img 28.png 步骤 %}
## 6.4 验证

{% asset_img 29.png 步骤 %}
导入：

{% asset_img 30.png 步骤 %}
查询：

{% asset_img 31.png 步骤 %}
运行日志没有任何错误；

{% asset_img 32.png 步骤 %}
## 6.5 重启服务

由于 上面创建的是范例程序，所以启动命令如下：
```bash
[root@node3 solr-5.5.0]#  ./bin/solr start -e cloud -noprompt
```

{% asset_img 33.png 步骤 %}
1.
# 7. 多机SolrCloud部署流程[成功]

## 7.1 环境

版本Solr5.5

三节点Solr+三节点Zookeeper，搭建SolrCloud。

 为了不影响node3上已经成功的单节点SolrCloud，故保留源文件夹/usr/local/src/solr-5.5.0。

 多节点部署的文件夹使用/usr/local/src/solr-5.5.0-2。

 必须jdk1.7以上

## 7.2 配置文件

在configsets\data\_driven\_schema\_configs\conf文件基础上添加DIH的配置。

在如下位置新建文件夹

{% asset_img 34.png 步骤 %}
## 7.2 jar包

【1】将solr-dataimporthandler-5.5.0.jar和solr-dataimporthandler-extras-5.5.0.jar从solr/dist/文件夹下copy到solr/server/solr-webapp/webapp/WEB-INF/lib当中，此java包是导入数据用的。

【2】从mysql官网中下载一个mysql-connector-java-5.1.35.zip压缩包，解压出一个mysql-connector-java-5.1.35-bin.jar包，将它copy到solr/server/lib下。

## 6.3 启动ZK集群

## 6.4 运行命令

启动solrcloud
```bash
[root@node1 solr]# ./bin/solr start -c -z node1:2181,node2:2181,node3:2181

[root@node2 solr]# ./bin/solr start -c -z node1:2181,node2:2181,node3:2181

[root@node3 solr]# ./bin/solr start -c -z node1:2181,node2:2181,node3:2181
```
{% asset_img 35.png 步骤 %}
在node1节点执行如下命令：

| 当创建collection出现异常时执行如下命令： 
```bash
./server/scripts/cloud-scripts/zkcli.sh -cmd upconfig -confdir ./server/solr/configsets/fj\_collection/conf -confname myconf -z node1:2181,node2:2181,node3:2181 ./server/scripts/cloud-scripts/zkcli.sh -cmd linkconfig -collection gettingstarted -confname myconf -z node1:2181,node2:2181,node3:2181 
```
创建名称为gettingstarted2 的collection
```bash
./bin/solr create\_collection -c gettingstarted2 -d ./server/solr/configsets/fj\_collection/conf/ -shards 3 -replicationFactor 2 
```
创建名称为bjtcda的collection
```bash
./bin/solr create\_collection -c bjtcda-d ./server/solr/configsets/fj\_collection/conf/ -shards 3 -replicationFactor 2 |
```
| --- |

导入数据

{% asset_img 36.png 步骤 %}
验证

{% asset_img 37.png 步骤 %}
查询

{% asset_img 38.png 步骤 %}
后台不报错

{% asset_img 39.png 步骤 %}
```bash
./bin/solr create\_collection -c gettingstarted3 -d ./server/solr/configsets/fj\_collection/conf/ -shards 3 -replicationFactor 2
```