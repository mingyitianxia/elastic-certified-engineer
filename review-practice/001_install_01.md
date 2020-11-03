## INSTALLATION AND CONFIGURATION

集群安装和配置

## GOAL: Setup an Elasticsearch cluster that satisfies a given set of requirements

目标： 根据给定需求配置一个ES集群。过程略，[官方链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/install-elasticsearch.html)

## REQUIRED SETUP

### 第0题，按要求配置集群

1. Download the exam version of Elasticsearch
    1. 下载考试版本的ES（7.2 [官方页面](https://www.elastic.co/cn/training/elastic-certified-engineer-exam)）
1. Deploy the cluster `eoc-01-cluster`, so that it satisfies the following requirements:
    1. 部署一个叫`eoc-01-cluster`的集群，下面是要求：
        1. has three nodes, named `node1`, `node2`, and `node3`
            1. 有三个节点，分别叫`node1`，`node2`和`node3`
        1. all nodes are eligible master nodes
            1. 所有三个节点都有成为master节点的资格
1. 配置文件:
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/settings.html)
    1. 页面路径：Set up Elasticsearch =》 Configuring Elasticsearch


### 第0题题解

修改配置文件：`$ES_HOME/config/elasticsearch.yml`

Node1
```yml
cluster.name: eoc-01-cluster

node.name: node1
node.master: true
```

Node2
```yml
cluster.name: eoc-01-cluster

node.name: node2
node.master: true
```

Node3
```yml
cluster.name: eoc-01-cluster

node.name: node3
node.master: true
```

### 第0题题解说明

1. 创建一个叫`eoc-01-cluster`的集群
    1. 对应`$ES_HOME/config/elasticsearch.yml`文件中的`cluster.name`参数，同网段的ES节点会尝试寻找和自己`cluster.name`相同的节点组成集群。
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster.name.html)
    1. 页面路径：Set up Elasticsearch =》Important Elasticsearch configuration =》 cluster.name

1. 包含三个节点，名字叫`node1`，`node2`和`node3`
    1. 对应`$ES_HOME/config/elasticsearch.yml`文件中的`node.name`参数，这个参数会成为当前节点的标记，不设ES会尝试使用当前节点的hostname，它和nodeId（ES自行计算）都可以作为节点唯一标识，所以在同一个集群之中，`node.name`是唯一的。
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/node.name.html)
    1. 页面路径：Set up Elasticsearch =》Important Elasticsearch configuration =》 node.name

1. 每个节点都有成为master节点的资格
    1. 对应`$ES_HOME/config/elasticsearch.yml`文件中的`node.master`参数，这个参数标记了当前节点是否有资格成为master节点，为`true`代表了它有资格成为master节点，但是否真的能成为master节点还需要进行集群内投票的过程；为`false`代表该节点没有资格成为master节点，只有投票选举没有被选举的资格。
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html#master-node)
    1. 页面路径：Modules =》 Node

### 第0题题解延伸

1. 与`node.master`同级的还有以下一些常用参数
    1. `node.data` 标记了当前节点是否有资格成为data节点，只有data节点才会存储数据。
    1. `node.ingest` 标记了当前节点是否需要处理数据预处理任务，后面的章节中的`pipeline`等功能就是通过ingest节点来处理的。
    1. `node.ml` 和 `xpack.ml.enabled` 标记了该节点是否能承载machine learning的任务，目前ml的功能还是收费的。
    1. `cluster.remote.connect` 标记了节点是否能支持跨集群搜索，在后续的章节中会有有关跨集群搜索的配置和使用的部分。

### 第1题，按要求配置集群

1. Bind `node1` to the IP address “151.101.2.217” and port “9201”
    1. 给`node1`指定ip地址“151.101.2.217”，端口“9201”
1. Bind `node2` to the IP address “151.101.2.218” and port “9202”
    1. 给`node2`指定ip地址“151.101.2.218”，端口“9202”
1. Bind `node3` to the IP address “151.101.2.219” and port “9203”
    1. 给`node3`指定ip地址“151.101.2.219”，端口“9203”
1. Configure the cluster discovery module of `node2` and `node3` so as to use `node1` as seed host
    1. 修改`node2`和`node3`的集群发现配置，使得它们以`node1`为种子master
1. Configure the nodes to avoid the split brain scenario
    1. 调整节点配置，防止脑裂的发生

### 第1题题解

修改配置文件：`$ES_HOME/config/elasticsearch.yml`

Node1
```yml
node.name: node1
node.master: true

http.host: 151.101.2.217
http.port: 9201

initial_master_nodes: ["node1"]
```

Node2
```yml
node.name: node2

http.host: 151.101.2.218
http.port: 9202

discovery.seed_host: ["151.101.2.217:9201","151.101.2.218:9202","151.101.2.219:9203"]

initial_master_nodes: ["node1"]
```

Node3
```yml
node.name: node3

http.host: 151.101.2.219
http.port: 9203

discovery.seed_host: ["151.101.2.217:9201","151.101.2.218:9202","151.101.2.219:9203"]

initial_master_nodes: ["node1"]
```

### 第1题题解说明

1. 给`node`指定ip地址和端口
    1. 这个配置主要和`http.host`和`http.port`相关，当然，一般我们常用的可能是`network.host`，用来绑定ES节点的监听ip。同时，ES还存在分别针对http和tcp不同的监听端口`http.port`和`transport.port`。
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-network.html)
    1. 页面路径：Modules =》 Network Settings

1. 修改监听配置，让`node2`和`node3`以`node1`为seed host
    1. 这个参数在7.x之前是`discovery.zen.xxxx`但在考试版本（7.2）中已经换成了`discovery.seed_hosts`了。所以这里涉及到的会是`discovery.seed_hosts`和`cluster.initial_master_nodes`这俩配置项。
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/discovery-settings.html)
    1. 页面路径：Set up Elasticsearch =》 Important Elasticsearch configuration =》 Discovery and cluster formation settings

1. 调整节点配置，防止脑裂的发生。
    1. 其实在第0题中，我们曾尝试把三个节点都作为master节点的候选人，在7.x的启动过程中，如果三个节点中某个节点暂时离开网络，那很有可能会自成集群造成脑裂，所以这里需要一个配置让集群在初始化启动，还没正式选出master节点的时候以某一/几个节点优先成为master节点。这里涉及到的就是 ⬆️ 里面提到过的`cluster.initial_master_nodes`配置项。
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/discovery-settings.html)
    1. 页面路径：Set up Elasticsearch =》 Important Elasticsearch configuration =》 Discovery and cluster formation settings

### 第2题，按要求调整节点配置

1. Configure `node1` to be a data node but not an ingest node
    1. 调整`node1`的配置，让它可以成为一个data节点但是不能做ingest节点
1. Configure `node2` and `node3` to be both an ingest and data node
    1. 调整`node2`和`node3`的配置，让它们既可以成为一个data节点也能做ingest节点

### 第2题题解

修改配置文件：`$ES_HOME/config/elasticsearch.yml`

Node1
```yml
node.name: node1
node.master: false
node.data: true
node.ingest: false
```

Node2
```yml
node.name: node2
node.master: false
node.data: true
node.ingest: true
```

Node3
```yml
node.name: node3
node.master: false
node.data: true
node.ingest: true
```

### 第2题题解说明

1. 调整`node1`的配置，让它可以成为一个data节点但是不能做ingest节点
1. 调整`node2`和`node3`的配置，让它们既可以成为一个data节点也能做ingest节点
    1. 这两个主要解决的事节点的身份，分别对应的是`node.master`，`node.data`，`node.ingest`等属性。
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html)
    1. 页面路径：Modules =》 Node

### 第3题，禁用磁盘交换区

1. Configure `node1` to disallow swapping on its host
    1. 配置`node1`节点，禁止磁盘交换区

### 第3题题解

在命令行中运行`sudo swapoff -a`

### 第3题题解说明

* 配置`node1`节点，禁止磁盘交换区
    1. 。操作系统会尝试通过磁盘交换区的来代替部分内存的使用，但是我们知道磁盘的读写速度比内存慢好几个量级，这样会严重影响到ES这种频繁进行数据读写的服务的使用效率，所以各类文档都会建议关闭磁盘交换区的使用。
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/setup-configuration-memory.html)
    1. 页面路径：Set up Elasticsearch =》 Important System Configuration =》 Disable swapping

### 第4题，调整JVM参数

1. Configure the JVM settings of each node so that it uses a minimum and maximum of 8 GB for the heap
    1. 在每个节点上调整JVM参数，把最大、最小堆栈数都设为8G

### 第4题题解

修改配置文件：`$ES_HOME/config/jvm.options`

```bash
-Xms8g
-Xmx8g
```

### 第4题题解说明

1. 在每个节点上调整JVM参数，把最大、最小堆栈数都设为8G
    1. ES是一个Java项目，所以对于Java JVM的参数都会对它生效，这里已经明确的指出要调整JVM中的最大最小堆栈大小。我们会发现，这里的最大最小堆栈的值都是8G，其实在现实配置中建议：
        1. 最大最小堆栈数相等，这样JVM就不需要在运行过程中重新申请/回收内存来
        1. 最大最小堆栈数设为Math.min(当前可用内存的一半， 32G)，设为可用内存的一半是为了在运行ES实例自己的同时，还能留一部分内存给lucene和其他的对外内存（比如文件缓存），32G是因为ES的主键ID默认是Long型，超过32G之后存储位数会变多，浪费更多的空间存储同样数目的数据。
    1. [参考链接1](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/heap-dump-path.html)，[参考链接2](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/jvm-options.html)
    1. 页面路径1：Set up Elasticsearch =》 Important Elasticsearch configuration =》JVM heap dump path
    1. 页面路径2：Set up Elasticsearch =》 Configuring Elasticsearch =》 Setting JVM options

### 第5题，调整日志配置

1. Configure the logging settings of each node so that
    1. (i)  the logs directory is not the default one,
    1. (ii) the log level for transport-related events is "debug"
    1. 在所有节点上调整日志配置：
        1. 调整日志路径，取代默认值
        1. 调整ES中transport相关的日志级别为“debug”

### 第5题题解

修改配置文件：`$ES_HOME/config/elasticsearch.yml`

```yml
# 把这个参数后面路径改成要求的路径
path.logs: /path/to/logs
```

在kibana/head等UI工具中执行以下命令：

```bash
PUT /_cluster/settings
{
  "transient": {
    "logger.org.elasticsearch.transport": "debug"
  }
}
```

在命令行中直接执行以下命令：

```bash
curl -X POST 'http://localhost:9200/_cluster/settings' \
-H 'Content-Type: application/json' \
-d '{
  "transient": {
    "logger.org.elasticsearch.transport": "debug"
  }
}'
```

### 第5题题解说明

1. 在所有节点上调整日志配置：
    1. 调整日志路径，取代默认值
    1. 调整ES中transport相关的日志级别为“debug”

* 这里涉及两部分：
    1. 日志路径的部分会伴随ES节点的启动而生效，同时对它的修改也需要重启节点生效，所以会存在于`$ES_HOME/config/elasticsearch.yml`里面
    1. 日志等级的调整有两种方式：
        1. 调整`$ES_HOME/config/log4j2.properties`文件中的配置，这个配置和其他Java项目中的log4j2是一样的。
        1. 通过集群设置的接口来调整，方式如 ⬆️
    1. [参考链接1](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/path-settings.html)，[参考链接2](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/logging.html)
    1. 页面路径1：Set up Elasticsearch =》 Important Elasticsearch configuration =》path.data and path.logs
    1. Set up Elasticsearch =》 Configuring Elasticsearch =》 Logging configuration
    1. 在命令里存在一个关键字`transient`，代表这个配置对于当前节点是临时生效的，在节点重启之后会失效，与之相对的是`persistent`永久生效，这俩关键字存在于很多配置api中

### 第6题，调整节点配置

1. Configure the nodes so as to disable the possibility to delete indices using wildcards
    1. 调整所有节点的配置，禁止通过通配符删除索引

### 第6题题解

两种方式：

1. 修改配置文件：`$ES_HOME/config/elasticsearch.yml`，添加`action.destructive_requires_name: true`
1. 通过集群配置接口进行设置：

```bash
PUT /_cluster/settings
{
    "persistent" : {
       "action.destructive_requires_name":true
    }
}
```

```bash
curl -X PUT 'http://localhost:9200/_cluster/settings' \
-H 'Content-Type: application/json' \
-d '{
  "persistent" : {
       "action.destructive_requires_name":true
  }
}'
```

### 第6题题解说明

1. 调整所有节点的配置，禁止通过通配符删除索引
    1. 这个题目怪怪的，不知道为什么要放这里，可能是想考察集群设置更新的操作吧。这里涉及到的配置是`action.destructive_requires_name`。
    1. [参考链接1](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-delete-index.html)，[参考链接2](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-update-settings.html)
    1. 页面路径1：Indices APIs =》 Delete Index
    1. 页面路径2：Cluster APIs =》 Cluster Update Settings

### 第6题题解延伸

* 既然考察的可能是集群配置更新，那也有可能会根据文档内容改编其他一些配置项的更新：
    1. 更新索引每秒恢复最大速度为20mb
        1. `"indices.recovery.max_bytes_per_sec" : "20mb"`
    1. 移除索引恢复速度的配置
        1. `"indices.recovery.max_bytes_per_sec" : null`
    1. 移除所有索引恢复相关的配置
        1. `"indices.recovery.*" : null`
    1. 临时更新xx配置，通过`transient`关键字

        1. ```bash
            PUT /_cluster/settings
            {
                "transient": {
                    "indices.recovery.*" : null
                }
            }
            ```

    1. 永久更新xx配置，通过`persistent`关键字

        1. ```bash

            PUT /_cluster/settings
            {
                "persistent" : {
                    "indices.recovery.max_bytes_per_sec" : "50mb"
                }
            }
            ```

    1. 获取集群配置信息

        1. ```bash
            GET /_cluster/settings
            ```
