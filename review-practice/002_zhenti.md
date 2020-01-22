# 1、集群部署篇
## 1.1 部署 3 节点的集群，需要同时满足以下要求
 
- 集群名为“geektime” 
- 将每个节点的名字设为和机器名一样，分别为 node1，node2，node3 
- node1 配置成dedicated master-eligable节点
- node2和node3配置成 ingest 和 data node
- 设置 jvm 为1g

## 1.2 配置 3节点的集群，加上一个 Kibana 的实例，设定以下安全防护
- 为集群配置 basic authentication 
- 将 Kibana 连接到 Elasticsearch
- 创建一个名为 geektime 的用户
- 创建一个名为 orders 的索引
- geektime 用户只能读取和写入 oders 的索引，不能删除及修改 orders

## 1.3 配置 3节点的集群，同时满足以下要求
- 确保索引 A 的分片全部落在在节点1
- 索引 B 分片全部落在 节点 2和3 
- 不允许删除数据的情况下，保证集群状态为 Green

# 2、索引数据篇
## 2.1 为一个索引，按要求设置以下 dynamic Mapping
- 一切 text 类型的字段，类型全部映射成 keyword
- 一切以 int_开头命名的字段，类型都设置成 integer

## 2.2 设置一个Index Template，符合以下的要求
- 为 log 和log- 开头的索引。创建 3 个主分片，1 个副本分片
- 同时为索引创建一个相应的 alias
- 使用 bulk API，写入多条电影数据
## 2.3 为 movies index 设定一个 Index Alias，默认查询只返回评分大于3的电影

## 2.4 给一个索引 A，要求创建索引 B，通过 Reindex API，将索引 A 中的文档写入索引 B，同时满足以下要求
- 增加一个整形字段，将索引 A中的一个字段的字符串长度，计算后写入
- 将 A 文档中的字符串以“；”分隔后，写入索引B中的数组字段中

## 2.5 定义一个 Pipeline，并且将 eathquakes 索引的文档进行更新
- pipeline的 ID 为 eathquakes_pipeline
- 将 magnitude_type 的字段值改为大写
- 如果文档不包含 “batch_number”, 增加这个字段，将数值设置为 1
- 如果已经包含 batch_number, 字段值➕1 

## 2.6 为索引中的文档增加一个新的字段，字段值为 现有字段1+现有字段2+现有字段3

# 3、查询篇
## 3.1 写一个查询，要求某个关键字在文档的 4 个字段中至少包含两个以上
 
- bool 查询，should / minimum_should_match

## 3.2 按照要求写一个 search template
- 写入 search template
- 根据 search template 写出相应的 query

## 3.3 对一个文档的多个字段进行查询，要求最终的算分是几个字段上算分的总和，同时要求对特定字段设置 boosting 值

## 3.4 针对一个索引进行查询，当索引的文档中存在对象数组时，会搜索到了不期望的数据。需要重新定义 mapping，并提供改写后的 query 语句
 
- Nested Object

# 4、聚合篇
## 4.1 earthquakes索引中包含了过去11个月的地震信息，请通过一句查询，获取以下信息
 
- 过去11个月，每个月的平均 地震等级（magiitude）
- 过去11个月里，平均地震等级最高的一个月及其平均地震等级
- 搜索不能返回任何文档

## 4.2 Query Fileter Bucket Filter
 
## 4.3 Pipeline Aggregation -> Bucket Filter

# 5、映射与分词篇
 
## 5.1 一篇文档，字段内容包括了 “hello & world”，索引后，要求使用 match_phrase query, 
查询 hello & world 或者 hello and world 都能匹配

## 5.2 reindex 索引，同时确保给定的两个查询，都能搜索到相关的文档，并且文档的算分是一样的
 
- match 查询，分别查 “smith's” ，“smiths”
 
- 在不改变字段的属性，将数据索引到新的索引上
 
- 确保两个查询有一致的搜索结果和算分

# 6 集群管理篇
## 6.1 安装并配置 一个 hot & warm 架构的集群
- 三个节点， node 1 为 hot ， node2 为 warm，node 3 为cold
- 三个节点均为 master-eligable 节点
- 新创建的索引，数据写入 hot 节点
- 通过一条命令，将数据从 hot 节点移动到 warm 节点

## 6.2 为两个集群配置跨集群搜索
 
- 两个集群都有 movies 的索引
- 创建跨集群搜索
- 创建一条查询，能够同时查到两个集群上的 movies 数据

## 6.3 解决集群变红或者变黄的问题
- 技能1：通过 explain API 查看
- 技能2：shard filtering API，查看 include
- 技能3： 更新一下 routing，确认 replica 可以分配（include 更加多的 rack）
- 解决方案
 （1）为集群配置延迟分配和一个节点上最多几个分片的配置
 （2）设置 Replica 为 0
 （3）删除 dangling index
 （4）使用了错误的 routing node attribute
 
## 6.4 备份一个集群中指定的几个索引
