# 2020-06 考试真题

### 1,sharding awarness，保持cluster健康

### 2,先自定义分词，再reIndex，保证kings和king's评分一样

### 3,先netsted字段定义，再reIndex， 最后netsted查询

### 4,multi_match和most——filed

### 5,RBAC

### 6,update_by_query, 多个字段concat

### 7,先date_histogram, 再max depth

### 8,生成快照

### 9,dynamic mapping

### 10,query、highlight、 sort

来源：死磕Elasticsearch 知识星球 https://t.zsxq.com/Z7yZBEM

# 2019-11月考试真题

考察点主要包括：

### 1、集群的安装和配置：基本安装，配置，安全配置，角色和用户管理等
### 2、索引数据：索引和文档的各种操作
### 3、查询：各种查询场景
### 4、聚合：各种聚合查询，如指标聚合，分桶聚合，嵌套聚合，pipeline聚合等
### 5、Mapping和Analzysis: 索引mapping和分词相关操作
### 6、集群管理： shard分配，集群健康诊断，备份与恢复，冷热分离，跨集群检索等。

来源：https://elasticsearch.cn/article/13530

# 2018 真题
### 1、给一个状态是red的集群，要求不损失数据的前提下，让集群变green。
### 2、有一个文档，内容类似dog & cat， 要求索引这条文档，并且使用match_phrase query，查询dog & cat或者dog and cat都能match。
### 3、有index_a包含一些文档， 要求创建索引index_b，通过reindex api将index_a的文档索引到index_b。 

要求增加一个整形字段，value是index_a的field_x的字符长度； 再增加一个数组类型的字段，value是field_y的词集合。

(field_y是空格分割的一组词，比方"foo bar"，索引到index_b后，要求变成["foo", "bar"]。

### 4、按照要求创建一个index template，并且通过bulk api索引一些文档，达到自动创建索引的效果。 

创建的索引的settings和mappings应该符合要求。

### 5、按要求写一个查询， 其中一个条件是某个关键词必须包含在4个字段中至少2个。

### 6、按照要求写一个search template

### 7、多层嵌套聚合，其中还包括bucket过滤

### 8、给定一个json文档，要求创建一个索引，定义一个nested field，将json文档索引成嵌套类型，同时完成指定的嵌套查询和排序。

### 9、给定两个集群，都包含有某个索引。 要求配置cross cluster search，能够从其中一个集群执行跨集群搜索，写出搜索的url和query body。

### 10、有一个3结点集群，还有一个kibana。 es集群没有安装x-pack，但是安装包已经放在了机器上，kibana有安装x-pack，并且启用了security，

所以此时还连接不到集群。 要求给3个结点配置security，给内置的几个用户分别设定指定的密码。 

之后添加指定的新用户，指定的role，并给用户赋予role a, role b。

来源：https://elasticsearch.cn/article/6133
