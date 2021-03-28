## EXAM OBJECTIVE: QUERIES
考点：queries

## GOAL: Create search queries for analyzed text, highlight, pagination, and sort
考试目标：创建搜索语句以对文档进行分析、高亮、分页和排序

## REQUIRED SETUP: 
初始化步骤：
建议docker-compose文件：`1e1k_base_cluster.yml`

1. a running Elasticsearch cluster with at least one node and a Kibana instance,
   1. 运行一个至少含有1ES节点1Kibana节点的集群
2. add the "Sample web logs" and "Sample eCommerce orders" to Kibana
   1. 在Kibana里添加"Sample web logs" 和 "Sample eCommerce orders"两个示例数据集
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
   * 如果出现`kibana_sample_data_ecommerce` 和 `kibana_sample_data_logs`，代表添加成功
    ```bash
    green  open hamlet_1                     5tk876UATbeP6aRbikqaNw 1 0     4 0      4kb      4kb
    green  open .kibana_task_manager         2eUVrIdMTqusMMBTKcymPw 1 0     2 0   21.4kb   21.4kb
    green  open hamlet_2                     6OwlCLIhTg-v526MRvNeIg 1 0     4 0    9.8kb    9.8kb
    green  open hamlet_3                     dOLRjkPLRc65iBrmQIWpRQ 1 0     3 0   48.6kb   48.6kb
    green  open test                         CN_uoHfyRSWv8PE6L8RDaA 1 0     2 0      9kb      9kb
    green  open hamlet-new                   8GRHRUtFRNG59T3Z_6lm4Q 2 0     8 0   13.7kb   13.7kb
    green  open .kibana_1                    lzAn4TPDR_WY7vuJrA8EFA 1 0    88 1 1007.6kb 1007.6kb
    green  open hamlet-1                     G-iX2celQtmnqhx5fBlWww 2 0     4 0    7.5kb    7.5kb
    green  open hamlet-2                     jWqxhIZEQTC_hF_GE7GgeQ 2 0     4 0    7.5kb    7.5kb
    yellow open hamlet-raw                   QLO4rZNgTSKnvPDJuRVLQA 1 3     2 1   19.3kb   19.3kb
    green  open kibana_sample_data_ecommerce -BmN-n3MRgOdtIrINDeufw 1 0  4675 0    4.9mb    4.9mb
    green  open hamlet-new-1                 swvrxEElRdydDsXH5IMa2A 1 0     8 0    8.3kb    8.3kb
    green  open kibana_sample_data_logs      CfNuYq1kTvelLLVG0T6biA 1 0 14074 0   11.8mb   11.8mb
    ```

## 第1题，构建搜索语句

1. Let’s search the `kibana_sample_data_logs` index for logs that contain the string “Firefox” in their message. Because the data type of the message field is an analysed text, the standard query for performing full-text searches is the match query.
   1. 让我们在索引`kibana_sample_data_logs`里搜索 “message” 字段里包含字符串“Firefox”的条目。因为 “message” 字段的数据被索引成了被分词的文本（`analysed text`），比较常见的全文搜索语句就是用 “match“ 命令了
2. Search for documents with the `message` field containing the string "Firefox"
   1. 搜索那些在`message`字段里包含字符串 “Firefox” 的文档
3. What do you think would happen if you searched for “firefox” with a lowercase “f”? Nothing, right, because the standard analyzer applied to the message field will lowercase all tokens anyway. 
   1. 你觉得如果我们用 “firefox” （小写的 “f”开头）搜会发生什么？啥也没有对吧，因为对于标准分词器（`standard analyzer`）默认会对它处理的所有的词元进行小写（`lowercase`）操作（所以其实是可以搜出来东西的）
4. By default, a query response can include up to ten results. But what if you wanted to show up to 50 results? And then what if you wanted to fetch the next 50?
   1. 默认情况下，一条搜索命令会返回最多 10 条结果，但是如果你想搜索最多 50 条怎么办？然后你想再看下 50 条怎么办？
5. Search for documents with the `message` field containing thestring "Firefox" and return (up to) 50 results. 
   1. 搜索 `message` 字段中包含 “Firefox” 的文档，返回（最多） 50 个结果。
6. As above, but return up to 50 results with an offset of 50 from the first
   1. 延续上一条搜索，然后返回最多 50 条结果，但是要加上 50 的偏移量（offset of 50）
7. Deep pagination is highly inefficient when realised using the from and size parameters, as memory consumption and response time grow with the value of the parameters. The best practice is to use the search_after parameter instead.
   1. 深度翻页的效率很低，内存消耗和响应时间会随着翻页参数的增加而上升，最佳实践是用 `search_after` 代替 `from/size`

## 第1题，题解

1. 搜索包含“Firefox”的条目
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "match": {
          "message": "Firefox"
        }
      }
    }
    ```

1. 用“firefox”搜索文档
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "match": {
          "message": "firefox"
        }
      }
    }
    ```

1. 搜 50 条文档
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "from": 0,
      "size": 50,
      "query": {
        "match": {
          "message": "Firefox"
        }
      }
    }
    ```

1. 从第 51 条开始（offset of 50）搜 50条数据
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "from": 50,
      "size": 50,
      "query": {
        "match": {
          "message": "Firefox"
        }
      }
    }
    ```

1. 用 `search_after` 做深度翻页
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "from": 0,
      "size": 50,
      "query": {
        "match": {
          "message": "Firefox"
        }
      },
      "sort": {
        "_id": "desc"
      }
    }
    ```

    * 返回的最后2条
    ```json
    {
      "_index" : "kibana_sample_data_logs",
      "_type" : "_doc",
      "_id" : "zb_2FnYBblECTPDi309x",
      "_score" : null,
      "_source" : {
      }
    },
    {
      "_index" : "kibana_sample_data_logs",
      "_type" : "_doc",
      "_id" : "zb_2FnYBblECTPDi10hj",
      "_score" : null,
      "_source" : {
      }
    }
    ```

    * `search_after`
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "size": 50,
      "query": {
        "match": {
          "message": "Firefox"
        }
      },
      "sort": {
        "_id": "desc"
      },
      "search_after": ["zb_2FnYBblECTPDi309x"]
    }
    ```

    * 返回前2条
    ```json
    {
      "_index" : "kibana_sample_data_logs",
      "_type" : "_doc",
      "_id" : "zb_2FnYBblECTPDi10hj",
      "_score" : null,
      "_source" : {
      }
    },
    {
      "_index" : "kibana_sample_data_logs",
      "_type" : "_doc",
      "_id" : "zL_2FnYBblECTPDiz0DX",
      "_score" : null,
      "_source" : {
      }
    }
    ```

## 第1题，题解说明

* 这题主要考察的是match query，standard analyzer，分页召回和`search_after`
  * match query时，ES在没特别指定时会按字段的分词器处理query语句，然后从文档中找所有包含query里所有的词元里任何一个的文档进行召回计算
  * `standard analyzer`是常规的分词器，一半会按空格和标点符号进行分割，并对所有词元进行转小写处理，不会做特别特异化的词元分析
  * 分页召回主要就是ES按query语句里的size进行数据截取召回排序，再按from + size对最终的结果进行截取返回
  * `search_after`会按之前的排序方式将数据进行排序，然后根据id而不是数据的位置进行翻页
  1. [参考链接-match-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-match-query.html)，[参考链接-standard-analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-standard-analyzer.html)，[参考链接-分页返回](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-from-size.html)，[参考链接-search-after](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/paginate-search-results.html#search-after)
  2. 页面路径-match-query：Query DSL =》 Full text queries =》 Match
  3. 页面路径-standard-analyzer：Analysis =》 Analyzers =》 Standard Analyzer
  4. 页面路径-分页返回：Search APIs =》 Request Body Search =》 From / Size
  5. 页面路径-search-after：Search your data =》 Paginate search results

## 第2题，多条件匹配

1. Search for documents with the `message` field containing the strings "Firefox" or "Kibana"
   1. 在 `message` 字段里搜索包含 “Firefox” 或 “Kibana” 的文档
1. Search for documents with the `message` field containing both the strings "Firefox" and "Kibana" 
   1. 在 `message` 字段里搜索包含 “Firefox” 和 “Kibana” 的文档
2. Search for documents with the `message` field containing at least two of the following strings: "Firefox", "Kibana", "159.64.35.129"
   1. 在 `message` 字段里搜索最少包含2个以下字符串的文档："Firefox", "Kibana", "159.64.35.129"

## 第2题，题解

1. 包含“Firefox” 或 “Kibana”
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                "message": "Firefox"
              }
            },{
              "match": {
                "message": "Kibana"
              }
            }
          ]
        }
      }
    }
    ```
1. 同时包含“Firefox” 和 “Kibana”
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "message": "Firefox"
              }
            },{
              "match": {
                "message": "Kibana"
              }
            }
          ]
        }
      }
    }
    ```
1. 包含至少2个
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                "message": "Firefox"
              }
            },{
              "match": {
                "message": "Kibana"
              }
            },{
              "match": {
                "message": "159.64.35.129"
              }
            }
          ],
          "minimum_should_match": 2
        }
      }
    }
    ```

## 第2题，题解说明

* 这题主要考察的是通过 `bool` 配合 `must`， `should`， `must_not`，等关键字将多个检索条件逻辑关联起来进行搜索
  * `must` 类似 sql 里的 “and” 关键字，是 “与” / “且” 的关系
  * `must_not` 类似 sql 里的 “not” 关键字，是 “非” 的关系
  * `should` 类似 sql 里的 “or” 关键字，是 “或” 的关系
    * `minimum_should_match` 是在 `should` 里最少匹配几个条件的限制字段
  1. [参考链接-bool-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-bool-query.html#score-bool-filter)，[参考链接-minimum_should_match](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-minimum-should-match.html)
  2. 页面路径-bool-query：Query DSL -》 Compound queries =》 Boolean
  3. 页面路径-minimum_should_match：Query DSL =》 minimum_should_match parameter

## 第2题，拓展

* 如果抛开这题考察的 `bool` + `should` 关键字来看，我们用下面的query可能也能得到类似的结果
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "match": {
          "message": "Kibana Firefox"
        }
      }
    }
    ```
  * 因为在 match query 里面，所有词元的关系也是或，只是这样就没法做其他逻辑判断，而且没法指定最少匹配条件个数了

## 第3题，关键字高亮

1. Search for documents with the `message` field containing the strings "Firefox" or "Kibana" 
   1. 在文档中搜索 `message` 字段包含 "Firefox" 或 "Kibana" 的
2. As above, but also return the highlights for the `message` field
   1. 在上面的query结果中，同时返回对 `message` 命中内容的高亮
3. As above, but also wrap the highlights in "{{" and "}}"
   1. 在上面的query 结果中，用 "{{" 和 "}}" 把高亮的结果框起来

## 第3题，题解

1. 搜索内容
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "size": 1,
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                "message": "Firefox"
              }
            },
            {
              "match": {
                "message": "Kibana"
              }
            }
          ]
        }
      }
    }
    ```

    * 返回值
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
          "value" : 6336,
          "relation" : "eq"
        },
        "max_score" : 3.9088674,
        "hits" : [
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "F7_2FnYBblECTPDiz0DL",
            "_score" : 3.9088674,
            "_source" : {
              "agent" : "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1",
              "bytes" : 8489,
              "clientip" : "159.64.35.129",
              "extension" : "gz",
              "geo" : {
                "srcdest" : "ET:BG",
                "src" : "ET",
                "dest" : "BG",
                "coordinates" : {
                  "lat" : 28.03925,
                  "lon" : -97.54244444
                }
              },
              "host" : "artifacts.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "159.64.35.129",
              "machine" : {
                "ram" : 16106127360,
                "os" : "win xp"
              },
              "memory" : null,
              "message" : """159.64.35.129 - - [2018-07-22T06:06:42.742Z] "GET /kibana/kibana-6.3.2-linux-x86_64.tar.gz_1 HTTP/1.1" 200 8489 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/frederick-w-leslie",
              "request" : "/kibana/kibana-6.3.2-linux-x86_64.tar.gz",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2020-11-22T06:06:42.742Z",
              "url" : "https://artifacts.elastic.co/downloads/kibana/kibana-6.3.2-linux-x86_64.tar.gz_1",
              "utc_time" : "2020-11-22T06:06:42.742Z"
            }
          }
        ]
      }
    }
    ```
1. 加高亮
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "size": 1,
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                "message": "Firefox"
              }
            },
            {
              "match": {
                "message": "Kibana"
              }
            }
          ]
        }
      },
      "highlight": {
        "fields": {
          "message": {}
        }
      }
    }
    ```

    * 返回值
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
          "value" : 6336,
          "relation" : "eq"
        },
        "max_score" : 3.9088674,
        "hits" : [
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "F7_2FnYBblECTPDiz0DL",
            "_score" : 3.9088674,
            "_source" : {
              "agent" : "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1",
              "bytes" : 8489,
              "clientip" : "159.64.35.129",
              "extension" : "gz",
              "geo" : {
                "srcdest" : "ET:BG",
                "src" : "ET",
                "dest" : "BG",
                "coordinates" : {
                  "lat" : 28.03925,
                  "lon" : -97.54244444
                }
              },
              "host" : "artifacts.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "159.64.35.129",
              "machine" : {
                "ram" : 16106127360,
                "os" : "win xp"
              },
              "memory" : null,
              "message" : """159.64.35.129 - - [2018-07-22T06:06:42.742Z] "GET /kibana/kibana-6.3.2-linux-x86_64.tar.gz_1 HTTP/1.1" 200 8489 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/frederick-w-leslie",
              "request" : "/kibana/kibana-6.3.2-linux-x86_64.tar.gz",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2020-11-22T06:06:42.742Z",
              "url" : "https://artifacts.elastic.co/downloads/kibana/kibana-6.3.2-linux-x86_64.tar.gz_1",
              "utc_time" : "2020-11-22T06:06:42.742Z"
            },
            "highlight" : {
              "message" : [
                "159.64.35.129 - - [2018-07-22T06:06:42.742Z] \"GET /<em>kibana</em>/<em>kibana</em>-6.3.2-linux-x86_64.tar.gz_1 HTTP/1.1",
                """" 200 8489 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 <em>Firefox</em>/6.0a1""""
              ]
            }
          }
        ]
      }
    }
    ```
1. 用 "{{" 和 "}}" 把高亮的结果框起来
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "size": 1,
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                "message": "Firefox"
              }
            },
            {
              "match": {
                "message": "Kibana"
              }
            }
          ]
        }
      },
      "highlight": {
        "fields": {
          "message": {
            "pre_tags": "{{",
            "post_tags": "}}"
          }
        }
      }
    }
    ```

    * 返回值
    ```json
    {
      "took" : 8,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 6336,
          "relation" : "eq"
        },
        "max_score" : 3.9088674,
        "hits" : [
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "F7_2FnYBblECTPDiz0DL",
            "_score" : 3.9088674,
            "_source" : {
              "agent" : "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1",
              "bytes" : 8489,
              "clientip" : "159.64.35.129",
              "extension" : "gz",
              "geo" : {
                "srcdest" : "ET:BG",
                "src" : "ET",
                "dest" : "BG",
                "coordinates" : {
                  "lat" : 28.03925,
                  "lon" : -97.54244444
                }
              },
              "host" : "artifacts.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "159.64.35.129",
              "machine" : {
                "ram" : 16106127360,
                "os" : "win xp"
              },
              "memory" : null,
              "message" : """159.64.35.129 - - [2018-07-22T06:06:42.742Z] "GET /kibana/kibana-6.3.2-linux-x86_64.tar.gz_1 HTTP/1.1" 200 8489 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/frederick-w-leslie",
              "request" : "/kibana/kibana-6.3.2-linux-x86_64.tar.gz",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2020-11-22T06:06:42.742Z",
              "url" : "https://artifacts.elastic.co/downloads/kibana/kibana-6.3.2-linux-x86_64.tar.gz_1",
              "utc_time" : "2020-11-22T06:06:42.742Z"
            },
            "highlight" : {
              "message" : [
                "159.64.35.129 - - [2018-07-22T06:06:42.742Z] \"GET /{{kibana}}/{{kibana}}-6.3.2-linux-x86_64.tar.gz_1 HTTP/1.1",
                """" 200 8489 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 {{Firefox}}/6.0a1""""
              ]
            }
          }
        ]
      }
    }
    ```

## 第3题，题解说明

* 这题主要考察高亮以及高亮相关的参数
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-highlighting.html)
  2. 页面路径：Search APIs =》 Request Body Search =》 Highlighting

## 第4题，phrase query 及排序

1. Search for documents with the `message` field containing the phrase "HTTP/1.1 200 51"
   1. 搜索`message`字段中包含phrase "HTTP/1.1 200 51"的
1. Search for documents with the `message` field containing the phrase "HTTP/1.1 200 51", and sort the results by the `machine.os` field in descending order
   1. 搜索`message`字段中包含phrase "HTTP/1.1 200 51"的，并且按`machine.os`的降序排列
1. As above, but also sort the results by the `timestamp` field in ascending order
   1. 接上一条搜索，同时对`timestamp`进行升序排列
1. Run the next queries on the `kibana_sample_data_ecommerce` index
   1. 在另一个索引`kibana_sample_data_ecommerce`中运行下面的检索
1. Search for documents with the `day_of_week` field containing the string "Monday"
   1. 搜索`day_of_week`字段中，包含“Monday”的文档
2. As above, but sort the results by the `products.base_price` field in descending order, picking the lowest value of the array
   1. 接上条检索，但是对`products.base_price`字段中，以数组里最小的一个的值为标准做降序排列

## 第4题，题解

1. phrase 搜索
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "match_phrase": {
          "message": "HTTP/1.1 200 51"
        }
      }
    }
    ```

    * 返回值
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
          "value" : 3,
          "relation" : "eq"
        },
        "max_score" : 4.335473,
        "hits" : [
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "q7_2FnYBblECTPDi9YIW",
            "_score" : 4.335473,
            "_source" : {
              "agent" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
              "bytes" : 51,
              "clientip" : "158.238.118.139",
              "extension" : "",
              "geo" : {
                "srcdest" : "ES:DZ",
                "src" : "ES",
                "dest" : "DZ",
                "coordinates" : {
                  "lat" : 47.3582025,
                  "lon" : -118.6733264
                }
              },
              "host" : "www.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "158.238.118.139",
              "machine" : {
                "ram" : 10737418240,
                "os" : "win 7"
              },
              "memory" : null,
              "message" : """158.238.118.139 - - [2018-09-13T12:36:05.476Z] "GET /security-analytics HTTP/1.1" 200 51 "-" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/luca-parmitano",
              "request" : "/security-analytics",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2021-01-14T12:36:05.476Z",
              "url" : "https://www.elastic.co/solutions/security-analytics",
              "utc_time" : "2021-01-14T12:36:05.476Z"
            }
          },
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "Xr_2FnYBblECTPDi832_",
            "_score" : 4.2687926,
            "_source" : {
              "agent" : "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24",
              "bytes" : 51,
              "clientip" : "226.241.242.182",
              "extension" : "",
              "geo" : {
                "srcdest" : "IQ:CN",
                "src" : "IQ",
                "dest" : "CN",
                "coordinates" : {
                  "lat" : 37.76312194,
                  "lon" : -99.96542389
                }
              },
              "host" : "www.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "226.241.242.182",
              "machine" : {
                "ram" : 9663676416,
                "os" : "win 7"
              },
              "memory" : null,
              "message" : """226.241.242.182 - - [2018-09-08T15:22:31.285Z] "GET /logging HTTP/1.1" 200 51 "-" "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/douglas-wheelock",
              "request" : "/logging",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2021-01-09T15:22:31.285Z",
              "url" : "https://www.elastic.co/solutions/logging",
              "utc_time" : "2021-01-09T15:22:31.285Z"
            }
          },
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "xL_2FnYBblECTPDi833B",
            "_score" : 4.2687926,
            "_source" : {
              "agent" : "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24",
              "bytes" : 51,
              "clientip" : "243.236.31.15",
              "extension" : "",
              "geo" : {
                "srcdest" : "IN:TH",
                "src" : "IN",
                "dest" : "TH",
                "coordinates" : {
                  "lat" : 37.74532639,
                  "lon" : -111.5701653
                }
              },
              "host" : "www.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "243.236.31.15",
              "machine" : {
                "ram" : 7516192768,
                "os" : "win xp"
              },
              "memory" : null,
              "message" : """243.236.31.15 - - [2018-09-08T11:29:16.093Z] "GET /logging HTTP/1.1" 200 51 "-" "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/michael-foale",
              "request" : "/logging",
              "response" : 200,
              "tags" : [
                "success",
                "security"
              ],
              "timestamp" : "2021-01-09T11:29:16.093Z",
              "url" : "https://www.elastic.co/solutions/logging",
              "utc_time" : "2021-01-09T11:29:16.093Z"
            }
          }
        ]
      }
    }
    ```

1. 加`machine.os`降序
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "match_phrase": {
          "message": "HTTP/1.1 200 51"
        }
      },
      "sort": {
        "machine.os": "desc"
      }
    }
    ```

    * 返回报错
    ```json
    {
      "error": {
        "root_cause": [
          {
            "type": "illegal_argument_exception",
            "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [machine.os] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
          }
        ],
        "type": "search_phase_execution_exception",
        "reason": "all shards failed",
        "phase": "query",
        "grouped": true,
        "failed_shards": [
          {
            "shard": 0,
            "index": "kibana_sample_data_logs",
            "node": "cHK2nQePQo-HoCqwRf97Eg",
            "reason": {
              "type": "illegal_argument_exception",
              "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [machine.os] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
            }
          }
        ],
        "caused_by": {
          "type": "illegal_argument_exception",
          "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [machine.os] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead.",
          "caused_by": {
            "type": "illegal_argument_exception",
            "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [machine.os] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
          }
        }
      },
      "status": 400
    }
    ```

    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "match_phrase": {
          "message": "HTTP/1.1 200 51"
        }
      },
      "sort": {
        "machine.os.keyword": "desc"
      }
    }
    ```

    * 返回值
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
          "value" : 3,
          "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "xL_2FnYBblECTPDi833B",
            "_score" : null,
            "_source" : {
              "agent" : "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24",
              "bytes" : 51,
              "clientip" : "243.236.31.15",
              "extension" : "",
              "geo" : {
                "srcdest" : "IN:TH",
                "src" : "IN",
                "dest" : "TH",
                "coordinates" : {
                  "lat" : 37.74532639,
                  "lon" : -111.5701653
                }
              },
              "host" : "www.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "243.236.31.15",
              "machine" : {
                "ram" : 7516192768,
                "os" : "win xp"
              },
              "memory" : null,
              "message" : """243.236.31.15 - - [2018-09-08T11:29:16.093Z] "GET /logging HTTP/1.1" 200 51 "-" "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/michael-foale",
              "request" : "/logging",
              "response" : 200,
              "tags" : [
                "success",
                "security"
              ],
              "timestamp" : "2021-01-09T11:29:16.093Z",
              "url" : "https://www.elastic.co/solutions/logging",
              "utc_time" : "2021-01-09T11:29:16.093Z"
            },
            "sort" : [
              "win xp"
            ]
          },
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "Xr_2FnYBblECTPDi832_",
            "_score" : null,
            "_source" : {
              "agent" : "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24",
              "bytes" : 51,
              "clientip" : "226.241.242.182",
              "extension" : "",
              "geo" : {
                "srcdest" : "IQ:CN",
                "src" : "IQ",
                "dest" : "CN",
                "coordinates" : {
                  "lat" : 37.76312194,
                  "lon" : -99.96542389
                }
              },
              "host" : "www.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "226.241.242.182",
              "machine" : {
                "ram" : 9663676416,
                "os" : "win 7"
              },
              "memory" : null,
              "message" : """226.241.242.182 - - [2018-09-08T15:22:31.285Z] "GET /logging HTTP/1.1" 200 51 "-" "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/douglas-wheelock",
              "request" : "/logging",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2021-01-09T15:22:31.285Z",
              "url" : "https://www.elastic.co/solutions/logging",
              "utc_time" : "2021-01-09T15:22:31.285Z"
            },
            "sort" : [
              "win 7"
            ]
          },
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "q7_2FnYBblECTPDi9YIW",
            "_score" : null,
            "_source" : {
              "agent" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
              "bytes" : 51,
              "clientip" : "158.238.118.139",
              "extension" : "",
              "geo" : {
                "srcdest" : "ES:DZ",
                "src" : "ES",
                "dest" : "DZ",
                "coordinates" : {
                  "lat" : 47.3582025,
                  "lon" : -118.6733264
                }
              },
              "host" : "www.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "158.238.118.139",
              "machine" : {
                "ram" : 10737418240,
                "os" : "win 7"
              },
              "memory" : null,
              "message" : """158.238.118.139 - - [2018-09-13T12:36:05.476Z] "GET /security-analytics HTTP/1.1" 200 51 "-" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/luca-parmitano",
              "request" : "/security-analytics",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2021-01-14T12:36:05.476Z",
              "url" : "https://www.elastic.co/solutions/security-analytics",
              "utc_time" : "2021-01-14T12:36:05.476Z"
            },
            "sort" : [
              "win 7"
            ]
          }
        ]
      }
    }
    ```

1. 加`timestamp`升序
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "match_phrase": {
          "message": "HTTP/1.1 200 51"
        }
      },
      "sort": [
        {
          "machine.os.keyword": "desc"
        },
        {
          "timestamp": "asc"
        }
      ]
    }
    ```

    * 返回值
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
        "max_score" : null,
        "hits" : [
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "xL_2FnYBblECTPDi833B",
            "_score" : null,
            "_source" : {
              "agent" : "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24",
              "bytes" : 51,
              "clientip" : "243.236.31.15",
              "extension" : "",
              "geo" : {
                "srcdest" : "IN:TH",
                "src" : "IN",
                "dest" : "TH",
                "coordinates" : {
                  "lat" : 37.74532639,
                  "lon" : -111.5701653
                }
              },
              "host" : "www.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "243.236.31.15",
              "machine" : {
                "ram" : 7516192768,
                "os" : "win xp"
              },
              "memory" : null,
              "message" : """243.236.31.15 - - [2018-09-08T11:29:16.093Z] "GET /logging HTTP/1.1" 200 51 "-" "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/michael-foale",
              "request" : "/logging",
              "response" : 200,
              "tags" : [
                "success",
                "security"
              ],
              "timestamp" : "2021-01-09T11:29:16.093Z",
              "url" : "https://www.elastic.co/solutions/logging",
              "utc_time" : "2021-01-09T11:29:16.093Z"
            },
            "sort" : [
              "win xp",
              1610191756093
            ]
          },
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "Xr_2FnYBblECTPDi832_",
            "_score" : null,
            "_source" : {
              "agent" : "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24",
              "bytes" : 51,
              "clientip" : "226.241.242.182",
              "extension" : "",
              "geo" : {
                "srcdest" : "IQ:CN",
                "src" : "IQ",
                "dest" : "CN",
                "coordinates" : {
                  "lat" : 37.76312194,
                  "lon" : -99.96542389
                }
              },
              "host" : "www.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "226.241.242.182",
              "machine" : {
                "ram" : 9663676416,
                "os" : "win 7"
              },
              "memory" : null,
              "message" : """226.241.242.182 - - [2018-09-08T15:22:31.285Z] "GET /logging HTTP/1.1" 200 51 "-" "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/douglas-wheelock",
              "request" : "/logging",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2021-01-09T15:22:31.285Z",
              "url" : "https://www.elastic.co/solutions/logging",
              "utc_time" : "2021-01-09T15:22:31.285Z"
            },
            "sort" : [
              "win 7",
              1610205751285
            ]
          },
          {
            "_index" : "kibana_sample_data_logs",
            "_type" : "_doc",
            "_id" : "q7_2FnYBblECTPDi9YIW",
            "_score" : null,
            "_source" : {
              "agent" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
              "bytes" : 51,
              "clientip" : "158.238.118.139",
              "extension" : "",
              "geo" : {
                "srcdest" : "ES:DZ",
                "src" : "ES",
                "dest" : "DZ",
                "coordinates" : {
                  "lat" : 47.3582025,
                  "lon" : -118.6733264
                }
              },
              "host" : "www.elastic.co",
              "index" : "kibana_sample_data_logs",
              "ip" : "158.238.118.139",
              "machine" : {
                "ram" : 10737418240,
                "os" : "win 7"
              },
              "memory" : null,
              "message" : """158.238.118.139 - - [2018-09-13T12:36:05.476Z] "GET /security-analytics HTTP/1.1" 200 51 "-" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)"""",
              "phpmemory" : null,
              "referer" : "http://www.elastic-elastic-elastic.com/success/luca-parmitano",
              "request" : "/security-analytics",
              "response" : 200,
              "tags" : [
                "success",
                "info"
              ],
              "timestamp" : "2021-01-14T12:36:05.476Z",
              "url" : "https://www.elastic.co/solutions/security-analytics",
              "utc_time" : "2021-01-14T12:36:05.476Z"
            },
            "sort" : [
              "win 7",
              1610627765476
            ]
          }
        ]
      }
    }
    ```

1. 搜索`kibana_sample_data_ecommerce`
```bash
POST kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "day_of_week": "Monday"
    }
  }
}
```

* 返回值
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
      "value" : 579,
      "relation" : "eq"
    },
    "max_score" : 2.0880327,
    "hits" : [
      {
        "_index" : "kibana_sample_data_ecommerce",
        "_type" : "_doc",
        "_id" : "_7_2FnYBblECTPDi0UGN",
        "_score" : 2.0880327,
        "_source" : {
          "category" : [
            "Men's Clothing"
          ],
          "currency" : "EUR",
          "customer_first_name" : "Eddie",
          "customer_full_name" : "Eddie Underwood",
          "customer_gender" : "MALE",
          "customer_id" : 38,
          "customer_last_name" : "Underwood",
          "customer_phone" : "",
          "day_of_week" : "Monday",
          "day_of_week_i" : 0,
          "email" : "eddie@underwood-family.zzz",
          "manufacturer" : [
            "Elitelligence",
            "Oceanavigations"
          ],
          "order_date" : "2020-12-14T09:28:48+00:00",
          "order_id" : 584677,
          "products" : [
            {
              "base_price" : 11.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Elitelligence",
              "tax_amount" : 0,
              "product_id" : 6283,
              "category" : "Men's Clothing",
              "sku" : "ZO0549605496",
              "taxless_price" : 11.99,
              "unit_discount_amount" : 0,
              "min_price" : 6.35,
              "_id" : "sold_product_584677_6283",
              "discount_amount" : 0,
              "created_on" : "2016-12-26T09:28:48+00:00",
              "product_name" : "Basic T-shirt - dark blue/white",
              "price" : 11.99,
              "taxful_price" : 11.99,
              "base_unit_price" : 11.99
            },
            {
              "base_price" : 24.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Oceanavigations",
              "tax_amount" : 0,
              "product_id" : 19400,
              "category" : "Men's Clothing",
              "sku" : "ZO0299602996",
              "taxless_price" : 24.99,
              "unit_discount_amount" : 0,
              "min_price" : 11.75,
              "_id" : "sold_product_584677_19400",
              "discount_amount" : 0,
              "created_on" : "2016-12-26T09:28:48+00:00",
              "product_name" : "Sweatshirt - grey multicolor",
              "price" : 24.99,
              "taxful_price" : 24.99,
              "base_unit_price" : 24.99
            }
          ],
          "sku" : [
            "ZO0549605496",
            "ZO0299602996"
          ],
          "taxful_total_price" : 36.98,
          "taxless_total_price" : 36.98,
          "total_quantity" : 2,
          "total_unique_products" : 2,
          "type" : "order",
          "user" : "eddie",
          "geoip" : {
            "country_iso_code" : "EG",
            "location" : {
              "lon" : 31.3,
              "lat" : 30.1
            },
            "region_name" : "Cairo Governorate",
            "continent_name" : "Africa",
            "city_name" : "Cairo"
          }
        }
      },
      {
        "_index" : "kibana_sample_data_ecommerce",
        "_type" : "_doc",
        "_id" : "A7_2FnYBblECTPDi0UKO",
        "_score" : 2.0880327,
        "_source" : {
          "category" : [
            "Men's Clothing",
            "Men's Accessories"
          ],
          "currency" : "EUR",
          "customer_first_name" : "Eddie",
          "customer_full_name" : "Eddie Weber",
          "customer_gender" : "MALE",
          "customer_id" : 38,
          "customer_last_name" : "Weber",
          "customer_phone" : "",
          "day_of_week" : "Monday",
          "day_of_week_i" : 0,
          "email" : "eddie@weber-family.zzz",
          "manufacturer" : [
            "Elitelligence"
          ],
          "order_date" : "2020-12-07T03:48:58+00:00",
          "order_id" : 574916,
          "products" : [
            {
              "base_price" : 59.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Elitelligence",
              "tax_amount" : 0,
              "product_id" : 11262,
              "category" : "Men's Clothing",
              "sku" : "ZO0542505425",
              "taxless_price" : 59.99,
              "unit_discount_amount" : 0,
              "min_price" : 28.2,
              "_id" : "sold_product_574916_11262",
              "discount_amount" : 0,
              "created_on" : "2016-12-19T03:48:58+00:00",
              "product_name" : "Winter jacket - black",
              "price" : 59.99,
              "taxful_price" : 59.99,
              "base_unit_price" : 59.99
            },
            {
              "base_price" : 20.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Elitelligence",
              "tax_amount" : 0,
              "product_id" : 15713,
              "category" : "Men's Accessories",
              "sku" : "ZO0601306013",
              "taxless_price" : 20.99,
              "unit_discount_amount" : 0,
              "min_price" : 10.7,
              "_id" : "sold_product_574916_15713",
              "discount_amount" : 0,
              "created_on" : "2016-12-19T03:48:58+00:00",
              "product_name" : "Watch - green",
              "price" : 20.99,
              "taxful_price" : 20.99,
              "base_unit_price" : 20.99
            }
          ],
          "sku" : [
            "ZO0542505425",
            "ZO0601306013"
          ],
          "taxful_total_price" : 80.98,
          "taxless_total_price" : 80.98,
          "total_quantity" : 2,
          "total_unique_products" : 2,
          "type" : "order",
          "user" : "eddie",
          "geoip" : {
            "country_iso_code" : "EG",
            "location" : {
              "lon" : 31.3,
              "lat" : 30.1
            },
            "region_name" : "Cairo Governorate",
            "continent_name" : "Africa",
            "city_name" : "Cairo"
          }
        }
      },
      {
        "_index" : "kibana_sample_data_ecommerce",
        "_type" : "_doc",
        "_id" : "Bb_2FnYBblECTPDi0UKO",
        "_score" : 2.0880327,
        "_source" : {
          "category" : [
            "Men's Clothing"
          ],
          "currency" : "EUR",
          "customer_first_name" : "Oliver",
          "customer_full_name" : "Oliver Rios",
          "customer_gender" : "MALE",
          "customer_id" : 7,
          "customer_last_name" : "Rios",
          "customer_phone" : "",
          "day_of_week" : "Monday",
          "day_of_week_i" : 0,
          "email" : "oliver@rios-family.zzz",
          "manufacturer" : [
            "Low Tide Media",
            "Elitelligence"
          ],
          "order_date" : "2020-11-30T09:27:22+00:00",
          "order_id" : 565855,
          "products" : [
            {
              "base_price" : 20.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Low Tide Media",
              "tax_amount" : 0,
              "product_id" : 19919,
              "category" : "Men's Clothing",
              "sku" : "ZO0417504175",
              "taxless_price" : 20.99,
              "unit_discount_amount" : 0,
              "min_price" : 9.87,
              "_id" : "sold_product_565855_19919",
              "discount_amount" : 0,
              "created_on" : "2016-12-12T09:27:22+00:00",
              "product_name" : "Shirt - dark blue white",
              "price" : 20.99,
              "taxful_price" : 20.99,
              "base_unit_price" : 20.99
            },
            {
              "base_price" : 24.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Elitelligence",
              "tax_amount" : 0,
              "product_id" : 24502,
              "category" : "Men's Clothing",
              "sku" : "ZO0535205352",
              "taxless_price" : 24.99,
              "unit_discount_amount" : 0,
              "min_price" : 12.49,
              "_id" : "sold_product_565855_24502",
              "discount_amount" : 0,
              "created_on" : "2016-12-12T09:27:22+00:00",
              "product_name" : "Slim fit jeans - raw blue",
              "price" : 24.99,
              "taxful_price" : 24.99,
              "base_unit_price" : 24.99
            }
          ],
          "sku" : [
            "ZO0417504175",
            "ZO0535205352"
          ],
          "taxful_total_price" : 45.98,
          "taxless_total_price" : 45.98,
          "total_quantity" : 2,
          "total_unique_products" : 2,
          "type" : "order",
          "user" : "oliver",
          "geoip" : {
            "country_iso_code" : "GB",
            "location" : {
              "lon" : -0.1,
              "lat" : 51.5
            },
            "continent_name" : "Europe"
          }
        }
      }
    ]
  }
}
```

1. 对`products.base_price`字段数组中最小的一个进行降序排列
```bash
POST kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "day_of_week": "Monday"
    }
  },
  "sort": {
    "products.base_price": {
      "order": "desc",
      "mode": "min"
    }
  }
}
```

* 返回值
```json
{
  "took" : 160,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 579,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "kibana_sample_data_ecommerce",
        "_type" : "_doc",
        "_id" : "H7_2FnYBblECTPDi2033",
        "_score" : null,
        "_source" : {
          "category" : [
            "Men's Clothing"
          ],
          "currency" : "EUR",
          "customer_first_name" : "Wagdi",
          "customer_full_name" : "Wagdi Shaw",
          "customer_gender" : "MALE",
          "customer_id" : 15,
          "customer_last_name" : "Shaw",
          "customer_phone" : "",
          "day_of_week" : "Monday",
          "day_of_week_i" : 0,
          "email" : "wagdi@shaw-family.zzz",
          "manufacturer" : [
            "Oceanavigations"
          ],
          "order_date" : "2020-11-23T06:16:12+00:00",
          "order_id" : 739290,
          "products" : [
            {
              "base_price" : 1079.98,
              "discount_percentage" : 0,
              "quantity" : 2,
              "manufacturer" : "Oceanavigations",
              "tax_amount" : 0,
              "product_id" : 2669,
              "category" : "Men's Clothing",
              "sku" : "ZO0288302883",
              "taxless_price" : 1079.98,
              "unit_discount_amount" : 0,
              "min_price" : 259.2,
              "_id" : "sold_product_739290_2669",
              "discount_amount" : 0,
              "created_on" : "2016-12-05T06:16:12+00:00",
              "product_name" : "Leather jacket - black",
              "price" : 1079.98,
              "taxful_price" : 1079.98,
              "base_unit_price" : 539.99
            },
            {
              "base_price" : 419.98,
              "discount_percentage" : 0,
              "quantity" : 2,
              "manufacturer" : "Oceanavigations",
              "tax_amount" : 0,
              "product_id" : 16673,
              "category" : "Men's Clothing",
              "sku" : "ZO0274002740",
              "taxless_price" : 419.98,
              "unit_discount_amount" : 0,
              "min_price" : 113.39,
              "_id" : "sold_product_739290_16673",
              "discount_amount" : 0,
              "created_on" : "2016-12-05T06:16:12+00:00",
              "product_name" : "Suit - dark grey",
              "price" : 419.98,
              "taxful_price" : 419.98,
              "base_unit_price" : 209.99
            },
            {
              "base_price" : 399.98,
              "discount_percentage" : 0,
              "quantity" : 2,
              "manufacturer" : "Oceanavigations",
              "tax_amount" : 0,
              "product_id" : 14843,
              "category" : "Men's Clothing",
              "sku" : "ZO0291502915",
              "taxless_price" : 399.98,
              "unit_discount_amount" : 0,
              "min_price" : 90,
              "_id" : "sold_product_739290_14843",
              "discount_amount" : 0,
              "created_on" : "2016-12-05T06:16:12+00:00",
              "product_name" : "Classic coat - camel multicolor",
              "price" : 399.98,
              "taxful_price" : 399.98,
              "base_unit_price" : 199.99
            },
            {
              "base_price" : 349.98,
              "discount_percentage" : 0,
              "quantity" : 2,
              "manufacturer" : "Oceanavigations",
              "tax_amount" : 0,
              "product_id" : 24351,
              "category" : "Men's Clothing",
              "sku" : "ZO0288702887",
              "taxless_price" : 349.98,
              "unit_discount_amount" : 0,
              "min_price" : 82.25,
              "_id" : "sold_product_739290_24351",
              "discount_amount" : 0,
              "created_on" : "2016-12-05T06:16:12+00:00",
              "product_name" : "Down coat - black",
              "price" : 349.98,
              "taxful_price" : 349.98,
              "base_unit_price" : 174.99
            }
          ],
          "sku" : [
            "ZO0288302883",
            "ZO0288702887",
            "ZO0274002740",
            "ZO0291502915"
          ],
          "taxful_total_price" : 2249.92,
          "taxless_total_price" : 2249.92,
          "total_quantity" : 8,
          "total_unique_products" : 4,
          "type" : "order",
          "user" : "wagdi",
          "geoip" : {
            "country_iso_code" : "SA",
            "location" : {
              "lon" : 45,
              "lat" : 25
            },
            "continent_name" : "Asia"
          }
        },
        "sort" : [
          350.0
        ]
      },
      {
        "_index" : "kibana_sample_data_ecommerce",
        "_type" : "_doc",
        "_id" : "_r_2FnYBblECTPDi4FI2",
        "_score" : null,
        "_source" : {
          "category" : [
            "Women's Shoes",
            "Women's Clothing"
          ],
          "currency" : "EUR",
          "customer_first_name" : "Elyssa",
          "customer_full_name" : "Elyssa Davidson",
          "customer_gender" : "FEMALE",
          "customer_id" : 27,
          "customer_last_name" : "Davidson",
          "customer_phone" : "",
          "day_of_week" : "Monday",
          "day_of_week_i" : 0,
          "email" : "elyssa@davidson-family.zzz",
          "manufacturer" : [
            "Gnomehouse"
          ],
          "order_date" : "2020-12-07T02:19:41+00:00",
          "order_id" : 574828,
          "products" : [
            {
              "base_price" : 74.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Gnomehouse",
              "tax_amount" : 0,
              "product_id" : 14417,
              "category" : "Women's Shoes",
              "sku" : "ZO0324903249",
              "taxless_price" : 74.99,
              "unit_discount_amount" : 0,
              "min_price" : 40.49,
              "_id" : "sold_product_574828_14417",
              "discount_amount" : 0,
              "created_on" : "2016-12-19T02:19:41+00:00",
              "product_name" : "Lace-up boots - camel",
              "price" : 74.99,
              "taxful_price" : 74.99,
              "base_unit_price" : 74.99
            },
            {
              "base_price" : 99.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Gnomehouse",
              "tax_amount" : 0,
              "product_id" : 19888,
              "category" : "Women's Clothing",
              "sku" : "ZO0354103541",
              "taxless_price" : 99.99,
              "unit_discount_amount" : 0,
              "min_price" : 50.99,
              "_id" : "sold_product_574828_19888",
              "discount_amount" : 0,
              "created_on" : "2016-12-19T02:19:41+00:00",
              "product_name" : "Classic coat - camel",
              "price" : 99.99,
              "taxful_price" : 99.99,
              "base_unit_price" : 99.99
            }
          ],
          "sku" : [
            "ZO0324903249",
            "ZO0354103541"
          ],
          "taxful_total_price" : 174.98,
          "taxless_total_price" : 174.98,
          "total_quantity" : 2,
          "total_unique_products" : 2,
          "type" : "order",
          "user" : "elyssa",
          "geoip" : {
            "country_iso_code" : "US",
            "location" : {
              "lon" : -74,
              "lat" : 40.8
            },
            "region_name" : "New York",
            "continent_name" : "North America",
            "city_name" : "New York"
          }
        },
        "sort" : [
          75.0
        ]
      },
      {
        "_index" : "kibana_sample_data_ecommerce",
        "_type" : "_doc",
        "_id" : "xr_2FnYBblECTPDi0UOY",
        "_score" : null,
        "_source" : {
          "category" : [
            "Women's Shoes"
          ],
          "currency" : "EUR",
          "customer_first_name" : "Abigail",
          "customer_full_name" : "Abigail Phelps",
          "customer_gender" : "FEMALE",
          "customer_id" : 46,
          "customer_last_name" : "Phelps",
          "customer_phone" : "",
          "day_of_week" : "Monday",
          "day_of_week_i" : 0,
          "email" : "abigail@phelps-family.zzz",
          "manufacturer" : [
            "Gnomehouse",
            "Karmanite"
          ],
          "order_date" : "2020-11-30T15:18:43+00:00",
          "order_id" : 566170,
          "products" : [
            {
              "base_price" : 64.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Gnomehouse",
              "tax_amount" : 0,
              "product_id" : 7278,
              "category" : "Women's Shoes",
              "sku" : "ZO0324803248",
              "taxless_price" : 64.99,
              "unit_discount_amount" : 0,
              "min_price" : 31.85,
              "_id" : "sold_product_566170_7278",
              "discount_amount" : 0,
              "created_on" : "2016-12-12T15:18:43+00:00",
              "product_name" : "Boots - navy",
              "price" : 64.99,
              "taxful_price" : 64.99,
              "base_unit_price" : 64.99
            },
            {
              "base_price" : 84.99,
              "discount_percentage" : 0,
              "quantity" : 1,
              "manufacturer" : "Karmanite",
              "tax_amount" : 0,
              "product_id" : 5214,
              "category" : "Women's Shoes",
              "sku" : "ZO0703907039",
              "taxless_price" : 84.99,
              "unit_discount_amount" : 0,
              "min_price" : 43.34,
              "_id" : "sold_product_566170_5214",
              "discount_amount" : 0,
              "created_on" : "2016-12-12T15:18:43+00:00",
              "product_name" : "Ankle boots - wood",
              "price" : 84.99,
              "taxful_price" : 84.99,
              "base_unit_price" : 84.99
            }
          ],
          "sku" : [
            "ZO0324803248",
            "ZO0703907039"
          ],
          "taxful_total_price" : 149.98,
          "taxless_total_price" : 149.98,
          "total_quantity" : 2,
          "total_unique_products" : 2,
          "type" : "order",
          "user" : "abigail",
          "geoip" : {
            "country_iso_code" : "GB",
            "location" : {
              "lon" : -1.9,
              "lat" : 52.5
            },
            "region_name" : "Birmingham",
            "continent_name" : "Europe",
            "city_name" : "Birmingham"
          }
        },
        "sort" : [
          65.0
        ]
      }
    ]
  }
}
```

## 第4题，题解说明

* 这题主要考察的是phrase query以及单/多字段排序
  * phrase query，主要指的是`match_phrase`是一种强制保持搜索内容顺序的搜索命令，不像`match`命中query（分词之后的词元）中任何一个都能返回，`match_phrase`需要按顺序命中query中的词元才会返回。
    * 如下面的对比测试，用`match` 和 `match_phrase` 能命中的数据差了好多
      ```bash
      POST kibana_sample_data_logs/_count
      {
        "query": {
          "match": {
            "message": "HTTP/1.1 200 51"
          }
        }
      }
      ```
      * 返回值
      ```json
      {
        "count" : 14074,
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "skipped" : 0,
          "failed" : 0
        }
      }
      ```

      ```bash
      POST kibana_sample_data_logs/_count
      {
        "query": {
          "match_phrase": {
            "message": "HTTP/1.1 200 51"
          }
        }
      }
      ```

      * 返回值
      ```json
      {
        "count" : 3,
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "skipped" : 0,
          "failed" : 0
        }
      }
      ```
  * 排序和mysql一样分正序倒序，可以支持单/多个字段的混合排序，多字段是按 `sort` 数组的顺序依次生效的
    * 由于ES所有字段都支持数组，所以在排序的时候可能会存在需要对数组字段进行排序的需求，所以sort中支持包括最大（`max`）最小（`min`）等数组排序模式
  1. [参考链接-match-phrase](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-match-query-phrase.html)，[参考链接-sort](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-sort.html)
  2. 页面路径-match-phrase：Query DSL =》 Full text queries =》 Match phrase
  3. 页面路径-sort：Search APIs =》 Request Body Search =》 Sort
