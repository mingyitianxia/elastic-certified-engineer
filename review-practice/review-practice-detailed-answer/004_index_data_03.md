## Analyze & Save
分析和保存（数据）

## GOAL: analyze and save data against requirements
目标：按要求分析（分词）和保存数据
建议docker-compose文件：`1e1k_base_cluster.yml`

## 第1题，索引与别名

1. Create the indices `hamlet-1` and `hamlet-2`, each with two primary shards and no replicas
   1. 创建两个索引，`hamlet-1` 和 `hamlet-2`，让他们都有2个主分片没有副本
2. Add some documents to `hamlet-1` by running the following command
   1. 用下面的命令给`hamlet-1`添加一些数据
      ```bash
      PUT hamlet-1/_doc/_bulk
      {"index":{"_index":"hamlet-1","_id":0}}
      {"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
      {"index":{"_index":"hamlet-1","_id":1}}
      {"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
      {"index":{"_index":"hamlet-1","_id":2}}
      {"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
      {"index":{"_index":"hamlet-1","_id":3}}
      {"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
      ```
3. Add some documents to `hamlet-2` by running the following command
   1. 用下面的命令给`hamlet-2`添加一些数据
      ```bash
      PUT hamlet-2/_doc/_bulk
      {"index":{"_index":"hamlet-2","_id":4}}
      {"line_number":"2.1.1","speaker":"LORD POLONIUS","text_entry":"Give him this money and these notes, Reynaldo."}
      {"index":{"_index":"hamlet-2","_id":5}}
      {"line_number":"2.1.2","speaker":"REYNALDO","text_entry":"I will, my lord."}
      {"index":{"_index":"hamlet-2","_id":6}}
      {"line_number":"2.1.3","speaker":"LORD POLONIUS","text_entry":"You shall do marvellous wisely, good Reynaldo,"}
      {"index":{"_index":"hamlet-2","_id":7}}
      {"line_number":"2.1.4","speaker":"LORD POLONIUS","text_entry":"Before you visit him, to make inquire"}
      ```
4. Create the alias `hamlet` that maps both `hamlet-1` and `hamlet-2` 
   1. 创建一个叫`hamlet`的索引别名，同时指向`hamlet-1`和`hamlet-2`两个索引
5. Verify that the documents grouped by `hamlet` are 8
   1. 校验一下所有被`hamlet`覆盖的数据一共有8条

## 第1题，题解

1. 创建索引
   1. 清理现有数据
      ```bash
      DELETE hamlet
      DELETE hamlet-1
      DELETE hamlet-2
      ```

   2. 创建新索引
      ```bash
      PUT hamlet-1
      {
        "settings": {
          "number_of_shards": 2,
          "number_of_replicas": 0
        }
      }
      ```

      ```bash
      PUT hamlet-2
      {
        "settings": {
          "number_of_shards": 2,
          "number_of_replicas": 0
        }
      }
      ```

   3. 索引校验
   
      `GET hamlet-1`

      `GET hamlet-2`
2. 插数据，命令如上 ⬆️ 略
3. 创建别名
    ```bash
    PUT _aliases
    {
      "actions": [
          {
            "add": {
                "index": "hamlet-1",
                "alias": "hamlet"
            }
          },
          {
            "add": {
                "index": "hamlet-2",
                "alias": "hamlet"
            }
          }
      ]
    }
    ```
4. 校验数据，`GET hamlet/_count`
   * 返回值
   ```json
   {
     "count" : 8,
     "_shards" : {
       "total" : 4,
       "successful" : 4,
       "skipped" : 0,
       "failed" : 0
     }
   }
   ```

## 第1题，题解说明

* 这题主要考察的是索引别名的使用，索引的别名类似操作系统里的快捷方式或者软连接，访问它就相当于访问了它所指向的所有的索引
  * 对索引别名的查询操作，ES会把它所保护的所有的索引作为一个整体进行召回和排序
  * 对索引别名的写操作只能作用在一个索引上，否则会报找不到特定索引的错，一般如果有类似使用场景的话可以考虑索引和别名1对1设置，或者对需要进行写操作的索引设置为写索引（`"is_write_index" : true`）
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-aliases.html)
  2. 页面路径：Indices APIs =》 Index Aliases

## 第2题，通过script对数据进行修改

1. Add a document to `hamlet`, so that the document:
   1. 添加一个满足下列要求的文档到`hamlet`
   2. has id "8", 
      1. id是“8”
   3. has "_doc" type,
      1. type是"_doc"
   4. has a field `text_entry` with value "With turbulent and dangerous lunacy?"
      1. 有个叫`text_entry` 的字段，值是 "With turbulent and dangerous lunacy?"
   5. has a field `line_number` with value "3.1.4"
      1. 有个叫`line_number`的字段，值是 "3.1.4",
   6. has a field `speaker` with value "KING CLAUDIUS"
      1. 有个叫`speaker`的字段，值是"KING CLAUDIUS"
2.  Create a script named `control_reindex_batch` and save it into the cluster state. 
    1.  创建一个满足以下条件的名字叫`control_reindex_batch`的script
    2.  The script checks whether a document has the field `reindexBatch`, and
        1.  这script会去检查文档存不存在一个`reindexBath`字段
        2.  in the affirmative case, it increments the field value by a script parameter named `increment`
            1.  如果存在，script会在原有的值上加上script参数里`increment`对应的值
        3.  otherwise, the script adds the field to the document setting its value to "1"
            1.  否则，script就在原有的文档里加上这个字段，并把它设为1
1. Create the index `hamlet-new` with 2 primary shards and no replicas
   1. 创建一个拥有2分片0副本的新索引，叫`hamlet-new`
2. Reindex `hamlet` into `hamlet-new`, while satisfying the following criteria:
   1. 把`hamlet`里的数据通过下面的要求reindex到`hamlet-new`里面
   2. apply the `control_reindex_batch` script with the `increment` parameter set to "1", 
      1. 应用`control_reindex_bath`脚本，并且使用“1”作为`increment`的值
   3. reindex using two parallel slices
      1. 通过2个线程来进行这个reindex操作

## 第2题，题解

1. 添加数据
    ```bash
    POST hamlet/_doc/8
    {
      "text_entry": "With turbulent and dangerous lunacy?",
      "line_number": "3.1.4",
      "speaker": "KING CLAUDIUS"
    }
    ```
1. 创建script
    ```bash
    POST _scripts/control_reindex_batch
    {
      "script": {
        "lang": "painless",
        "source": "if (null == ctx._source.reindexBath) { ctx._source.reindexBath = 1 } else { ctx._source.reindexBath += params.increments}"
      }
    }
    ```
1. 创建索引
    ```bash
    DELETE hamlet-new
    PUT hamlet-new
    {
      "settings": {
        "number_of_shards": 2,
        "number_of_replicas": 0
      }
    }
    ```
1. 做reindex操作
    ```bash
    POST _reindex?slices=2
    {
      "source": {
        "index": "hamlet"
      },
      "dest": {
        "index": "hamlet-new"
      },
      "script": {
        "id": "control_reindex_batch",
        "params": {
          "increments": 1
        }
      }
    }
    ```

    * 返回值
    ```json
    {
      "took" : 556,
      "timed_out" : false,
      "total" : 8,
      "updated" : 0,
      "created" : 8,
      "deleted" : 0,
      "batches" : 2,
      "version_conflicts" : 0,
      "noops" : 0,
      "retries" : {
        "bulk" : 0,
        "search" : 0
      },
      "throttled_millis" : 0,
      "requests_per_second" : -1.0,
      "throttled_until_millis" : 0,
      "slices" : [
        {
          "slice_id" : 0,
          "total" : 6,
          "updated" : 0,
          "created" : 6,
          "deleted" : 0,
          "batches" : 1,
          "version_conflicts" : 0,
          "noops" : 0,
          "retries" : {
            "bulk" : 0,
            "search" : 0
          },
          "throttled_millis" : 0,
          "requests_per_second" : -1.0,
          "throttled_until_millis" : 0
        },
        {
          "slice_id" : 1,
          "total" : 2,
          "updated" : 0,
          "created" : 2,
          "deleted" : 0,
          "batches" : 1,
          "version_conflicts" : 0,
          "noops" : 0,
          "retries" : {
            "bulk" : 0,
            "search" : 0
          },
          "throttled_millis" : 0,
          "requests_per_second" : -1.0,
          "throttled_until_millis" : 0
        }
      ],
      "failures" : [ ]
    }
    ```
1. 校验数据
   1. ```GET hamlet-new/_search``` （用来确定script中当`reindexBath`为空时，把它设为“1”的逻辑成功）
      ```json
      {
        "took" : 1,
        "timed_out" : false,
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "skipped" : 0,
          "failed" : 0
        },
        "hits" : {
          "total" : {
            "value" : 8,
            "relation" : "eq"
          },
          "max_score" : 1.0,
          "hits" : [
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "1",
              "_score" : 1.0,
              "_source" : {
                "reindexBath" : 1,
                "line_number" : "1.1.2",
                "text_entry" : "Nay, answer me: stand, and unfold yourself.",
                "speaker" : "FRANCISCO"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "5",
              "_score" : 1.0,
              "_source" : {
                "reindexBath" : 1,
                "line_number" : "2.1.2",
                "text_entry" : "I will, my lord.",
                "speaker" : "REYNALDO"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "2",
              "_score" : 1.0,
              "_source" : {
                "reindexBath" : 1,
                "line_number" : "1.1.3",
                "text_entry" : "Long live the king!",
                "speaker" : "BERNARDO"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "6",
              "_score" : 1.0,
              "_source" : {
                "reindexBath" : 1,
                "line_number" : "2.1.3",
                "text_entry" : "You shall do marvellous wisely, good Reynaldo,",
                "speaker" : "LORD POLONIUS"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "3",
              "_score" : 1.0,
              "_source" : {
                "reindexBath" : 1,
                "line_number" : "1.2.1",
                "text_entry" : "Though yet of Hamlet our dear brothers death",
                "speaker" : "KING CLAUDIUS"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "7",
              "_score" : 1.0,
              "_source" : {
                "reindexBath" : 1,
                "line_number" : "2.1.4",
                "text_entry" : "Before you visit him, to make inquire",
                "speaker" : "LORD POLONIUS"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "0",
              "_score" : 1.0,
              "_source" : {
                "reindexBath" : 1,
                "line_number" : "1.1.1",
                "text_entry" : "Whos there?",
                "speaker" : "BERNARDO"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "4",
              "_score" : 1.0,
              "_source" : {
                "reindexBath" : 1,
                "line_number" : "2.1.1",
                "text_entry" : "Give him this money and these notes, Reynaldo.",
                "speaker" : "LORD POLONIUS"
              }
            }
          ]
        }
      }
      ```

   2. 重新运行上面 ⬆️ 的reindex命令 （用来确定script中当`reindexBath`不为空时，给他增加`params.increments`的逻辑成功）
      1. 把source里的index改成 `hamlet-new`
      2. 把dest里的index改成`hamlet-new1`
          ```bash
          POST _reindex?slices=2
          {
            "source": {
              "index": "hamlet-new"
            },
            "dest": {
              "index": "hamlet-new-1"
            },
            "script": {
              "id": "control_reindex_batch",
              "params": {
                "increments": 1
              }
            }
          }
          ```
      3. 运行```GET hamlet-new-1/_search```
          ```json
          {
            "took" : 2,
            "timed_out" : false,
            "_shards" : {
              "total" : 1,
              "successful" : 1,
              "skipped" : 0,
              "failed" : 0
            },
            "hits" : {
              "total" : {
                "value" : 8,
                "relation" : "eq"
              },
              "max_score" : 1.0,
              "hits" : [
                {
                  "_index" : "hamlet-new-1",
                  "_type" : "_doc",
                  "_id" : "0",
                  "_score" : 1.0,
                  "_source" : {
                    "reindexBath" : 2,
                    "line_number" : "1.1.1",
                    "text_entry" : "Whos there?",
                    "speaker" : "BERNARDO"
                  }
                },
                {
                  "_index" : "hamlet-new-1",
                  "_type" : "_doc",
                  "_id" : "5",
                  "_score" : 1.0,
                  "_source" : {
                    "reindexBath" : 2,
                    "line_number" : "2.1.2",
                    "text_entry" : "I will, my lord.",
                    "speaker" : "REYNALDO"
                  }
                },
                {
                  "_index" : "hamlet-new-1",
                  "_type" : "_doc",
                  "_id" : "4",
                  "_score" : 1.0,
                  "_source" : {
                    "reindexBath" : 2,
                    "line_number" : "2.1.1",
                    "text_entry" : "Give him this money and these notes, Reynaldo.",
                    "speaker" : "LORD POLONIUS"
                  }
                },
                {
                  "_index" : "hamlet-new-1",
                  "_type" : "_doc",
                  "_id" : "1",
                  "_score" : 1.0,
                  "_source" : {
                    "reindexBath" : 2,
                    "line_number" : "1.1.2",
                    "text_entry" : "Nay, answer me: stand, and unfold yourself.",
                    "speaker" : "FRANCISCO"
                  }
                },
                {
                  "_index" : "hamlet-new-1",
                  "_type" : "_doc",
                  "_id" : "2",
                  "_score" : 1.0,
                  "_source" : {
                    "reindexBath" : 2,
                    "line_number" : "1.1.3",
                    "text_entry" : "Long live the king!",
                    "speaker" : "BERNARDO"
                  }
                },
                {
                  "_index" : "hamlet-new-1",
                  "_type" : "_doc",
                  "_id" : "6",
                  "_score" : 1.0,
                  "_source" : {
                    "reindexBath" : 2,
                    "line_number" : "2.1.3",
                    "text_entry" : "You shall do marvellous wisely, good Reynaldo,",
                    "speaker" : "LORD POLONIUS"
                  }
                },
                {
                  "_index" : "hamlet-new-1",
                  "_type" : "_doc",
                  "_id" : "3",
                  "_score" : 1.0,
                  "_source" : {
                    "reindexBath" : 2,
                    "line_number" : "1.2.1",
                    "text_entry" : "Though yet of Hamlet our dear brothers death",
                    "speaker" : "KING CLAUDIUS"
                  }
                },
                {
                  "_index" : "hamlet-new-1",
                  "_type" : "_doc",
                  "_id" : "7",
                  "_score" : 1.0,
                  "_source" : {
                    "reindexBath" : 2,
                    "line_number" : "2.1.4",
                    "text_entry" : "Before you visit him, to make inquire",
                    "speaker" : "LORD POLONIUS"
                  }
                }
              ]
            }
          }
          ```

## 第2题，题解说明

* 这题主要考察的是对索引配合script进行reindex的操作，reindex命令可以在集群内部通过一些筛选/修改命令将源索引（单个或多个）写入满足条件的索引
  * 在这题中涉及到的script部分的用法，主要涉及到保存、作为pipeline处理数据等，其中比较容易出错的是在script里获取原始数据是通过`ctx._source`来实现的。在原数据中取值可以用`ctx._source.fieldA`或者`ctx._source['fieldA']`两种方式，但是对于非字符串数据可能需要通过`ctx._source.fieldA.value`等方式来获取它的真实值。
  1. [参考链接-scripting](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-scripting.html)，[参考链接-how-to-use-scripts](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-scripting-using.html)，[参考链接-accessing-document-fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-scripting-fields.html)
  2. 页面路径-scripting：Scripting
  3. 页面路径-how-to-use-scripts：Scripting =》 How to use scripts
  4. 页面路径-accessing-document-fieldds：Scripting =》 Accessing document fields and special variables

## 第3题，通过pipeline处理数据

1. Create a pipeline named `split_act_scene_line`. The pipeline splits the value of `line_number` using the dots as a separator, and stores the split values into three new fields named `number_act`, `number_scene`, and `number_line`, respectively
   1. 创建一个叫`split_act_scene_line`的pipeline，它需要能用"."做分隔符把`line_number`拆成3个字段，分别叫`number_act`, `number_scene` 和 `number_line`。
1. To verify that an ingest pipeline works as expected, you can rely on the `_simulate` pipeline API. Test the pipeline on the following document:
   1. 你可以用下面的数据通过`_simulate`API来校验你的pipeline是否满足条件
      ```json
      {
        "_source": {
          "line_number": "1.2.3"
        }
      }
      ```
1. Satisfied with the outcome? Go update your documents, then! Update all documents in `hamlet-new` by using the `split_act_scene_line` pipeline
   1. 结果还不错吧？那去把你到文档更新了吧！通过`split_act_scene_line`把`hamlet-new`中所有的文档都更新掉

## 第3题，题解

1. 创建pipeline
   1. 数据拆分的pipeline
      ```json
      {
        "split": {
          "if": "null != ctx.line_number && ctx.line_number.indexOf('.') > 0",
          "field": "line_number",
          "target_field": "line_number_array",
          "separator": "\\."
        }
      }
      ```
   2. 数据赋值的几种pipeline
      1. 从list/数组里依次取
          ```json
          {
            "set": {
              "field": "number_act",
              "value": "{{line_number.0}}"
            }
          },
          {
            "set": {
              "field": "number_scene",
              "value": "{{line_number.1}}"
            }
          },
          {
            "set": {
              "field": "number_line",
              "value": "{{line_number.2}}"
            }
          }
          ```
      1. 通过script直接赋值
          ```json
          {
            "script": {
              "source": """
                ctx.a1 = ctx.line_number.0;
                ctx.a2 = ctx.line_number.1;
                ctx.a3 = ctx.line_number.2;
              """
            }
          }
          ```
   4. 完整的pipeline（之一）
      ```bash
      PUT _ingest/pipeline/split_act_scene_line
      {
        "description": "split 'line_number' by '.' into 3 fields: 'number_act', 'number_scene' and 'number_line'",
        "processors": [
          {
            "split": {
              "if": "null != ctx.line_number && ctx.line_number.indexOf('.') > 0",
              "field": "line_number",
              "target_field": "line_number_array",
              "separator": "\\."
            }
          },
            {
              "script": {
                "source": """
                ArrayList array = ctx.line_number_array;
                if (null == array || 3 > array.length) { return; }
                ctx.number_act = array[0];
                ctx.number_scene = array[1];
                ctx.number_line = array[2];
                """
              }
            }
        ]
      }
      ```

1. 校验数据
   1. 完整写法
      ```bash
      POST _ingest/pipeline/_simulate
      {
        "docs": [
          {
            "_id": "1",
            "_source": {
              "line_number": "1.2.3"
            }
          },
          {
            "_id": "2",
            "_source": {
              "line_number": "1.2"
            }
          },
          {
            "_id": "3",
            "_source": {
              "line_number": "1"
            }
          }
        ],
        "pipeline": {
          "processors": [
            {
              "split": {
                "if": "null != ctx.line_number && ctx.line_number.indexOf('.') > 0",
                "field": "line_number",
                "target_field": "line_number_array",
                "separator": "\\."
              }
            },
            {
              "script": {
                "source": """
                ArrayList array = ctx.line_number_array;
                if (null == array || 3 > array.length) { return; }
                ctx.number_act = array[0];
                ctx.number_scene = array[1];
                ctx.number_line = array[2];
              """
              }
            }
          ]
        }
      }
      ```

      * 返回值
      ```json
      {
        "docs" : [
          {
            "doc" : {
              "_index" : "_index",
              "_type" : "_doc",
              "_id" : "1",
              "_source" : {
                "line_number_array" : [
                  "1",
                  "2",
                  "3"
                ],
                "line_number" : "1.2.3",
                "number_act" : "1",
                "number_scene" : "2",
                "number_line" : "3"
              },
              "_ingest" : {
                "timestamp" : "2020-11-24T02:10:35.848893Z"
              }
            }
          },
          {
            "doc" : {
              "_index" : "_index",
              "_type" : "_doc",
              "_id" : "2",
              "_source" : {
                "line_number_array" : [
                  "1",
                  "2"
                ],
                "line_number" : "1.2"
              },
              "_ingest" : {
                "timestamp" : "2020-11-24T02:10:35.848897Z"
              }
            }
          },
          {
            "doc" : {
              "_index" : "_index",
              "_type" : "_doc",
              "_id" : "3",
              "_source" : {
                "line_number" : "1"
              },
              "_ingest" : {
                "timestamp" : "2020-11-24T02:10:35.848899Z"
              }
            }
          }
        ]
      }
      ```
   2. 由于我们之前已经把pipeline保存到集群里了，可以直接通过name调用，所以有个简单写法如下
      ```bash
      POST _ingest/pipeline/_simulate
      {
        "docs": [
          {
            "_id": "1",
            "_source": {
              "line_number": "1.2.3"
            }
          },
          {
            "_id": "2",
            "_source": {
              "line_number": "1.2"
            }
          },
          {
            "_id": "3",
            "_source": {
              "line_number": "1"
            }
          }
        ],
        "pipeline": {
          "processors": [
            {
              "pipeline": {
                "name": "split_act_scene_line"
              }
            }
          ]
        }
      }
      ```

      * 返回值和上面一种写法一致
      ```json
      {
        "docs" : [
          {
            "doc" : {
              "_index" : "_index",
              "_type" : "_doc",
              "_id" : "1",
              "_source" : {
                "line_number_array" : [
                  "1",
                  "2",
                  "3"
                ],
                "line_number" : "1.2.3",
                "number_act" : "1",
                "number_scene" : "2",
                "number_line" : "3"
              },
              "_ingest" : {
                "timestamp" : "2020-11-24T02:13:19.631722Z"
              }
            }
          },
          {
            "doc" : {
              "_index" : "_index",
              "_type" : "_doc",
              "_id" : "2",
              "_source" : {
                "line_number_array" : [
                  "1",
                  "2"
                ],
                "line_number" : "1.2"
              },
              "_ingest" : {
                "timestamp" : "2020-11-24T02:13:19.631727Z"
              }
            }
          },
          {
            "doc" : {
              "_index" : "_index",
              "_type" : "_doc",
              "_id" : "3",
              "_source" : {
                "line_number" : "1"
              },
              "_ingest" : {
                "timestamp" : "2020-11-24T02:13:19.631729Z"
              }
            }
          }
        ]
      }
      ```

1. 更新索引里所有数据
    ```bash
    POST hamlet-new/_update_by_query?pipeline=split_act_scene_line
    ```

    * 返回值
    ```json
    {
      "took" : 144,
      "timed_out" : false,
      "total" : 8,
      "updated" : 8,
      "deleted" : 0,
      "batches" : 1,
      "version_conflicts" : 0,
      "noops" : 0,
      "retries" : {
        "bulk" : 0,
        "search" : 0
      },
      "throttled_millis" : 0,
      "requests_per_second" : -1.0,
      "throttled_until_millis" : 0,
      "failures" : [ ]
    }
    ```

1. 校验结果，`GET hamlet-new/_search`
      * 返回值
      ```json
      {
        "took" : 2,
        "timed_out" : false,
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "skipped" : 0,
          "failed" : 0
        },
        "hits" : {
          "total" : {
            "value" : 8,
            "relation" : "eq"
          },
          "max_score" : 1.0,
          "hits" : [
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "1",
              "_score" : 1.0,
              "_source" : {
                "text_entry" : "Nay, answer me: stand, and unfold yourself.",
                "line_number_array" : [
                  "1",
                  "1",
                  "2"
                ],
                "reindexBath" : 1,
                "line_number" : "1.1.2",
                "speaker" : "FRANCISCO",
                "number_act" : "1",
                "number_scene" : "1",
                "number_line" : "2"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "5",
              "_score" : 1.0,
              "_source" : {
                "text_entry" : "I will, my lord.",
                "line_number_array" : [
                  "2",
                  "1",
                  "2"
                ],
                "reindexBath" : 1,
                "line_number" : "2.1.2",
                "speaker" : "REYNALDO",
                "number_act" : "2",
                "number_scene" : "1",
                "number_line" : "2"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "2",
              "_score" : 1.0,
              "_source" : {
                "text_entry" : "Long live the king!",
                "line_number_array" : [
                  "1",
                  "1",
                  "3"
                ],
                "reindexBath" : 1,
                "line_number" : "1.1.3",
                "speaker" : "BERNARDO",
                "number_act" : "1",
                "number_scene" : "1",
                "number_line" : "3"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "6",
              "_score" : 1.0,
              "_source" : {
                "text_entry" : "You shall do marvellous wisely, good Reynaldo,",
                "line_number_array" : [
                  "2",
                  "1",
                  "3"
                ],
                "reindexBath" : 1,
                "line_number" : "2.1.3",
                "speaker" : "LORD POLONIUS",
                "number_act" : "2",
                "number_scene" : "1",
                "number_line" : "3"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "3",
              "_score" : 1.0,
              "_source" : {
                "text_entry" : "Though yet of Hamlet our dear brothers death",
                "line_number_array" : [
                  "1",
                  "2",
                  "1"
                ],
                "reindexBath" : 1,
                "line_number" : "1.2.1",
                "speaker" : "KING CLAUDIUS",
                "number_act" : "1",
                "number_scene" : "2",
                "number_line" : "1"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "7",
              "_score" : 1.0,
              "_source" : {
                "text_entry" : "Before you visit him, to make inquire",
                "line_number_array" : [
                  "2",
                  "1",
                  "4"
                ],
                "reindexBath" : 1,
                "line_number" : "2.1.4",
                "speaker" : "LORD POLONIUS",
                "number_act" : "2",
                "number_scene" : "1",
                "number_line" : "4"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "0",
              "_score" : 1.0,
              "_source" : {
                "text_entry" : "Whos there?",
                "line_number_array" : [
                  "1",
                  "1",
                  "1"
                ],
                "reindexBath" : 1,
                "line_number" : "1.1.1",
                "speaker" : "BERNARDO",
                "number_act" : "1",
                "number_scene" : "1",
                "number_line" : "1"
              }
            },
            {
              "_index" : "hamlet-new",
              "_type" : "_doc",
              "_id" : "4",
              "_score" : 1.0,
              "_source" : {
                "text_entry" : "Give him this money and these notes, Reynaldo.",
                "line_number_array" : [
                  "2",
                  "1",
                  "1"
                ],
                "reindexBath" : 1,
                "line_number" : "2.1.1",
                "speaker" : "LORD POLONIUS",
                "number_act" : "2",
                "number_scene" : "1",
                "number_line" : "1"
              }
            }
          ]
        }
      }
      ```

## 第3题，题解说明

* 这一题主要考察的是各种pipeline的使用，需要注意的是，在考试时可能不需要过多的在意数据的归整形，但是在实际生产中可能会存在数据缺失、数据不规则、数据错误等，这就需要在设计pipeline等时候配合`if`参数以及`on_failure`和`ignore_failure`之类的公共参数了。
  * 本题涉及的pipeline包含拆分（`split`），赋值（`set`），脚本（`script`）和管道（`pipeline`）
    * 其中`split`中使用的分隔符和Java里一样，是需要转译的[.] => [\\.]
    * 对于数组/arraylist里的值，在ES里可以直接通过`list.$idx`的方式进行取值，但是要小心可能这个数组是空，或者因为数组不够长导致的index越界
    * ES里的脚本有个类似语法糖的存在，就是当脚本太长了，可以不用使用一对双引号框起来，而是用一对三个连续的双引号框起来一段话
  * 以及用来测试pipeline用的`_simulate`接口
  1. [参考链接-split](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/split-processor.html)，[参考链接-set](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/set-processor.html)，[参考链接-script](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/script-processor.html)，[参考链接-pipeline](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/pipeline-processor.html)，[参考链接-pipeline-simulate](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/simulate-pipeline-api.html)
  2. 页面路径-split：Ingest node =》 Processors =》 Split Processor
  3. 页面路径-set：Ingest node =》 Processors =》 Set Processor
  4. 页面路径-script：Ingest node =》 Processors =》 Script Processor
  5. 页面路径-pipeline：Ingest node =》 Processors =》 Pipeline Processor
  6. 页面路径-pipeline-simulate：Ingest node =》 Ingest APIs => Simulate Pipeline API
