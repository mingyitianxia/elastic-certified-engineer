## INDEX TEMPLATE
索引模板

## GOAL: build index template and index some documents
目标：按要求创建索引模板并存入数据

建议docker-compose文件：`1e1k_base_cluster.yml`

## 第1题，按要求创建索引模板
1. Create the index template `hamlet_template`, so that the template:
   1. 创建一个叫`hamlet_template`的索引模板，满足以下要求
   2. matches any index that starts by "hamlet_" or "hamlet-", 
      1. 能匹配索引名称以"hamlet_" 或者 "hamlet-" 开头的索引
   3. allocates one primary shard and no replicas for each matching index 
      1. 每个满足条件的索引都有1分片0副本
2. Create the indices `hamlet2` and `hamlet_test`
   1. 创建索引`hamlet2` 和 `hamlet_test`
3. Verify that only `hamlet_test` applies the settings defined in `hamlet_template`
   1. 校验一下只有`hamlet_test`能满足我们在`hamlet_template`中的设定

## 第1题，题解
1. 创建索引模板
    ```bash
    PUT _teplate/hamlet_template
    {
      "index_patterns": ["hamlet_*", "hamlet-"],
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      }
    }
    ```
1. 创建索引
    ```bash
    PUT hamlet2
    PUT hamlet_test
    ```
1. 校验
    1. `GET hamlet2`
        ```json
        {
          "hamlet2" : {
            "aliases" : { },
            "mappings" : { },
            "settings" : {
              "index" : {
                "creation_date" : "1605671989488",
                "number_of_shards" : "1",
                "number_of_replicas" : "1",
                "uuid" : "V1vwGlKwRgaVTKOqPDKr-A",
                "version" : {
                  "created" : "7020199"
                },
                "provided_name" : "hamlet2"
              }
            }
          }
        }
        ```

    1. `GET hamlet_test`
        ```json
        {
          "hamlet_test" : {
            "aliases" : { },
            "mappings" : { },
            "settings" : {
              "index" : {
                "creation_date" : "1605671990921",
                "number_of_shards" : "1",
                "number_of_replicas" : "0",
                "uuid" : "IQWAOtrxSVCi9sns-K7vyw",
                "version" : {
                  "created" : "7020199"
                },
                "provided_name" : "hamlet_test"
              }
            }
          }
        }
        ```

## 第1题，题解说明
* 这题主要考察索引模板的建立和索引模式匹配
  * 索引名模式匹配的key是带复数形式的，因为它的值是个数组`index_patterns`
  * 索引名的匹配是支持通配符的，所以一般我们会使用`?` 或者 `*` 来创建匹配模式
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html)
  2. 页面路径：Indices APIs =》 Index Templates

## 第2题，通过索引模板设置mapping结构

1. Update `hamlet_template` by defining a mapping for the type "_doc", so that
   1. 更新索引模板`hamlet_template`，使它可以让`_doc`这type的mappping结构满足以下需求
   2. the type has three fields, named `speaker`,   `line_number`, and `text_entry`
      1. 它有三个字段，分别叫`speaker`,   `line_number`, 和 `text_entry`
   3. `text_entry` uses an "english" analyzer
      1. 其中`text_entry`字段使用“english”分词器
2. Verify that the updates in `hamlet_template` did not apply to the existing indices 
   1. 校验一下这个对`hamlet_template`的更新不会影响到已经存在的索引
3. In one request, delete both `hamlet2` and `hamlet_test` 
   1. 在一个请求中删除`hamlet2` 和 `hamlet_test` 两个索引
4. Create the index `hamlet-1` and add some documents by running the following _bulk command
   1. 创建一个叫`hamlet-1`的索引并用下面的`_bulk`命令给他插入一些数据
      ```bash
      POST _bulk
      {"index":{"_index":"hamlet-1","_id":0}}
      {"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
      {"index":{"_index":"hamlet-1","_id":1}}
      {"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
      {"index":{"_index":"hamlet-1","_id":2}}
      {"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
      {"index":{"_index":"hamlet-1","_id":3}}
      {"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
      ```
5. Verify that the mapping of `hamlet-1` is consistent with what defined in `hamlet_template`
   1. 校验一下`hamlet-1`的mapping结构是否满足`hamlet_template`的设定

## 第2题，题解

1. 更新索引模板
    ```bash
    PUT _template/hamlet_template
    {
      "settings": {
          "number_of_shards": 1,
          "number_of_replicas": 0
      },
      "mappings": {
          "properties": {
            "text_entry": {
                "type": "text",
                "analyzer": "english"
            },
            "speaker": {
                "type": "text"
            },
            "line_number": {
                "type": "text"
            }
          }
      },
      "index_patterns": [
          "hamlet_*",
          "hamlet-*"
      ]
    }
    ```
   1. 校验已有索引
      1. `GET hamlet2`
      2. `GET hamlet_test`
2. 删除索引
   1. `DELETE hamlet2,hamlet_test`
3. 创建索引插数据
   1. `PUT hamlet-1`
   2. 运行上面 ⬆️ 那些bulk命令
   3. 校验索引 `GET hamlet-1`
   4. 校验数据 `POST hamlet-1/_search`

## 第2题，题解说明

* 这题主要考察的是索引模板在索引创建中的作用
  * 比较重要的是
    * `index_patterns`不要拼错，里面的索引匹配语句别写错
    * `settings`和`mappings`会在建立索引的时候直接作用在索引里
    * 多个`index_patterns`甚至多个索引模板的pattern可能会匹配到同一个索引上，这里可以通过`order`参数调整执行顺序，数字越小的越先执行，数字大的配置可能会覆盖/merge数字小的模板的配置
  1. [参考链接-index-template](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html)
  2. 页面路径-index-template：Indices APIs =》 Index Templates

* 删除索引不是这题的重点，题目中提到要用一个命令来删掉两个索引，需要意的是：
  * 如果集群配置了不允许通配符执行的话`action.destructive_requires_name: true`就不能直接`DELETE hamlet*`来删索引了
  * 如果强行用`DELETE hamlet*`删索引的话，要确保这个匹配语句不会匹配到其他索引
    * 可以通过`GET hamlet*`命令先查看一下这个匹配语句能命中哪些索引
  1. [参考链接-delete-index](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-delete-index.html)，[参考链接-destructive_requires_name](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/index-management-settings.html)
  2. 页面路径-delete-index：Indices APIs =》 Delete Index
  3. 页面路径-destructive_requires_name：Set up Elasticsearch =》 Configuring Elasticsearch =》 Index management settings （这条配置在7.2版没有具体页面，所以这里使用的是7.9版的网页）

## 第3题，动态模板

1. Update `hamlet_template` so as to reject any document having a field that is not defined in the mapping 
   1. 更新模板`hamlet_template`使它拒绝索引那些没被定义的字段
2. Verify that you cannot index the following document in `hamlet-1` 
   1. 校验一下你不能在`hamlet-1`里索引这个文档
      ```bash
      POST hamlet-1/_doc/99
      { 
        "author": "Shakespeare" 
      } 
      ```
3. Update `hamlet_template` so as to enable dynamic mapping again
   1. 更新`hamlet_template`，重新开启动态索引功能
4. Update `hamlet_template` so as to: 
   1. 更新`hamlet_template`使得它可以应对以下情形
   2. dynamically map to an integer any field that starts by "number_",
      1. 动态索引所有一"number_"开头的数据，把他们设定为"integer"型的索引
   3. dynamically map to unanalysed text any string field
      1. 动态索引所有字符串（string）型的数据，让他们不被分词
5. Create the index `hamlet-2` and add a document by running the following command
   1. 创建一个叫`hamlet-2`的索引，并通过下面这些命令来添加一些数据
      ```bash
      POST hamlet-2/_doc/1
      {
        "text_entry": "With turbulent and dangerous lunacy?",
        "line_number": "3.1.4",
        "number_act": "3",
        "speaker": "KING CLAUDIUS"
      }
      ```
6. Verify that the mapping of `hamlet-2` is consistent with what defined in `hamlet_template`
   1. 校验一下`hamlet-2`里的字段是不是按照`hamlet_template`里的设置正确的索引了

## 第3题，题解

1. 更新模板，并使之不再支持动态索引
    ```bash
    PUT _template/hamlet_template
    {
      "index_patterns": [
        "hamlet_*",
        "hamlet-*"
      ],
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      },
      "mappings": {
        "dynamic": "strict"
      }
    }
    ```
1. 插数据并校验
   1. 插数据，命令同上
      * 返回值
      ```json
      {
        "error": {
          "root_cause": [
            {
              "type": "strict_dynamic_mapping_exception",
              "reason": "mapping set to strict, dynamic introduction of [author] within [_doc] is not allowed"
            }
          ],
          "type": "strict_dynamic_mapping_exception",
          "reason": "mapping set to strict, dynamic introduction of [author] within [_doc] is not allowed"
        },
        "status": 400
      }
      ```
   1. 校验索引：`GET hamlet-1`
      * 返回值
      ```json
      {
        "hamlet-1" : {
          "aliases" : { },
          "mappings" : {
            "dynamic" : "strict"
            }
          },
          "settings" : {
            "index" : {
              "creation_date" : "1605765222745",
              "number_of_shards" : "1",
              "number_of_replicas" : "0",
              "uuid" : "CiHizX7-QfqaOs4CpsPTPg",
              "version" : {
                "created" : "7020199"
              },
              "provided_name" : "hamlet-1"
            }
          }
        }
      }
      ```

   2. 校验数据：`GET hamlet-1/_doc/99`
      * 返回值
      ```json
      {
        "_index" : "hamlet-1",
        "_type" : "_doc",
        "_id" : "99",
        "found" : false
      }
      ```

2. 更新模板，开启动态索引，两种方式
   1. 把`dynamic`直接删掉或者设为“true”（这里使用设为“true”的方式）
      ```bash
      PUT _template/hamlet_template
      {
        "index_patterns": [
          "hamlet_*",
          "hamlet-*"
        ],
        "settings": {
          "number_of_shards": 1,
          "number_of_replicas": 0
        },
        "mappings": {
          "dynamic": true
        }
      }
      ```
3. 更新模板使之能处理下面的动态索引需求
    ```bash
    PUT _template/hamlet_template
    {
      "index_patterns": [
        "hamlet_*",
        "hamlet-*"
      ],
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      },
      "mappings": {
        "dynamic": "true",
        "dynamic_templates": [
          {
            "number_fields_as_integers": {
              "match": "number_*",
              "mapping": {
                "type": "integer"
              }
            }
          },
          {
            "string_fields": {
              "match_mapping_type": "string",
              "mapping": {
                "type": "keyword"
              }
            }
          }
        ]
      }
    }
    ```
1. 插入数据并校验
   1. 插数据命令同上
      * 返回值
      ```json
      {
        "_index" : "hamlet-2",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1
      }
      ```
   2. 校验索引：`GET hamlet-2`
      * 返回值
      ```json
      {
        "hamlet-2" : {
          "aliases" : { },
          "mappings" : {
            "dynamic" : "true",
            "dynamic_templates" : [
              {
                "number_fields_as_integers" : {
                  "match" : "number_*",
                  "mapping" : {
                    "type" : "integer"
                  }
                }
              },
              {
                "string_fields" : {
                  "match_mapping_type" : "string",
                  "mapping" : {
                    "type" : "keyword"
                  }
                }
              }
            ],
            "properties" : {
              "line_number" : {
                "type" : "keyword"
              },
              "number_act" : {
                "type" : "integer"
              },
              "speaker" : {
                "type" : "keyword"
              },
              "text_entry" : {
                "type" : "keyword"
              }
            }
          },
          "settings" : {
            "index" : {
              "creation_date" : "1605766333188",
              "number_of_shards" : "1",
              "number_of_replicas" : "0",
              "uuid" : "DZ2DCnqoRSCwy40rncb_aA",
              "version" : {
                "created" : "7020199"
              },
              "provided_name" : "hamlet-2"
            }
          }
        }
      }
      ```
   3. 校验数据：`GET hamlet-2/_doc/1`
      * 返回值
      ```json
      {
        "_index" : "hamlet-2",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 1,
        "_seq_no" : 0,
        "_primary_term" : 1,
        "found" : true,
        "_source" : {
          "text_entry" : "With turbulent and dangerous lunacy?",
          "line_number" : "3.1.4",
          "number_act" : "3",
          "speaker" : "KING CLAUDIUS"
        }
      }
      ```

## 第3题，题解说明

* 这题主要考察索引模板和动态模板
  * 索引模板的变更不会影响到已经存在的索引，所以不管是开/关 `dynamic` 设置还是添加其他字段定义都不会在已经存在的索引中生效
  * `dynamic`为“true”时会拒绝任何包含没被预先设置的文档，但是让它失效有两种方式
    * 把它设为“false”
    * 直接把它删掉
  * 目前看起来这俩方式是等效的，但是不能像其他的设置一样把它设为 “null”，会报以下错误
    ```json
    {
      "error": {
        "root_cause": [
          {
            "type": "mapper_parsing_exception",
            "reason": "Failed to parse mapping [_doc]: null"
          }
        ],
        "type": "mapper_parsing_exception",
        "reason": "Failed to parse mapping [_doc]: null",
        "caused_by": {
          "type": "null_pointer_exception",
          "reason": null
        }
      },
      "status": 400
    }
    ```
  1. [参考链接-index-template](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html)，[参考链接-dynamic-templates](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/dynamic-templates.html)，[参考链接-keyword-datatype](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/keyword.html)，[参考链接-analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analyzer.html)
  2. 页面路径-index-template：Indices APIs =》 Index Templates
  3. 页面路径-dynamic-templates：Mapping =》 Dynamic Mapping =》 Dynamic templates
  4. 页面路径-keyword-datatype：Mapping =》 Field datatypes =》 Keyword
  5. 页面路径-analyzer：Mapping =》 Mapping parameters =》 analyzer
