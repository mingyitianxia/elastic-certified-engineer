## CLUSTER ADMINISTRATION
集群管理

## GOAL: Backup and cross-cluster search
目标：备份和跨集群搜索

## REQUIRED SETUP: 
建议docker-compose文件：`2e2k_two_clusters.yml`

需求几步骤：

Let’s create a one-node cluster and index some data in it.
让我们先搞一个但节点的集群，然后存点数据进去
1. Download the exam version of Elasticsearch
   1. 下载考试版本的ES
2. Deploy the cluster `eoc-06-original-cluster`, with one node named `node-1`
   1. 初始化一个单节点的集群，节点名`node-1`，集群名`eoc-06-original-cluster`
3. Start the cluster
   1. 启动集群
4. Create the index `hamlet` and add some documents by running the following _bulk command
   1. 创建一个叫`hamlet`的索引，并用下面 ⬇️ 的语句存谢数据进去

```bash
PUT hamlet/_bulk
{"index":{"_index":"hamlet","_id":0}}
{"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet","_id":1}}
{"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet","_id":2}}
{"line_number":"3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet","_id":3}}
{"line_number":"4","speaker":"FRANCISCO","text_entry":"Bernardo?"}
{"index":{"_index":"hamlet","_id":4}}
{"line_number":"5","speaker":"BERNARDO","text_entry":"He."}
```
或者
```bash
DELETE hamlet
PUT hamlet/_doc/1
{"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
PUT hamlet/_doc/2
{"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
```

## 第1题，创建备份存储空间

1. Configure `node-1` to support a shared file system repository for backups located in
   1. 修改`node-1`的节点配置，让它可以支持把文件存在以下目录
  1. "[home_folder]/repo" and
  2. "[home_folder]/elastic/repo" - e.g., "glc/elastic/repo"
1. Create the `hamlet_backup` shared file system repository in "[home_folder]/elastic/repo"
   1. 创建一个叫`hamlet_backup`存储空间在"[home_folder]/elastic/repo"

## 第1题，题解
（由于我是基于我的docker-compose文件启动的集群，所以会和题目不完全一样，比如题目里的node1我用的是esNode1，下同）

1. 创建共享存储
   1. 在安装ES的机器的命令行里执行`mkdir ${dir}`
   2. 设置成nfs共享存储，并挂载到集群的所有节点上
2. 把这部分地址写进配置文件`$ES_HOME/config/elasticsearch.yml`
   1. `echo 'path.repo: [${dir}]' >> /$ES_HOME/config/elasticsearch.yml`
3. 重启集群中的所有节点是这个配置生效
4. 把共享存储注册为`hamlet_backup`的存储仓库
   1. 执行以下命令
   ```bash
   PUT _snapshot/hamlet_backup
    {
      "type": "fs",
      "settings": {
        "location": "/usr/share/elasticsearch/backup"
      }
    }
   ```

## 第1题，题解说明

* 这题主要考察snapshot仓库的创建，在生产中主要可能存在的问题是：
  * 共享存储的路径应该是被集群中所有节点（包括源集群和目标集群）都能共享访问的（带读写权限）
  * 如果是集群先存在，共享存储后挂载的话，需要对集群中的所有节点进行rolling restart
    * 可以通过前两章中的`"index.routing.allocation.exclude._name": "$node_name"`，`"index.routing.allocation.exclude._ip_": "$node_ip",`等设置将索引移出待重启节点，然后再对索引进行重启以使得配置生效
  1. [参考链接-modules-snapshot](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-snapshots.html)，[参考链接-create-snapshot](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/snapshots-take-snapshot.html)
     1. 由于7.X早期版本的`Create snapshot`页面不见了，这里贴的是7.9的相关页面，但其实从命令到操作方式上几乎没区别
  2. 页面路径-modules-snapshot：Modules =》 Snapshot And Restore
  3. 页面路径-create-snapshot：Snapshot and restore =》 Register repository

## 第2题，创建snapshot，自体集群的数据备份和恢复

1. Create a snapshot of the `hamlet` index, so that the snapshot
   1. 创建一个索引`hamlet`的snapshot
   2. is named `hamlet_snapshot_1`,
      1. 取名叫`hamlet_snapshot_1`
   3. is stored into `hamlet_backup` 
      1. 并保存在`hamlet_backup`的仓库里
2. Delete the index `hamlet`
   1. 把`hamlet`这索引删了
3. Restore the index `hamlet` using `hamlet_snapshot_1`
   1. 再从`hamlet_snapshot_1`里恢复`hamlet`的数据

## 第2题，题解

1. 创建snapshot，运行以下命令
   
  ```bash
  PUT /_snapshot/hamlet_backup/hamlet_snapshot_1?wait_for_completion=true
  {
    "indices": "hamlet",
    "ignore_unavailable": true,
    "include_global_state": false
  }
  ```
   * 可能返回值

  ```json
  {
    "snapshot" : {
      "snapshot" : "hamlet_snapshot_1",
      "uuid" : "vv_unWlzTGyAKc8VJVA5aA",
      "version_id" : 7020199,
      "version" : "7.2.1",
      "indices" : [
        "hamlet"
      ],
      "include_global_state" : false,
      "state" : "SUCCESS",
      "start_time" : "2020-11-12T02:53:42.547Z",
      "start_time_in_millis" : 1605149622547,
      "end_time" : "2020-11-12T02:53:42.631Z",
      "end_time_in_millis" : 1605149622631,
      "duration_in_millis" : 84,
      "failures" : [ ],
      "shards" : {
        "total" : 1,
        "failed" : 0,
        "successful" : 1
      }
    }
  }
  ```
  
  * 如果多跑一次
   
  ```json
  {
    "error": {
      "root_cause": [
        {
          "type": "invalid_snapshot_name_exception",
          "reason": "[hamlet_backup:hamlet_snapshot_1] Invalid snapshot name [hamlet_snapshot_1], snapshot with the same name already exists"
        }
      ],
      "type": "invalid_snapshot_name_exception",
      "reason": "[hamlet_backup:hamlet_snapshot_1] Invalid snapshot name [hamlet_snapshot_1], snapshot with the same name already exists"
    },
    "status": 400
  }
  ```

1. 删除索引，`DELETE hamlet`
1. 从`hamlet_snapshot_1`恢复数据

  ```bash
  POST /_snapshot/hamlet_backup/hamlet_snapshot_1/_restore?wait_for_completion=true
  {
    "indices": "hamlet",
    "include_global_state": false,
    "ignore_unavailable": true
  }
  ```

  * 可能返回值（不加wait_for_completion=true）

  ```json
  {
    "accepted": true
  }
  ```

  * wait_for_completion情况

  ```json
  {
    "snapshot" : {
      "snapshot" : "hamlet_snapshot_1",
      "indices" : [
        "hamlet"
      ],
      "shards" : {
        "total" : 1,
        "failed" : 0,
        "successful" : 1
      }
    }
  }
  ```

  * 如果多跑一次

  ```json
  {
    "error": {
      "root_cause": [
        {
          "type": "snapshot_restore_exception",
          "reason": "[hamlet_backup:hamlet_snapshot_1/vv_unWlzTGyAKc8VJVA5aA] cannot restore index [hamlet] because an open index with same name already exists in the cluster. Either close or delete the existing index or restore the index under a different name by providing a rename pattern and replacement name"
        }
      ],
      "type": "snapshot_restore_exception",
      "reason": "[hamlet_backup:hamlet_snapshot_1/vv_unWlzTGyAKc8VJVA5aA] cannot restore index [hamlet] because an open index with same name already exists in the cluster. Either close or delete the existing index or restore the index under a different name by providing a rename pattern and replacement name"
    },
    "status": 500
  }
  ```

1. 运行`GET _cat/indices` 检查一下索引是否还存在
1. 运行`GET hamlet/_search`

## 第2题，题解说明

* 这题主要考察的是数据的恢复，需要注意的
  1. 因为snapshot的id和index的名字是唯一的，所以重复运行会报错
  2. 多个索引可以备份在同一个snapshot里，同一个snapshot也可以恢复1个到多个索引，在备份和恢复的时候要做好规划
  3. 在备份和恢复的时候会有一些附加参数可以设置，比如：
     * `ignore_unavailable`用来跳过不可用的索引
     * `include_global_state`用来决定是否包含集群的全局配置
     * `partial`用来决定如果恢复数据的时候有些index出现了报错，同批次的其他索引的恢复是继续还是一同失败
  4. 测试里的数据量比较小，snapshot保存和恢复速度较快，可以添加`wait_for_completion=true`来等待流程完成，在真正的生产环境中进行操作时，建议不加这个参数，而是通过提交任务以及调用`GET /_snapshot/hamlet_snapshot_1/_current`接口的方式来查看snapshot的创建和恢复的过程。
  5. [参考链接-modules-snapshots](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-snapshots.html)，[参考链接-restore-snapshots](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/snapshots-restore-snapshot.html)
  6. 页面路径-modules-snapshots：Modules =》 Snapshot And Restore
  7. 页面路径-restore-snapshots：Snapshot and restore =》 Restore a snapshot
  8. 可以直接用在生产上的，通过snapshot进行数据迁移的操作手顺：[参考链接](https://blog.csdn.net/weixin_40601534/article/details/108124488)

## 第3题，多集群间通过snapshot进行数据迁移，以及跨集群的数据搜索

1. Deploy a second cluster `eoc-06-adaptation-cluster`, with one node named `node-2`
   1. 部署第二个集群叫`eoc-06-adaptation-cluster`其中一个节点叫`node-2`
2. Start the cluster
   1. 启动集群
3. （我加的）通过snapshot把第一个集群里的`hamlet`恢复到这里来
4. Create the index `hamlet-pirate` on `node-2` and add documents using the _bulk command
   1. 在`node-2`里创建一个索引叫`hamlet-pirate`
5. Enable cross cluster search on `eoc-06-adaptation-cluster`, so that
   1. 给集群`eoc-06-adaptation-cluster`开启跨集群搜索功能
   2. the name of the remote cluster is `original`,
      1. 远程集群设置为`original`
   3. the seed is `node-1`, which is listening on the default transport port,
      1. 连接地址是`node-1`的，监听它的默认端口
   4. the cross cluster configuration persists across multiple restarts
      1. 让跨集群的配置生效可能需要重启几个集群
   5. Run the cross-cluster query below to check your setup 
      1. 运行下面的跨集群搜索命令来校验你的配置是否成功

## 第3题，题解

1. 创建集群大部分操作参考之前的章节，这里不再赘述
2. 跨集群数据恢复：
   1. 在新集群里注册存储地址
    ```bash
    PUT _snapshot/hamlet_backup
      {
        "type": "fs",
        "settings": {
          "location": "/usr/share/elasticsearch/backup"
        }
      }
    ```
   1. 使用和本集群自己恢复的命令一样（也可以根据需要稍作修改）
    ```bash
    POST /_snapshot/hamlet_backup/hamlet_snapshot_1/_restore?wait_for_completion=true
    {
      "indices": "hamlet",
      "include_global_state": false,
      "ignore_unavailable": true
    }
    ```
3. 开启跨集群搜索
4. 有两种方式：
     * 通过kibana的设置：
        1. 左边菜单栏里点`management`
        2. 在`Elasticsearch`里点`Remote Clusters`
        3. `Add a remote cluster`
        4. 表单里填：
           1. `Name`：`original`
           2. `Seed Node`：`es721Node1:9300` （因为我是在docker-compose集群里进行操作，所以写`es721Node1`这个内部域名，真正的生产环境换成实际的ip地址）
        5. `Save`
     * 通过ES的api：
        1. 运行以下命令：
          ```bash
          PUT _cluster/settings
          {
            "persistent": {
              "cluster": {
                "remote": {
                  "original": {
                    "seeds": [
                      "es721Node1:9300"
                    ]
                  }
                }
              }
            }
          }
          ```
1. 运行跨集群搜索命令进行数据校验
    ```bash
    POST /original:hamlet/_search
    {
      "query": {
        "match": {
          "speaker" : "BERNARDO"
        }
      }
    }
    ```

## 第3题，题解说明

* 这题前面一半考察的是数据恢复，和上一题一样，只是操作的平台从本集群换成了远程集群，那么需要注意的就是集群之间需要可以同时访问同一块共享存储的地址
* 后面一半考察的是跨集群搜索的设置，跨集群搜索要先注册远程集群，以一个或几个节点的域名/ip地址配合默认端口来相互通讯，但是这里要注意的是，集群间通讯的不是`http.port`（默认9200）而是`transport.port`（默认9300）
  1. [参考链接-modules-remote-clusters](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-remote-clusters.html)，[参考链接-modules-cross-cluster-search](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cross-cluster-search.html)
  2. 页面路径-modules-remote-clusters：Modules =》 Remote clusters
  3. 页面路径-modules-cross-cluster-search：Modules =》 Cross-cluster search