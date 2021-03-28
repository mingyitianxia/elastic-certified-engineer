## EXAM OBJECTIVE: QUERIES
考点：queries

## GOAL: Use scroll API, search templates, script queries
考试目标：使用 scroll API, search templates, script queries

## REQUIRED SETUP: 
初始化步骤：
建议docker-compose文件：`1e1k_base_cluster.yml`

1. a running Elasticsearch cluster with at least one node and a Kibana instance,
   1. 运行一个至少含有1ES节点1Kibana节点的集群
2. add the "Sample web logs" and "Sample flight data" to Kibana
   1. 在Kibana里添加"Sample web logs" 和 "Sample flight data"两个示例数据集

## 第1题，`scroll api`

1. Search for all documents in all indices
   1. 在所有索引中搜索所有文档
2. As above, but use the scroll API to return the first 100 results while keeping the search context alive for 2 minutes
   1. 接上个query，但是使用 `scroll API` 返回前100条结果，同时使搜索上下文保持2分钟不过期
3. Use the scroll id included in the response to the previous query and retrieve the next batch of results
   1. 使用之前结果中的 “scroll id” 来获取下一批的数据

## 第1题，题解

1. 搜索所有文档
    ```bash
    POST /_search
    {
      "query": {
        "match_all": {}
      }
    }
    ```
    或者
    ```bash
    GET /_search
    ```
   * 数据校验的时候可以在URL里拼上`?track_total_hits=true` =》 `POST/GET /_search?track_total_hits=true`来看具体集群里有多少条数据

1. 接上调用 “scroll api”，保持上下文2分钟
    ```bash
    POST /_search?scroll=2m
    {
      "size": 100,
      "query": {
        "match_all": {}
      }
    }
    ```
    或者
    ```bash
    GET /_search?scroll=2m&size=100
    ```

    * 返回值
    ```json
    {
      "_scroll_id" : "DnF1ZXJ5VGhlbkZldGNoEQAAAAAACDlJFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5SBZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOUoWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlLFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5TBZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOU0WY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlOFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5TxZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOVEWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlQFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5UhZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOVMWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlUFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5VRZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOVcWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlWFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5WBZjSEsyblFlUFFvLUhvQ3F3UmY5N0Vn",
      "took" : 5,
      "timed_out" : false,
      "_shards" : {
        "total" : 17,
        "successful" : 17,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 31910,
          "relation" : "eq"
        },
        "max_score" : 1.0,
        "hits" : [
        ]
      }
    }
    ```

1. 通过scroll_id拉取下一批数据
    ```bash
    POST /_search/scroll
    {
      "scroll": "2m",
      "scroll_id": "DnF1ZXJ5VGhlbkZldGNoEQAAAAAACDlJFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5SBZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOUoWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlLFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5TBZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOU0WY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlOFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5TxZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOVEWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlQFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5UhZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOVMWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlUFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5VRZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOVcWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDlWFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5WBZjSEsyblFlUFFvLUhvQ3F3UmY5N0Vn"
    }
    ```
    或者
    ```bash
    GET /_search/scroll?scroll=2m&scroll_id=DnF1ZXJ5VGhlbkZldGNoEQAAAAAACDmmFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5qBZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOacWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDmpFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5qhZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOasWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDmsFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5tRZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOa0WY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDmuFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5sBZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOa8WY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDm0FmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5shZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIObEWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDm2FmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5sxZjSEsyblFlUFFvLUhvQ3F3UmY5N0Vn
    ```

    * 返回值
    ```json
    {
      "_scroll_id" : "DnF1ZXJ5VGhlbkZldGNoEQAAAAAACDmmFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5qBZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOacWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDmpFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5qhZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOasWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDmsFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5tRZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOa0WY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDmuFmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5sBZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIOa8WY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDm0FmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5shZjSEsyblFlUFFvLUhvQ3F3UmY5N0VnAAAAAAAIObEWY0hLMm5RZVBRby1Ib0Nxd1JmOTdFZwAAAAAACDm2FmNISzJuUWVQUW8tSG9DcXdSZjk3RWcAAAAAAAg5sxZjSEsyblFlUFFvLUhvQ3F3UmY5N0Vn",
      "took" : 14,
      "timed_out" : false,
      "_shards" : {
        "total" : 17,
        "successful" : 17,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 31910,
          "relation" : "eq"
        },
        "max_score" : 1.0,
        "hits" : [
        ]
      }
    }
    ```

## 第1题，题解说明

* 这题主要考察的是 scroll api
  * 不同于普通的 `from + size`，或者 `search_after`，scroll api 类似传统数据库中的游标操作，可以在一个query里将大量的数据取出ES
  * scroll 里返回的数据不是每次都会进行计算，而是类似一个ES收到请求时当前数据状态的快照（`snapshot`），所以在通过 scroll 进行数据拉取时，ES里的数据有所修改的话，可能会产生数据的不一致
  * ES 为了不被这种快照占太多的空间，会强制要求穿一个过期参数（`scroll`），同时也会在集群内部对 scroll 的搜索上下文个数进行限制，所以建议在平时使用时不要将这个参数设的太大。
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-scroll.html)
  2. 页面路径：Search APIs =》 Request Body Search =》 Scroll

## 第2题，搜索模板（`search_template`）

1. Run the next queries on the `kibana_sample_data_logs` index
   1. 在索引`kibana_sample_data_logs`里运行下面的搜索
2. Filter documents with the `response` field greater or equal to 400
   1. 筛选出 `response` 字段大于等于400的文档
3. Create a search template for the above query, so that the template 
   1. 基于上面的搜索创建一个满足下面要求的搜索模板
   2. is named "with_response_and_tag",
      1. 这模板名字是 "with_response_and_tag"
   3. has a parameter "with_min_response" to represent the lower bound of the `response` field, 
      1. 有个叫 "with_min_response" 的参数用来指定 `response` 字段的下限
   4. has a parameter "with_max_response" to represent the upper bound of the `response` field, 
      1. 有个叫 "with_max_response" 的参数用来指定 `response` 字段的上限
   5. has a parameter "with_tag" to represent a possible value of the `tags` field
      1. 有个叫 "with_tag" 的参数用来指定 `tags` 的可用值
4. Test the "with_response_and_tag" search template by setting the parameters as follows:
   1. 用以下参数组合来测试 "with_response_and_tag" 这个搜索模板
   2. "with_min_response": 400
   3. "with_max_response": 500
   4. "with_tag": "security"

## 第2题，题解

1. 搜索
    ```bash
    POST kibana_sample_data_logs/_search
    {
      "query": {
        "range": {
          "response": {
            "gte": 400
          }
        }
      }
    }
    ```

    * 计数
    ```json
    {
      "took" : 90,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 1242,
          "relation" : "eq"
        },
        "max_score" : 1.0,
        "hits" : [
        ]
      }
    }
    ```

1. 创建搜索模板
    ```bash
    POST _scripts/with_response_and_tag
    {
      "script": {
        "lang": "mustache",
        "source": """
        {
          "query": {
            "bool": {
              "filter": [
                {
                  "range": {
                    "response": {
                      "gte": "{{with_min_response}}",
                      "lte": "{{with_max_response}}"
                    }
                  }
                },
                {
                  "terms": {{#toJson}}with_tag{{/toJson}}
                }
              ]
            }
          }
        }
        """
      }
    }
    ```

    * 渲染这个模板
    ```bash
    GET _render/template
    {
      "id": "with_response_and_tag",
      "params": {
        "with_min_response": 400,
        "with_max_response": 500,
        "with_tag": {
          "tags": [
            "security"
          ]
        }
      },
      "profile": true
    }
    ```

    * 返回值
    ```json
    {
      "template_output" : {
        "query" : {
          "bool" : {
            "filter" : [
              {
                "range" : {
                  "response" : {
                    "gte" : "400",
                    "lte" : "500"
                  }
                }
              },
              {
                "terms" : {
                  "tags" : [
                    "security"
                  ]
                }
              }
            ]
          }
        }
      }
    }
    ```

1. 测试这个模板
    ```bash
    POST _search/template
    {
      "id": "with_response_and_tag",
      "params": {
        "with_min_response": 400,
        "with_max_response": 500,
        "with_tag": {
          "tags": [
            "security"
          ]
        }
      },
      "profile": true
    }
    ```

    * 计数
    ```json
    {
      "took" : 35,
      "timed_out" : false,
      "_shards" : {
        "total" : 17,
        "successful" : 17,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 138,
          "relation" : "eq"
        },
        "max_score" : 0.0,
        "hits" : [
        ]
      }
    }
    ```

## 第2题，题解说明

* 这题主要考察的是搜索模板（`search_template`），搜索模板（可能）是为了解决，当存在大量同样字段、条件只是条件参数不同的query时，不需要每次都构建搜索语句，而把这些同样的条件抽成公共方法，通过填入参数的方式来完成不同的搜索
  * 搜索模板构建的时候和普通的搜索语句一样，同时它也支持包括占位符、简单的条件判断等
    * `{{field_value}}`
    * `{{#field_value}}{{/field_value}}`
    * `{{#url}}{{/url}}`
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-template.html)
  2. 页面路径：Search APIs =》 Search Template

## 第2题，拓展

* 这里要注意的是，为了能使用 `terms` 关键字，我们在模板里把原来的query转成了jsonString（表现为被`"""`包裹），但是这样就让`with_tag`成为了必填字段，如果不填可能会报以下错误
    ```json
    {
      "error": {
        "root_cause": [
          {
            "type": "json_parse_exception",
            "reason": "Unexpected character ('}' (code 125)): expected a value\n at [Source: java.io.StringReader@5e052a4d; line: 15, column: 14]"
          }
        ],
        "type": "json_parse_exception",
        "reason": "Unexpected character ('}' (code 125)): expected a value\n at [Source: java.io.StringReader@5e052a4d; line: 15, column: 14]"
      },
      "status": 500
    }
    ```
 * 另外两个字段由于是原生的query语句参数，所以不填ES只会当它们不存在，并不会报错。所以本题还有一种解法
    ```bash
    POST _scripts/with_response_and_tag
    {
      "script": {
        "lang": "mustache",
        "source": {
          "query": {
            "bool": {
              "filter": [
                {
                  "range": {
                      "response": {
                        "lte": "{{with_max_response}}",
                        "gte": "{{with_min_response}}"
                      }
                  }
                },
                {
                  "match": {
                      "tags": "{{with_tag}}"
                  }
                }
              ]
            }
          }
        }
      }
    }
    ```

    * 渲染这个模板
    ```bash
    GET _render/template
    {
      "id": "with_response_and_tag",
      "params": {
        "with_min_response": 400,
        "with_max_response": 500,
        "with_tag": "security"
      },
      "profile": true
    }
    ```

    * 返回值
    ```json
    {
      "template_output" : {
        "query" : {
          "bool" : {
            "filter" : [
              {
                "range" : {
                  "response" : {
                    "lte" : "500",
                    "gte" : "400"
                  }
                }
              },
              {
                "match" : {
                  "tags" : "security"
                }
              }
            ]
          }
        }
      }
    }
    ```

## 第3题，搜索模板里的条件判断

1. Update the "with_response_and_tag" search template, so that
   1. 更新模板 "with_response_and_tag" 使它满足下面要求
   2. if the "with_max_response" parameter is not set, then don't set an upper bound to the `response` value
      1. 如果 "with_max_response" 参数没设，那就不在条件里拼 `response` 的上限
   3. if the "with_tag" parameter is not set, then do not apply that filter at all
      1. 如果 "with_tag" 参数没设，那就不做任何筛选
2. Test the "with_response_and_tag" search template by setting only the "with_min_response" parameter to 500
   1. 只给参数 "with_min_response" 设成500，然后测试一下 "with_response_and_tag"

## 第3题，题解

1. 更新 "with_response_and_tag"
   1. 用 `terms`
      ```bash
      POST _scripts/with_response_and_tag
      {
        "script": {
          "lang": "mustache",
          "source": """
          {
            "query": {
              "bool": {
                "filter": [
                  {
                    "range": {
                      "response": {
                        {{#with_max_response}}
                        "lte": "{{with_max_response}}",
                        {{/with_max_response}}
                        "gte": "{{with_min_response}}"
                      }
                    }
                  }
                  {{#with_tag}}
                  ,
                  {
                    "terms": {{#toJson}}with_tag{{/toJson}}
                  }
                  {{/with_tag}}
                ]
              }
            }
          }
          """
        }
      }
      ```

      1. 全参数渲染
          ```bash
          GET _render/template
          {
            "id": "with_response_and_tag",
            "params": {
              "with_min_response": 400,
              "with_max_response": 500,
              "with_tag": {
                "tags": [
                  "security"
                ]
              }
            },
            "profile": true
          }
          ```

          * 返回值
          ```json
          {
            "template_output" : {
              "query" : {
                "bool" : {
                  "filter" : [
                    {
                      "range" : {
                        "response" : {
                          "lte" : "500",
                          "gte" : "400"
                        }
                      }
                    },
                    {
                      "terms" : {
                        "tags" : [
                          "security"
                        ]
                      }
                    }
                  ]
                }
              }
            }
          }
          ```

      2. 部分参数渲染
          ```bash
          GET _render/template
          {
            "id": "with_response_and_tag",
            "params": {
              "with_min_response": 500
            },
            "profile": true
          }
          ```

          * 返回值
          ```json
          {
            "template_output" : {
              "query" : {
                "bool" : {
                  "filter" : [
                    {
                      "range" : {
                        "response" : {
                          "gte" : "500"
                        }
                      }
                    }
                  ]
                }
              }
            }
          }
          ```

   2. 用 `match`
      ```bash
      POST _scripts/with_response_and_tag
      {
        "script": {
          "lang": "mustache",
          "source": """
          {
            "query": {
              "bool": {
                "filter": [
                  {
                    "range": {
                        "response": {
                          {{#with_max_response}}
                          "lte": "{{with_max_response}}",
                          {{/with_max_response}}
                          "gte": "{{with_min_response}}"
                        }
                    }
                  }
                  {{#with_tag}}
                  ,
                  {
                    "match": {
                        "tags": "{{with_tag}}"
                    }
                  }
                  {{/with_tag}}
                ]
              }
            }
          }
          """
        }
      }
      ```

      1. 全参数渲染
          ```bash
          GET _render/template
          {
            "id": "with_response_and_tag",
            "params": {
              "with_min_response": 400,
              "with_max_response": 500,
              "with_tag": "security"
            },
            "profile": true
          }
          ```

          * 返回值
          ```json
          {
            "template_output" : {
              "query" : {
                "bool" : {
                  "filter" : [
                    {
                      "range" : {
                        "response" : {
                          "lte" : "500",
                          "gte" : "400"
                        }
                      }
                    },
                    {
                      "match" : {
                        "tags" : "security"
                      }
                    }
                  ]
                }
              }
            }
          }
          ```

      2. 部分参数渲染
          ```bash
          GET _render/template
          {
            "id": "with_response_and_tag",
            "params": {
              "with_min_response": 500
            },
            "profile": true
          }
          ```

          * 返回值
          ```json
          {
            "template_output" : {
              "query" : {
                "bool" : {
                  "filter" : [
                    {
                      "range" : {
                        "response" : {
                          "gte" : "500"
                        }
                      }
                    }
                  ]
                }
              }
            }
          }
          ```

1. 用指定参数测试 "with_response_and_tag"
    ```bash
    POST _search/template
    {
      "id": "with_response_and_tag",
      "params": {
        "with_min_response": 500
      },
      "profile": true
    }
    ```

    * 计数
    ```json
    {
      "took" : 35,
      "timed_out" : false,
      "_shards" : {
        "total" : 17,
        "successful" : 17,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 441,
          "relation" : "eq"
        },
        "max_score" : 1.0,
        "hits" : [
        ]
      }
    }
    ```

## 第3题，题解说明

* 这题衔接上一题，主要考察的是在搜索模板里添加条件判断
  * 由于搜索模板使用的是 `mustache` 语言进行存储的，所以不仅仅可以用语法糖 `"""`，还可以直接转成字符串存下来
    ```bash
    POST _scripts/with_response_and_tag
    {
      "script": {
        "lang": "mustache",
        "source": "{\"query\": {\"bool\": {\"filter\": [{\"range\": {\"response\": {\"gte\": \"{{with_min_response}}\"{{#end}},{{/end}}{{#with_max_response}} \"lte\": \"{{with_max_response}}\"{{/with_max_response}} }}},{ {{#with_tag}}\"match\": {  \"tags\": \"{{with_tag}}\"}{{/with_tag}}}] }}}"
      }
    }
    ```
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-template.html)
  2. 页面路径：Search APIs =》 Search Template
