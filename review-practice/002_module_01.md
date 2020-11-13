## LUSTER ADMINISTRATION

集群管理

## GOAL: Allocate the shards in a way that satisfies a given set of requirements

目标，根据要求把分配放置在合适的位置

## REQUIRED SETUP: /

前期准备：

1. Download the exam version of Elasticsearch
   1. 下载考试版本的ES包
2. Deploy the cluster `eoc-06-cluster`, with three nodes named `node1`, `node2`, and `node3`
   1. 部署一个名叫`eoc-06-cluster`的集群，集群里有仨节点，分别叫`node1`、`node2`和 `node3`
3. Configure the Zen Discovery module of each node so that they can communicate
   1. 调整节点发现配置使他们能彼此通讯
4. Start the cluster
   1. 启动集群

## 第0题，初始化索引和数据

1. Create the index `hamlet-1` with two primary shards and one replica
   1. 创建一个名叫`hamlet-1`的索引，它需要有2个主分片和1个副本
1. Add some documents to `hamlet-1` by running the command below 
   1. 用下面命令添加一些数据到`hamlet-1`里面
    ```bash
    PUT hamlet-1/_doc/_bulk
    {"index":{"_index":"hamlet-1","_id":0}}  
    {"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
    {"index":{"_index":"hamlet-1","_id":1}} 
    {"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
    {"index":{"_index":"hamlet-1","_id":2}}
    {"line_number":"3","speaker":"BERNARDO","text_entry":"Long live the king!"}
    {"index":{"_index":"hamlet-1","_id":3}}
    {"line_number":"4","speaker":"FRANCISCO","text_entry":"Bernardo?"}
    {"index":{"_index":"hamlet-1","_id":4}}
    {"line_number":"5","speaker":"BERNARDO","text_entry":"He."}
    ```

1. Create the index `hamlet-2` with two primary shard and one replica
   1. 创建一个名叫`hamlet-2`的索引，它需要有2个主分片和1个副本
2. Add some documents to `hamlet-2` by running the command below
   1. 用下面命令添加一些数据到`hamlet-2`里面
    ```bash
    PUT hamlet-2/_doc/_bulk
    {"index":{"_index":"hamlet-2","_id":5}}
    {"line_number":"6","speaker":"FRANCISCO","text_entry":"You come most carefully upon your hour."}
    {"index":{"_index":"hamlet-2","_id":6}}
    {"line_number":"7","speaker":"BERNARDO","text_entry":"Tis now struck twelve; get thee to bed, Francisco."}
    {"index":{"_index":"hamlet-2","_id":7}}
    {"line_number":"8","speaker":"FRANCISCO","text_entry":"For this relief much thanks: tis bitter cold,"}
    {"index":{"_index":"hamlet-2","_id":8}}
    {"line_number":"9","speaker":"FRANCISCO","text_entry":"And I am sick at heart."}
    {"index":{"_index":"hamlet-2","_id":9}}
    {"line_number":"10","speaker":"BERNARDO","text_entry":"Have you had quiet guard?"}
    ```

## 第0题，题解
（由于我是基于我的docker-compose文件启动的集群，所以会和题目不完全一样，比如题目里的node1我用的是esNode1，下同）
1. 创建索引，在kibana里运行下面的命令
   ```bash
    PUT hamlet-1
    {
      "settings": {
        "number_of_shards":2,
        "number_of_replicas": 1
      }
    }
   ```
1. 把上面那些命令直接照抄在kibana里，运行

## 第0题，题解说明
* 这题主要考察索引的建立以及初始化的配置，两段索引建立和数据添加除了索引名字不一样，其他差不多。
  1. 主分片（primary shard）和副本（replica）分别对应着索引配置（_settings）里的`number_of_shards`和`number_of_replicas`
     1. 这里有个坑点在，如果不先指定索引分片/副本数，ES 7.X会默认创建一个1分片1副本点索引，所以如果直接运行bulk命令的话，索引设置就不对了
     2. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-create-index.html)
     3. 页面路径：Indices APIs =》 Create Index
  2. 插入数据的部分没什么特别的，直接照着执行就好，主要需要关注的是第一行写索引信息和索引命令，第二行写数据就好
     1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-bulk.html)
     2. 页面路径：Document APIs =》 Bulk API

## 第1题，检查分片放置情况

1. Check that the replicas of indices `hamlet-1` and `hamlet-2` have been allocated 
   1. 查看`hamlet-1`和`hamlet-2`的分片放置情况
1. Check the distribution of primary shards and replicas of indices `hamlet-1` and `hamlet-2` across the nodes of the cluster
   1. 通过集群中的节点查看`hamlet-1`和`hamlet-2`的分片分布情况
1. 

## 第1题，题解
1. 查看分片放置情况可以通过以下命令 `GET /_cat/allocation?v&h=shards,disk.indices,disk.used,disk.avail,disk.total,disk.percent,host,ip,node` 查看每个节点的资源分配情况
  ```bash
  shards disk.indices disk.used disk.avail disk.total disk.percent host       ip         node
      6      179.8kb    26.1gb     32.2gb     58.4gb           44 172.18.0.2 172.18.0.2 es721Node1
      5       65.1kb    26.1gb     32.2gb     58.4gb           44 172.18.0.5 172.18.0.5 es721Node3
      5      134.8kb    26.1gb     32.2gb     58.4gb           44 172.18.0.4 172.18.0.4 es721Node2
  ```

1. 通过以下命令 `GET /_cat/nodeattrs?v&h=node,id,pid,host,ip,port,attr,value` 查看节点属性
  ```bash
  node       id   pid host       ip         port attr              value
  es721Node2 q0pL 1   172.18.0.4 172.18.0.4 9300 name              es721Node2
  es721Node2 q0pL 1   172.18.0.4 172.18.0.4 9300 xpack.installed   true
  es721Node3 4ym8 1   172.18.0.5 172.18.0.5 9300 name              es721Node3
  es721Node3 4ym8 1   172.18.0.5 172.18.0.5 9300 xpack.installed   true
  es721Node1 PGDN 1   172.18.0.2 172.18.0.2 9300 ml.machine_memory 12564148224
  es721Node1 PGDN 1   172.18.0.2 172.18.0.2 9300 xpack.installed   true
  es721Node1 PGDN 1   172.18.0.2 172.18.0.2 9300 name              es721Node1
  es721Node1 PGDN 1   172.18.0.2 172.18.0.2 9300 ml.max_open_jobs  20
  ```

1. 通过以下命令 `GET /_cat/shards/hamlet-1,hamlet-2?v&h=index,shard,prirep,state,docs,store,ip,node` 查看索引的分配分配情况
  ```bash
  index    shard prirep state   docs store ip         node
  hamlet-1 1     r      STARTED    2 4.8kb 172.18.0.4 es721Node2
  hamlet-1 1     p      STARTED    2 4.8kb 172.18.0.5 es721Node3
  hamlet-1 0     p      STARTED    3 5.2kb 172.18.0.4 es721Node2
  hamlet-1 0     r      STARTED    3 5.2kb 172.18.0.5 es721Node3
  hamlet-2 1     p      STARTED    1 4.8kb 172.18.0.2 es721Node1
  hamlet-2 1     r      STARTED    1 4.8kb 172.18.0.5 es721Node3
  hamlet-2 0     r      STARTED    4 5.7kb 172.18.0.4 es721Node2
  hamlet-2 0     p      STARTED    4 5.7kb 172.18.0.2 es721Node1
  ```

1. （如果有分片无法正常放置）通过以下命令 `GET /_cluster/allocation/explain` 查看分片无法分配的原因

## 第1题，题解说明
* 这题主要考察集群属性，分片分配
  1. 大部分的状态都可以通过 `GET /_cat/${api}` 接口和 `GET /_cluster/${api}` 接口来查看
     1. `GET /_cat/allocation?v` 接口用来查看节点资源分布状况
        1. 可以通过 `?help` 来查看支持属性，通过 `?h=${header}` 来指定需要属性
        2. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cat-allocation.html)
        3. 页面路径：cat APIs =》 cat allocation
     2. `GET /_cat/nodeattrs?v` 接口用来查看节点自身属性
        1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cat-nodeattrs.html)
        2. 页面路径：cat APIs =》 cat nodeattrs
     3. `GET /_cat/shards` 来查看指定/所有索引的分片分配情况
        1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cat-shards.html)
        2. 页面路径：cat APIs =》 cat shards
  2. 当分片存在分配失败的时候，可以通过`GET /_cluster/allocation/explain`查看分片无法分配的原因
     1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-allocation-explain.html)
     2. 页面路径：Cluster APIs =》 Cluster Allocation Explain API

## 第2题，索引级别分片分配设置

1. Configure `hamlet-1` to allocate both primary shards to `node2`, using the node name
   1. 通过配置节点名字，指定`hamlet-1`的俩主分片都放在`node2`上
1. Configure `hamlet-2` so that no primary shard is allocated to `node3`
   1. 通过配置，避免`hamlet-2`的分片放置在`node3`
1. Remove any allocation filter setting associated with `hamlet-1` and `hamlet-2`
   1. 取消所有针对`hamlet-1`和`hamlet-2`分片放置的设置
1. Verify the success of the last action by using the _cat API
   1. 通过`_cat`API验证一下这些操作成功了没

## 第2题，题解

1. 把`hamlet-1`的主分片都放`node2`上
   ```bash
   PUT hamlet-1/_settings
   {
      "index.routing.allocation.include._name": "es721Node2"
   }
   ```

校验命令：`GET /_cat/shards/hamlet-1?v&h=index,shard,prirep,state,docs,store,ip,node`

运行之前：
```bash
index    shard prirep state   docs store ip         node
hamlet-1 1     p      STARTED    2 4.8kb 172.18.0.5 es721Node3
hamlet-1 0     p      STARTED    3 5.2kb 172.18.0.4 es721Node2
```

运行之后：
```bash
index    shard prirep state   docs store ip         node
hamlet-1 1     p      STARTED    2 4.8kb 172.18.0.4 es721Node2
hamlet-1 0     p      STARTED    3 5.2kb 172.18.0.4 es721Node2
```

1. 把`hamlet-2`所有主分片都移出`node3`
```bash
PUT hamlet-2/_settings
{
  "index.routing.allocation.exclude._name": "es721Node3"
}
```

校验命令：
```bash
GET /_cat/shards/hamlet-2?v&h=index,shard,prirep,state,docs,store,ip,node
```

运行之前：
```bash
index    shard prirep state   docs store ip         node
hamlet-2 1     p      STARTED    1 4.8kb 172.18.0.2 es721Node1
hamlet-2 1     r      STARTED    1 4.8kb 172.18.0.5 es721Node3
hamlet-2 0     r      STARTED    4 5.7kb 172.18.0.4 es721Node2
hamlet-2 0     p      STARTED    4 5.7kb 172.18.0.2 es721Node1
```

运行之后：
```bash
index    shard prirep state   docs store ip         node
hamlet-2 1     r      STARTED    1 4.8kb 172.18.0.4 es721Node2
hamlet-2 1     p      STARTED    1 4.8kb 172.18.0.2 es721Node1
hamlet-2 0     r      STARTED    4 5.7kb 172.18.0.4 es721Node2
hamlet-2 0     p      STARTED    4 5.7kb 172.18.0.2 es721Node1
```

1. 移除所有`hamlet-1`和`hamlet-2`的分配放置设置
```bash
PUT hamlet-1,hamlet-2/_settings
{
  "index.routing.allocation.include._name": null,
  "index.routing.allocation.exclude._name": null
}
```

校验命令：
```bash
GET /_cat/shards/hamlet-1,hamlet-2?v&h=index,shard,prirep,state,docs,store,ip,node
```

执行之后：
```bash
index    shard prirep state   docs store ip         node
hamlet-2 1     r      STARTED    1 4.8kb 172.18.0.4 es721Node2
hamlet-2 1     p      STARTED    1 4.8kb 172.18.0.2 es721Node1
hamlet-2 0     r      STARTED    4 5.7kb 172.18.0.4 es721Node2
hamlet-2 0     p      STARTED    4 5.7kb 172.18.0.5 es721Node3
hamlet-1 1     p      STARTED    2 4.8kb 172.18.0.4 es721Node2
hamlet-1 0     p      STARTED    3 5.2kb 172.18.0.2 es721Node1
```

## 第2题，题解说明
* 这题主要考察索引配置中的节点放置配置，会涉及到索引`_settings`里的`index.routing.allocation.include._name`和`index.routing.allocation.exclude._name`俩设置。
  1. 题目里用到的是节点名字（_name）其实也可以通过其他属性进行节点的筛选，比如`_ip` `_host` 等内置的属性，以及`${attr}`的一些外置的属性。
* 这里可能会有个坑点也是考点在于，有些索引的节点放置属性设置之后，会触发ES集群的分片放置限制，造成一些分片（主/副）无法被正常放置，被标记为`UNASSIGNED`
  1. 同一个节点上不可以同时放置某个分片的主分片和副本（如`shard 1`的主分片和副本不可以同时存在于`node1`）
  1. 当 `include`和`exclude`属性有冲突时，集群里所有的节点都没有资格放置分片等
* 本题官方文档：
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/shard-allocation-filtering.html)
  2. 页面路径：Index modules =》 Index Shard Allocation =》 Index-level shard allocation filtering

## 第3题，集群级别分片分配设置

1. Let's assume that we deployed the `eoc-06-cluster` cluster across two availability zones, named `earth` and `mars`. Add the attribute `AZ` to the nodes configuration, and set its value to "earth" for `node1` and `node2`, and to "mars" for `node3`
   1. 让我们假装我们部署的`eoc-06-cluster`集群横跨了俩数据中心，`earth` 和 `mars`。在节点配置里加入`AZ`这个属性，让`node1`和`node2`属于`earth`中心，`node3`属于`mars`中心
2. Restart the cluster
   1. 重启集群
3. Configure the cluster to force shard allocation awareness based on the two availability zones, and persist such configuration across cluster restarts
   1. 使得集群的分片分布基于俩数据中心的配置，并在集群重启时保持这个配置不失效
4. Verify the success of the last action by using the _cat API
   1. 通过`_cat`API来检验一下这些操作是否生效

## 第3题，题解

1. 修改每个节点的配置文件`$ES_HOME/config/elasticsearch.yml`
   1. 添加属性`node.attr.AZ: earth`和`node.attr.AZ: mars`
    Node1
    ```yml
    node.name: node1
    node.attr.AZ: earth
    ```

    Node2
    ```yml
    node.name: node2
    node.attr.AZ: earth
    ```

    Node3
    ```yml
    node.name: node3
    node.attr.AZ: mars
    ```
2. 重启集群
3. 运行`GET /_cat/nodeattrs?v&h=node,id,pid,host,ip,port,attr,value`命令来查看这个属性是否生效
```bash
       id   pid host       ip         port attr              value
es721Node2 q0pL 1   172.18.0.4 172.18.0.4 9300 name              es721Node2
es721Node2 q0pL 1   172.18.0.4 172.18.0.4 9300 AZ                earth
es721Node2 q0pL 1   172.18.0.4 172.18.0.4 9300 xpack.installed   true
es721Node1 PGDN 1   172.18.0.3 172.18.0.3 9300 ml.machine_memory 12564148224
es721Node1 PGDN 1   172.18.0.3 172.18.0.3 9300 xpack.installed   true
es721Node1 PGDN 1   172.18.0.3 172.18.0.3 9300 name              es721Node1
es721Node1 PGDN 1   172.18.0.3 172.18.0.3 9300 AZ                earth
es721Node1 PGDN 1   172.18.0.3 172.18.0.3 9300 ml.max_open_jobs  20
es721Node3 4ym8 1   172.18.0.5 172.18.0.5 9300 name              es721Node3
es721Node3 4ym8 1   172.18.0.5 172.18.0.5 9300 AZ                mars
es721Node3 4ym8 1   172.18.0.5 172.18.0.5 9300 xpack.installed   true
```
4. 给集群进行可用性配置的时候有两种方式
   1. 在master节点里指定需要使用的属性，可能的话加上可用的属性值
   ```yml
   cluster.routing.allocation.awareness.attributes: AZ
   cluster.routing.allocation.awareness.force.AZ.values: earth,mars
   ```
   1. 通过集群配置接口进行设置
   ```bash
   PUT /_cluster/settings
   {
      "persistent" : {
         "cluster.routing.allocation.awareness.attributes": "AZ",
         "cluster.routing.allocation.awareness.force.AZ.values": "earth,mars"
      }
   }
   ```  
5. 运行`GET /_cluster/settings`查看集群配置
   ```json
      {
      "persistent" : {
         "cluster" : {
            "routing" : {
            "allocation" : {
               "awareness" : {
                  "attributes" : "AZ"
               }
            }
            }
         }
      },
      "transient" : { }
      }
   ```

## 第3题，题解说明

* 上一题主要的配置写在每个 __index__ 的 __settings__ 里面，可以随时修改 + 生效，这一题中的配置主要写在配置文件里，作为节点的属性，需要修改/生效时就需要重启节点了。
  * 如果强行通过集群配置接口进行修改会报错不说，这种设置是节点级别的，在集群配置接口里也不好指定在哪个节点生效。
  ```bash
    PUT _cluster/settings
    {
      "persistent": {
        "node.attr.AZ":"earth"
      }
    }
    ```
    结果：
    ```json
    {
      "error": {
        "root_cause": [
          {
            "type": "illegal_argument_exception",
            "reason": "persistent setting [node.attr.AZ], not dynamically updateable"
          }
        ],
        "type": "illegal_argument_exception",
        "reason": "persistent setting [node.attr.AZ], not dynamically updateable"
      },
      "status": 400
    }
    ```
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/allocation-awareness.html)
    1. 页面路径：Modules =》 Shard allocation and cluster-level routing =》 Cluster level shard allocation
* 集群的分配属性可以通过配置文件或者集群配置接口来修改，但是要注意集群配置接口的更改分临时和永久两种情况，`persistent`是永久的`transient`是临时的，区别在于永久生效的配置集群重启之后还存在，临时的重启了就没了
   1. 通过接口设置的属性可以通过把值设为null来取消
   ```bash
      PUT /_cluster/settings
      {
         "persistent": {
            "cluster.routing.allocation.awareness.attributes": null,
            "cluster.routing.allocation.awareness.force.AZ.values": null
         }
      }
   ```
   1. 这里还要注意一点，两个awareness配置的写法不太一样，我在第一次写的时候也拼错了，他们前几位都一样都是`cluster.routing.allocation.`，但是最后几位一个是`awareness.attributes`用来标记用于发现的属性（key）的值，一个是`awareness.force.${key}.values`代表了这个属性（key）可用的值。
   1. [参考链接-awareness](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/allocation-awareness.html)，[参考链接-cluster 配置更新接口](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-update-settings.html)
   1. 页面路径：Modules =》 Shard allocation and cluster-level routing =》 Cluster level shard allocation
   1. 页面路径：Modules =》 Cluster APIs =》 Cluster Update Settings
* 集群配置获取接口是`GET /_cluster/settings` url参数 `include_defaults` 可以管理是否显示默认配置
   1. `GET /_cluster/settings?include_defaults=true` 可以带着系统默认配置一起返回
   2. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-get-settings.html)
   1. 页面路径：Modules =》 Cluster APIs =》 Cluster Get Settings

## 第4题，冷热架构部署

1. Configure the cluster to reflect a hot/warm architecture, with `node1` as the only hot node
   1. 配置集群以满足冷热架构的部署，让`node1`作为唯一的热节点
2. Configure the `hamlet-1` index to allocate its shards only to warm nodes
   1. 修改`hamlet-1`的索引配置，让他只能放置在温节点上
3. Verify the success of the last action by using the _cat API 
   1. 通过`_cat`API来校验这个操作是否成功
4. Remove the hot/warm shard filtering configuration from the `hamlet-1` configuration
   1. 从`hamlet-1`的配置中把冷热节点过滤的条件去掉

## 第4题，题解
1. 修改每个节点配置文件`$ES_HOME/config/elasticsearch.yml`
   1. 添加属性`node.attr.hot_warm_type: hot`和`node.attr.hot_warm_type: warm`
    Node1
    ```yml
    node.name: node1
    node.attr.hot_warm_type: hot
    ```

    Node2
    ```yml
    node.name: node2
    node.attr.hot_warm_type: warm
    ```

    Node3
    ```yml
    node.name: node3
    node.attr.hot_warm_type: warm
    ```

1. 检查节点属性`GET /_cat/nodeattrs?v&h=node,id,pid,host,ip,port,attr,value`
   ```bash
   node       id   pid host       ip         port attr              value
   es721Node2 q0pL 1   172.18.0.2 172.18.0.2 9300 name              es721Node2
   es721Node2 q0pL 1   172.18.0.2 172.18.0.2 9300 AZ                earth
   es721Node2 q0pL 1   172.18.0.2 172.18.0.2 9300 xpack.installed   true
   es721Node2 q0pL 1   172.18.0.2 172.18.0.2 9300 hot_warm_type     warm
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 ml.machine_memory 12564156416
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 xpack.installed   true
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 name              es721Node1
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 AZ                earth
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 ml.max_open_jobs  20
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 hot_warm_type     hot
   es721Node3 4ym8 1   172.18.0.3 172.18.0.3 9300 name              es721Node3
   es721Node3 4ym8 1   172.18.0.3 172.18.0.3 9300 AZ                mars
   es721Node3 4ym8 1   172.18.0.3 172.18.0.3 9300 xpack.installed   true
   es721Node3 4ym8 1   172.18.0.3 172.18.0.3 9300 hot_warm_type     warm
   ```

1. 把`hamlet-1`删了重建（避免其他配置的干扰）
   1. 索引删除`DELETE hamlet-1`
   2. 索引重建
   ```bash
   PUT hamlet-1
   {
      "settings": {
         "index.routing.allocation.include.hot_warm_type": "warm"
      }
   }
   ```
   或者
   ```bash
   PUT hamlet-1
   {
      "settings": {
         "index":{
            "number_of_shards":2,
            "number_of_replicas":0,
            "routing":{
            "allocation":{
               "include":{
                  "hot_warm_type":"warm"
               }
            }
            }
         }
      }
   }
   ```
1. 运行一下之前的给`hamlet-1`插数据的脚本
1. 运行`GET /_cat/shards/hamlet-1?v&h=index,shard,prirep,state,docs,store,ip,node`检查索引分片分布情况
   ```bash
   index    shard prirep state   docs store ip         node
   hamlet-1 1     p      STARTED    2 4.7kb 172.18.0.2 es721Node2
   hamlet-1 0     p      STARTED    3 5.1kb 172.18.0.2 es721Node2
   ```
1. 去掉`hamlet-1`有关冷热分片的配置
   ```bash
   PUT hamlet-1/_settings
   {
      "index.routing.allocation.include.hot_warm_type": null
   }
   ```
1. 运行`GET /_cat/shards/hamlet-1?v&h=index,shard,prirep,state,docs,store,ip,node`检查索引分片分布情况
   ```bash
   index    shard prirep state   docs store ip         node
   hamlet-1 1     p      STARTED    2 4.8kb 172.18.0.2 es721Node2
   hamlet-1 0     p      STARTED    3 5.2kb 172.18.0.4 es721Node1
   ```

## 第4题，题解说明

* 这题主要考察的是通过ES节点属性来进行冷热部署，本质上和之前的一些配置一样，也是基于`node.attr.`的附加配置，但是对于生产场景中，确实会存在集群节点的配置不尽相同，某些高性能节点具有更多的CPU，SSD硬盘用来存储、计算和召回数据更快，而另一些节点使用更少的内存，更差的硬盘，用来进行较低频数据的搜索
  1. 与上一题一样，首先我们需要给集群的各个节点添加节点属性，这个操作需要修改节点的配置/启动命令，所以需要重启生效
     1. [参考连接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/shard-allocation-filtering.html)
     2. 页面路径：Index modules =》 Index Shard Allocation =》 Index-level shard allocation filtering
  2. 接着对索引进行allocation的设置，可以直接通过`PUT ${index}/_settings`的接口进行修改
     1. [参考连接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/shard-allocation-filtering.html)
     2. 页面路径：Index modules =》 Index Shard Allocation =》 Index-level shard allocation filtering

## 第5题，基于节点存储属性部署
1. Let's assume that the nodes have either a "large" or "small" local storage. Add the attribute `storage` to the nodes config, and set its value so that `node2` is the only with a "small" storage
   1. 让我们假设节点存在 “大” 和 ”小“ 两种本地存储能力，给节点们添加 `storage` 属性，其中 `node2` 有 “小” 的存储，其他的都是 “大” 存储
1. Configure the `hamlet-2` index to allocate its shards only to nodes with a large storage size
   1. 修改 `hamlet-2` 索引的配置，让它只能把分片放置在 “大” 存储的节点上
2. Verify the success of the last action by using the _cat API
   1. 通过 `_cat` API 来校验操作成功与否

## 第4题，题解

1. 修改每个节点配置文件`$ES_HOME/config/elasticsearch.yml`
   1. 添加属性`node.attr.storage: large`和`node.attr.storage: small`
    Node1
    ```yml
    node.name: node1
    node.attr.storage: large
    ```

    Node2
    ```yml
    node.name: node2
    node.attr.storage: small
    ```

    Node3
    ```yml
    node.name: node3
    node.attr.storage: large
    ```
1. 修改`hamlet-2`的索引配置
   ```bash
   PUT hamlet-2
   {
      "settings": {
         "index.routing.allocation.include.storage": "large"
      }
   }
   ```
   或者
   ```bash
   PUT hamlet-2/_settings
   {
      "routing": {
         "allocation": {
         "include": {
            "storage": "small",
            "AZ": "earth",
            "hot_warm_type": "hot"
         }
         }
      }
   }
   ```
1. 通过接口`GET /_cat/nodeattrs?v&h=node,id,pid,host,ip,port,attr,value`，`GET hamlet-2`和`GET /_cat/shards/hamlet-2?v&h=index,shard,prirep,state,docs,store,ip,node`来检验索引、节点和分片的属性和放置情况
   1. `GET /_cat/nodeattrs?v&h=node,id,pid,host,ip,port,attr,value`
   ```bash
   node       id   pid host       ip         port attr              value
   es721Node2 q0pL 1   172.18.0.2 172.18.0.2 9300 name              es721Node2
   es721Node2 q0pL 1   172.18.0.2 172.18.0.2 9300 AZ                earth
   es721Node2 q0pL 1   172.18.0.2 172.18.0.2 9300 xpack.installed   true
   es721Node2 q0pL 1   172.18.0.2 172.18.0.2 9300 storage           small
   es721Node2 q0pL 1   172.18.0.2 172.18.0.2 9300 hot_warm_type     warm
   es721Node3 4ym8 1   172.18.0.3 172.18.0.3 9300 name              es721Node3
   es721Node3 4ym8 1   172.18.0.3 172.18.0.3 9300 AZ                mars
   es721Node3 4ym8 1   172.18.0.3 172.18.0.3 9300 xpack.installed   true
   es721Node3 4ym8 1   172.18.0.3 172.18.0.3 9300 storage           large
   es721Node3 4ym8 1   172.18.0.3 172.18.0.3 9300 hot_warm_type     warm
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 ml.machine_memory 12564156416
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 xpack.installed   true
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 name              es721Node1
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 AZ                earth
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 ml.max_open_jobs  20
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 storage           large
   es721Node1 PGDN 1   172.18.0.4 172.18.0.4 9300 hot_warm_type     hot
   ```
   1. `GET hamlet-2`
   ```json
   {
      "hamlet-2" : {
         "aliases" : { },
         "mappings" : { },
         "settings" : {
            "index" : {
            "routing" : {
               "allocation" : {
                  "include" : {
                  "AZ" : "earth",
                  "hot_warm_type" : "hot",
                  "storage" : "small"
                  }
               }
            },
            "number_of_shards" : "1",
            "provided_name" : "hamlet-2",
            "creation_date" : "1604987394421",
            "number_of_replicas" : "1",
            "uuid" : "9sVWeaDTTFS9XeQWzMJu4w",
            "version" : {
               "created" : "7020199"
            }
            }
         }
      }
   }
   ```
   1. `GET /_cat/shards/hamlet-2?v&h=index,shard,prirep,state,docs,store,ip,node`
   ```bash
      index    shard prirep state      docs store ip         node
   hamlet-2 0     p      STARTED       0  283b 172.18.0.2 es721Node2
   hamlet-2 0     r      UNASSIGNED                       
   ```
2. （如果存在）用`GET /_cluster/allocation/explain`来查看处于`UNASSIGNED`状态的分片/副本
   1. `GET /_cluster/allocation/explain`
   ```bash
   {
      "index" : "hamlet-2",
      "shard" : 0,
      "primary" : false,
      "current_state" : "unassigned",
      "unassigned_info" : {
         "reason" : "INDEX_CREATED",
         "at" : "2020-11-10T05:49:54.426Z",
         "last_allocation_status" : "no_attempt"
      },
      "can_allocate" : "no",
      "allocate_explanation" : "cannot allocate because allocation is not permitted to any of the nodes",
      "node_allocation_decisions" : [
         {
            "node_id" : "PGDN7jTpTPes0ICgNb19Bw",
            "node_name" : "es721Node1",
            "transport_address" : "172.18.0.4:9300",
            "node_attributes" : {
               "ml.machine_memory" : "12564156416",
               "xpack.installed" : "true",
               "name" : "es721Node1",
               "AZ" : "earth",
               "ml.max_open_jobs" : "20",
               "storage" : "large",
               "hot_warm_type" : "hot"
            },
            "node_decision" : "no",
            "weight_ranking" : 1,
            "deciders" : [
               {
                  "decider" : "awareness",
                  "decision" : "NO",
                  "explanation" : "there are too many copies of the shard allocated to nodes with attribute [AZ], there are [2] total configured shard copies for this shard id and [3] total attribute values, expected the allocated shard count per attribute [2] to be less than or equal to the upper bound of the required number of shards per attribute [1]"
               }
            ]
         },
         {
            "node_id" : "4ym8nm8WS7yu1RcN1v7mcg",
            "node_name" : "es721Node3",
            "transport_address" : "172.18.0.3:9300",
            "node_attributes" : {
               "name" : "es721Node3",
               "AZ" : "mars",
               "xpack.installed" : "true",
               "storage" : "large",
               "hot_warm_type" : "warm"
            },
            "node_decision" : "no",
            "weight_ranking" : 2,
            "deciders" : [
               {
                  "decider" : "filter",
                  "decision" : "NO",
                  "explanation" : """node does not match index setting [index.routing.allocation.include] filters [AZ:"earth",storage:"small",hot_warm_type:"hot"]"""
               }
            ]
         },
         {
            "node_id" : "q0pL9eXcSviCphv0EhV52g",
            "node_name" : "es721Node2",
            "transport_address" : "172.18.0.2:9300",
            "node_attributes" : {
               "name" : "es721Node2",
               "AZ" : "earth",
               "xpack.installed" : "true",
               "storage" : "small",
               "hot_warm_type" : "warm"
            },
            "node_decision" : "no",
            "weight_ranking" : 3,
            "deciders" : [
               {
                  "decider" : "same_shard",
                  "decision" : "NO",
                  "explanation" : "the shard cannot be allocated to the same node on which a copy of the shard already exists [[hamlet-2][0], node[q0pL9eXcSviCphv0EhV52g], [P], s[STARTED], a[id=KBmldvUvQfGulBhS2fQ9Ug]]"
               },
               {
                  "decider" : "awareness",
                  "decision" : "NO",
                  "explanation" : "there are too many copies of the shard allocated to nodes with attribute [AZ], there are [2] total configured shard copies for this shard id and [3] total attribute values, expected the allocated shard count per attribute [2] to be less than or equal to the upper bound of the required number of shards per attribute [1]"
               }
            ]
         }
      ]
   }
   ```

## 第4题，题解说明
* 这题前面一半和上一题一样，通过节点的额外属性`node.attr.${attribute}`配合索引的`index.routing.allocation.include.${attribute}`来管理索引分片的放置，后半段考察的也是平时做类似分片管理的时候需要注意的就是节点的属性可能会存在除斥（多个配置不存在交集），导致索引的分片无法被放置，以至于被标记为`UNASSIGNED`
  * 为了能满足这个状态，我把题目的一些要求稍做了修改
  1. 节点配置属性和索引添加节点筛选配置和上题一样，略
  2. 查看集群中未分配的分片以及不能分配的原因的接口是`GET /_cluster/allocation/explain`
     1. 这个接口的返回结果中主要会包含以下内容
        1. "index" : "hamlet-2"：当前分片所属索引
        2. "shard" : 0：第几个分片（从0开始）
        3. "primary" : false：是否主分片（true：主分片，false：副本）
        4. "current_state" : "unassigned"：当前状态：未分配
        5. "unassigned_info" : 未分配的相关信息
           1. "reason" : "INDEX_CREATED"：未分配原因，索引创建（失败）
           2. "at" : "2020-11-10T05:49:54.426Z"：操作时间
           3. "last_allocation_status" : "no_attempt"：最后一次（尝试）分配状态：失败
        6. "can_allocate" : "no"：是否可分配：否
        7. "allocate_explanation" : "cannot allocate because allocation is not permitted to any of the nodes"：分配解释：无法分配分片，因为分片配置无法匹配任何节点
        8. "node_allocation_decisions" : 各个节点分配情况详情
     2. 其他的不写了，看key和value也可以知道大概意思
     3. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-allocation-explain.html)
     4. 页面路径：Cluster APIs =》 Cluster Allocation Explain API
  3. 当`UNASSIGNED`分片存在的时候，通过接口`GET _cat/health`可以看出来集群是不健康的
     1. `GET _cat/health?v`
         ```bash
         epoch      timestamp cluster        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
         1604989564 06:26:04  docker-cluster yellow          3         3     11   7    0    0        1             0                  -                 91.7%
         ```
         1. 主要看`status`和`active_shards_percent`这两列
            1. `status`
               1. `yellow`：有副本无法分配或缺失，集群不健康，但是可以支持搜索，几乎无数据缺失
               2. `red`：有主分片无法分配或缺失，集群严重不健康，部分数据不能搜索，有数据缺失
            2. `active_shards_percent`
               1. 活跃（能正常使用）分片数占比
      1. [参考连接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cat-health.html)
      2. 页面路径：cat APIs =》 cat health