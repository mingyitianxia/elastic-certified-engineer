# Elastic 认证工程师考试

[官方页面](https://www.elastic.co/cn/training/elastic-certified-engineer-exam)

[官方QA页面](https://www.elastic.co/training/certification/faq)

---

## 考试环境

FAQ页面：[链接](https://www.elastic.co/cn/training/certification/faq)

#### Which version of Elasticsearch is the exam using?

Elastic Certified Engineer: __7.2__

Elastic Certified Analyst: __7.6__

Elastic Certified Observability Engineer: coming soon!

---

## 考试考纲及翻译

[考试介绍页面](https://www.elastic.co/cn/training/elastic-certified-engineer-exam)

### 1. Installation and Configuration

安装和配置：

1. Deploy and start an Elasticsearch cluster that satisfies a given set of requirement.
    1. 按照题目要求安装和配置一个ES集群。
1. Configure the nodes of a cluster to satisfy a given set of requirements.
    1. 按照要求配置一个节点。
1. Secure a cluster using Elasticsearch Security.
    1. 通过`Elasticsearch Security`给ES集群配置安全措施。
1. Define role-based access control using Elasticsearch Security.
    1. 通过`Elasticsearch Security`给ES集群添加按role访问的设置。

### 2. Indexing Data

索引数据：

1. Define an index that satisfies a given set of requirements.
    1. 按照题目要求设置一个索引。
1. Perform index, create, read, update, and delete operations on the documents of an index.
    1. 在索引中对文档进行索引、创建、读取、更新、删除等操作。
1. Define and use index aliases.
    1. 设置和使用索引别名。
1. Define and use an index template for a given pattern that satisfies a given set of requirements.
    1. 按照题目要求设置索引模板（template），以满足索引匹配要去（pattern）和索引设置等要求。
1. Define and use a dynamic template that satisfies a given set of requirements.
    1. 按照要求设置和使用动态模板。
1. Use the Reindex API and Update By Query API to reindex and/or update documents.
    1. 通过`reindex`和`update_by_query`等API对文档进行 reindex和/或更新。
1. Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents.
    1. 根据要求设置一个ingest pipeline（数据处理管道），并且包含通过 painless 进行文档修改的功能。

### 3. Queries

搜索：

1. Write and execute a search query for terms and/or phrases in one or more fields of an index.
    1. 编辑执行一个query可以通过terms 和/或 phrases 来匹配索引中文档的1..N个字段。
1. Write and execute a search query that is a Boolean combination of multiple queries and filters.
    1.编辑执行一个包含逻辑组合的多条件query/filter 搜索命令。
1. Highlight the search terms in the response of a query.
    1. 对搜索结果中的字段内容进行高亮。
1. Sort the results of a query by a given set of requirements.
    1. 基于给定要求，对搜索结果进行排序。
1. Implement pagination of the results of a search query.
    1. 实现搜索结果的分页需求。
1. Apply fuzzy matching to a query.
    1. 实现一些模糊匹配的查询。
1. Define and use a search template.
    1. 定义和使用搜索模板。
1. Write and execute a query that searches across multiple clusters.
    1. 编辑执行跨集群的搜索命令。

### 4. Aggregations

聚合：

1. Write and execute metric and bucket aggregations.
    1. 编辑执行一些矩阵/桶聚合。
1. Write and execute aggregations that contain sub-aggregations.
    1. 编辑执行包含子聚合的命令。

### 5. Mappings and Text Analysis

索引和文本分析

1. Define a mapping that satisfies a given set of requirements.
    1. 根据要求定义一个mapping（字段索引schema）。
1. Define and use a custom analyzer that satisfies a given set of requirements.
    1. 根据要求设置一个自定义的分词器。
1. Define and use multi-fields with different data types and/or analyzers.
    1. 使用多索引字段（multi-fields）来存储和处理不同种类的数据。
1. Configure an index so that it properly maintains the relationships of nested arrays of objects.
    1. 设置一个可以维护嵌套数据关系的索引（nested arrays of objects）

### 6. Cluster Administration

集群管理

1. Allocate the shards of an index to specific nodes based on a given set of requirements.
    1. 根据要求将索引的分片放置在指定位置。
1. Configure shard allocation awareness and forced awareness for an index.
    1. 配置和强制分片索引分片的位置。
1. Diagnose shard issues and repair a cluster's health.
    1. 分析和修复一些分片异常。
1. Backup and restore a cluster and/or specific indices.
    1. 备份和恢复集群的某些/全部的索引。
1. Configure a cluster for use with a hot/warm architecture.
    1. 配置一个基于冷热架构的集群。
1. Configure a cluster for cross cluster search.
    1. 配置一个可以支持跨集群搜索的集群。

---

## 难点分析

1. 相对比较有难度的可能会是自定分词器的编写，pipeline + painless 脚本的编写和使用，这部分可能需要多加练习。
1. 另一个相对较少使用的是集群安全的部分，包括认证、权限管理、跨集群访问等，这部分通过docker和/或多节点配合kibana进行练习就好，总体难度没有1. 中的部分大。
1. index template、search template和平时使用的index、search语句类似，但是属性并不完全一样，答题的时候需要小心因为书写习惯带来的疏忽。
1. 搜索和聚合的语句部分在考试kibana中可能会有输入提示，但是建议能够在不进行自动补全的状态下完成。
1. 牢记官方文档中对应章节的目录树位置，尽快找到对应章节，复制、黏贴、修改样例命令是考试成功的一大助力。
