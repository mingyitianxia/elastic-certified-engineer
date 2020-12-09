## EXAM OBJECTIVE: QUERIES
考点：queries

## GOAL: Create search queries for terms, numbers, dates, fuzzy, and 
考试目标：构建terms、数字、日期、模糊匹配及符合查询语句

## REQUIRED SETUP: 
初始化步骤：
建议docker-compose文件：`1e1k_base_cluster.yml`

1. a running Elasticsearch cluster with at least one node and a Kibana instance,
   1. 运行一个至少含有1ES节点1Kibana节点的集群
2. add the "Sample web logs" and "Sample flight data" to Kibana
   1. 在Kibana里添加"Sample web logs" 和 "Sample flight data"两个示例数据集
3. Run the next queries on the `kibana_sample_data_logs` index
   1. 在`kibana_sample_data_logs`索引中运行下面的搜索语句

## 初始化

1. 搭建集群，`docker-compose -f 1e1k_base_cluster.yml up -d --build`
2. 添加数据：
   1. Kibana点左上角Kibana图标回到首页
   2. 点右侧最上面一栏 `Add Data to Kibana`（中文大概是“为Kibana添加数据”，没有中文版可以测试，具体的要看具体翻译）
   3. 点最右边一个栏目 `Sample data` （中文大概是“样例数据”）
   4. 点第一第三个示例数据的 `Add data`
3. 校验数据，`GET _cat/indices`
   * 如果出现`kibana_sample_data_flights` 和 `kibana_sample_data_logs`，代表添加成功
    ```bash
    green  open kibana_sample_data_flights   TxLrY4R4RB2wRcNsw5bQ9Q 1 0 13059 0  6.5mb  6.5mb
    green  open kibana_sample_data_ecommerce -BmN-n3MRgOdtIrINDeufw 1 0  4675 0  4.9mb  4.9mb
    green  open kibana_sample_data_logs      CfNuYq1kTvelLLVG0T6biA 1 0 14074 0 11.8mb 11.8mb
    ```

## 第1题，构建搜索语句

1. Filter documents with the `response` field greater or equal to 400 and less than 500
   1. 筛选出`response`大于等于400而且小于等于500的文档
2. As above, but add a second filter for documents with the `referer` field matching "http://twitter.com/success/guion-bluford"
   1. 接上个query，加上第二个筛选，`referer` 字段需要匹配 "http://twitter.com/success/guion-bluford"
3. Filter documents with the `referer` field that starts by "http://twitter.com/success"
   1. 筛选出`referer` 字段以 "http://twitter.com/success" 开头的文档
4. Filter documents with the `request` field that starts by "/people"
   1. 筛选出 `request` 字段以 "/people" 开头的文档
5. Filter documents with the `memory` field containing any indexed value
   1. 筛选出包含 `memory` 字段的文档
6. (opposite of above) Filter documents with the `memory` field not containing any indexed value
   1. （和上一个相反）筛选出不包含 `memory` 字段的文档
7. Search for documents with the `agent` field containing the string "Windows" and the `url` field containing the string "name:john"
   1. 搜索 `agent` 字段包含 "Windows" 而且 `url` 字段包含 "name:john" 的文档
8.  As above, but also filter documents with the `phpmemory` field containing any indexed value
    1.  接上个query，但是过滤出 `phpmemory` 存在的文档
9.  Search for documents that have either the `response` field greater or equal to 400 or the `tags` field having the string "error"
    1.  搜索 `response` 字段大于等于 400 或者 `tags` 字段包含 "error" 的文档
10. Search for documents with the `tags` field that does not contain any of the following strings: "warning", "error", "info"
    1.  搜索 `tags` 不包含 "warning", "error", "info" 这三个任意一个字符串的文档
11. Filter documents with the `timestamp` field containing a date between today and one week ago
    1.  筛选出 `timestamp` 包含的日期在1周以前到现在的时间区间里

## 第1题，题解

1. 筛选 `response` 满足 [400, 500] 的文档
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "filter": {
            "range": {
              "response": {
                "gte": 400,
                "lte": 500
              }
            }
          }
        }
      }
    } 
    ```

    * 计数
    ```json
    {
      "took" : 6,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 801,
          "relation" : "eq"
        },
        "max_score" : 0.0,
        "hits" : [
        ]
      }
    }
    ```

1. 上一个query里加上`referer` 字段需要匹配 "http://twitter.com/success/guion-bluford"的筛选
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "filter": [
              {
              "range": {
                "response": {
                  "gte": 400,
                  "lte": 500
                }
              }
            },
            {
              "match": {
                "referer": "http://twitter.com/success/guion-bluford"
              }
            }
          ]
        }
      }
    } 
    ```

    * 计数
    ```json
    {
      "took" : 7,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 1,
          "relation" : "eq"
        },
        "max_score" : 0.0,
        "hits" : [
        ]
      }
    }
    ```

1. 筛选 `referer` 以 "http://twitter.com/success" 开头
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "filter": {
            "prefix": {
              "referer": "http://twitter.com/success"
            }
          }
        }
      }
    }
    ```

   * 计数
    ```json
    {
      "took" : 13,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 3584,
          "relation" : "eq"
        },
        "max_score" : 0.0,
        "hits" : [
        ]
      }
    }
    ```

1. 筛选 `request` 以 "/people" 开头
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "filter": {
            "prefix": {
              "request.keyword": "/people"
            }
          }
        }
      }
    }
    ```

   * 计数
   ```json
   {
     "took" : 4,
     "timed_out" : false,
     "_shards" : {
       "total" : 1,
       "successful" : 1,
       "skipped" : 0,
       "failed" : 0
     },
     "hits" : {
       "total" : {
         "value" : 452,
         "relation" : "eq"
       },
       "max_score" : 0.0,
       "hits" : [
       ]
     }
   }
   ```

1. 筛选包含 `memory` 的文档
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "filter": {
            "exists": {
              "field": "memory"
            }
          }
        }
      }
    }
    ```

   * 计数
   ```json
   {
     "took" : 0,
     "timed_out" : false,
     "_shards" : {
       "total" : 1,
       "successful" : 1,
       "skipped" : 0,
       "failed" : 0
     },
     "hits" : {
       "total" : {
         "value" : 552,
         "relation" : "eq"
       },
       "max_score" : 0.0,
       "hits" : [
       ]
     }
   }
   ```

1. 筛选不包含 `memory` 的文档
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must_not": {
            "exists": {
              "field": "memory"
            }
          }
        }
      }
    }
    ```

   * 计数
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
         "value" : 10000,
         "relation" : "gte"
       },
       "max_score" : 0.0,
       "hits" : [
       ]
     }
   }
   ```

1. 搜索 `agent` 包含 "Windows" 而且 `url` 字段包含 "name:john"
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "agent": "Windows"
              }
            },
            {
              "match": {
                "url": "name:john"
              }
            }
          ]
        }
      }
    }
    ```

   * 计数
   ```json
   {
     "took" : 1,
     "timed_out" : false,
     "_shards" : {
       "total" : 1,
       "successful" : 1,
       "skipped" : 0,
       "failed" : 0
     },
     "hits" : {
       "total" : {
         "value" : 3,
         "relation" : "eq"
       },
       "max_score" : 7.5268917,
       "hits" : [
       ]
     }
   }
   ```

1. 接上，但是过滤出 `phpmemory` 存在的文档
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "agent": "Windows"
              }
            },
            {
              "match": {
                "url": "name:john"
              }
            }
          ],
          "filter": {
            "exists": {
              "field": "phpmemory"
            }
          }
        }
      }
    }
    ```

   * 计数
   ```json
   {
     "took" : 6,
     "timed_out" : false,
     "_shards" : {
       "total" : 1,
       "successful" : 1,
       "skipped" : 0,
       "failed" : 0
     },
     "hits" : {
       "total" : {
         "value" : 3,
         "relation" : "eq"
       },
       "max_score" : 7.5268917,
       "hits" : [
       ]
     }
   }
   ```

1. 搜索 `response` 大于等于 400 或者 `tags` 包含 "error" 
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "should": [
            {
              "range": {
                "response": {
                  "gte": 400
                }
              }
            },
            {
              "match": {
                "tags": "error"
              }
            }
          ]
        }
      }
    }
    ```

   * 计数
   ```json
   {
     "took" : 10,
     "timed_out" : false,
     "_shards" : {
       "total" : 1,
       "successful" : 1,
       "skipped" : 0,
       "failed" : 0
     },
     "hits" : {
       "total" : {
         "value" : 2003,
         "relation" : "eq"
       },
       "max_score" : 3.8313324,
       "hits" : [
       ]
     }
   }
   ```

1. 搜索 `tags` 不包含 "warning", "error", "info" 
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must_not": [
            {
              "match": {
                "tags": "warning"
              }
            },
            {
              "match": {
                "tags": "error"
              }
            },
            {
              "match": {
                "tags": "info"
              }
            }
          ]
        }
      }
    }
    ```
    或者
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must_not": [
            {
              "terms": {
                "tags": ["warning", "error", "info"]
              }
            }
          ]
        }
      }
    }
    ```

     * 计数
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
           "value" : 2927,
           "relation" : "eq"
         },
         "max_score" : 0.0,
         "hits" : [
         ]
       }
     }
     ```

1. 筛选出 `timestamp` 包含的日期在1周以前到现在的时间区间里
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "range": {
          "timestamp": {
            "gte": "now-7d/d",
            "lte": "now/d"
          }
        }
      }
    }
    ```

   * 计数
   ```json
   {
     "took" : 4,
     "timed_out" : false,
     "_shards" : {
       "total" : 1,
       "successful" : 1,
       "skipped" : 0,
       "failed" : 0
     },
     "hits" : {
       "total" : {
         "value" : 1840,
         "relation" : "eq"
       },
       "max_score" : 1.0,
       "hits" : [
       ]
     }
   }
   ```

## 第1题，题解说明

* 这题主要考察的是通过`bool` 配合 `filter`，`must`，`should`，`must_not`等关键字进行多检索条件的逻辑计算召回，以及 `prefix`，`exists`，`match`，`range`，`terms` 等搜索关键字的使用
  * 这里可能会有一个大坑是 `prefix` 关键字只能作用在 `keyword` 字段上，所以上面的题解里会有 `prefix: referer` 和 `prefix: request.keyword` 的区别
  * 在实际生产中可能会存在需要判断字段的值存不存在和字段存不存在的情况，判断字段存不存在可以用 `exists` 而判断字段的值存不存在可能需要搭配 script 对字段值长度进行判断，或者通过 `null_value` 等方式
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must": {
            "script": {
              "script": {
                "source": "String message = doc['message.keyword'].value; return (null != message && 0 < message.length())"
              }
            }
          }
        }
      }
    }
    ```
  1. [参考链接-bool](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-bool-query.html)，[参考链接-prefix](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-prefix-query.html)，[参考链接-exists](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-exists-query.html)，[参考链接-match](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-match-query.html)，[参考链接-range](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-range-query.html)，[参考链接-terms](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-terms-query.html)，
  2. 页面路径-bool：Query DSL =》 Compound queries =》 Boolean
  3. 页面路径-prefix：Query DSL =》 Term-level queries =》 Prefix
  4. 页面路径-exists：Query DSL =》 Term-level queries =》 Exists
  5. 页面路径-match：Query DSL =》 Full text queries =》 Match
  6. 页面路径-prefix：Query DSL =》 Term-level queries =》 Range
  7. 页面路径-prefix：Query DSL =》 Term-level queries =》 Terms

## 第2题，模糊匹配

1. Run the next queries on the `kibana_sample_data_flights` index
   1. 下面的query请在 `kibana_sample_data_flights` 索引上执行
2. Filter documents with either the `OriginCityName` or the `DestCityName` fields matching the string "Sydney"
   1. 筛选出 `OriginCityName` 或者 `DestCityName`  字段里包含 "Sydney" 的文档
3. As above, but allow inexact fuzzy matching, with a maximum allowed “Levenshtein Edit Distance” set to 2. Test that the query strings "Sydney", "Sidney" and "Sidnei" always return the same number of results
   1. 如上，但是加上模糊匹配，把 “莱文施泰因编辑距离” （`Levenshtein Edit Distance`）设为2。测试一下当query是"Sydney", "Sidney" 和 "Sidnei" 时返回的结果条数一样。

## 第2题，题解

1. 筛选`OriginCityName` 或 `DestCityName`  包含 "Sydney"
    ```bash
    POST kibana_sample_data_flights/_search
    {
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                "OriginCityName": "Sydney"
              }
            },
            {
              "match": {
                "DestCityName": "Sydney"
              }
            }
          ]
        }
      }
    }
    ```
1. 加模糊匹配，调整编辑距离
   1. Sydney
      ```bash
      POST kibana_sample_data_flights/_search
      {
        "query": {
          "bool": {
            "should": [
              {
                "fuzzy": {
                "OriginCityName": {
                  "value": "Sydney",
                  "fuzziness": "2"
                } 
                }
              },
              {
                "fuzzy": {
                "DestCityName": {
                  "value": "Sydney",
                  "fuzziness": "2"
                } 
                }
              }
            ]
          }
        }
      }
      ```

      * 计数
      ```json
      {
        "took" : 200,
        "timed_out" : false,
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "skipped" : 0,
          "failed" : 0
        },
        "hits" : {
          "total" : {
            "value" : 405,
            "relation" : "eq"
          },
          "max_score" : 8.344088,
          "hits" : [
          ]
        }
      }
      ```

  1. Sidney
      ```bash
      POST kibana_sample_data_flights/_search
      {
        "query": {
          "bool": {
            "should": [
              {
                "fuzzy": {
                "OriginCityName": {
                  "value": "Sidney",
                  "fuzziness": "2"
                } 
                }
              },
              {
                "fuzzy": {
                "DestCityName": {
                  "value": "Sidney",
                  "fuzziness": "2"
                } 
                }
              }
            ]
          }
        }
      }
      ```

      * 计数
      ```json
      {
        "took" : 11,
        "timed_out" : false,
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "skipped" : 0,
          "failed" : 0
        },
        "hits" : {
          "total" : {
            "value" : 405,
            "relation" : "eq"
          },
          "max_score" : 6.9534063,
          "hits" : [
          ]
        }
      }
      ```

  1. Sidnei
      ```bash
      POST kibana_sample_data_flights/_search
      {
        "query": {
          "bool": {
            "should": [
              {
                "fuzzy": {
                "OriginCityName": {
                  "value": "Sidnei",
                  "fuzziness": "2"
                } 
                }
              },
              {
                "fuzzy": {
                "DestCityName": {
                  "value": "Sidnei",
                  "fuzziness": "2"
                } 
                }
              }
            ]
          }
        }
      }
      ```

      * 计数
      ```json
      {
        "took" : 5,
        "timed_out" : false,
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "skipped" : 0,
          "failed" : 0
        },
        "hits" : {
          "total" : {
            "value" : 405,
            "relation" : "eq"
          },
          "max_score" : 5.5627246,
          "hits" : [
          ]
        }
      }
      ```

## 第2题，题解说明

* 这题主要考察的是`fuzzy`的使用
  * 在`fuzzy`里ES会尝试对原query 进行一定的改写以尝试对可能的错误拼写进行模糊匹配（类似纠错功能）
  * `fuzzy`中可以对 [编辑距离](https://en.wikipedia.org/wiki/Levenshtein_distance)、改写长度等进行限制，从而平衡召回率和准确率
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-fuzzy-query.html)
  2. 页面路径：Query DSL =》 Term-level queries =》 Fuzzy
