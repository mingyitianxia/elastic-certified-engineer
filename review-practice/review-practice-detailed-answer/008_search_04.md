#某索引index_a有多个字段，

#要求实现如下的查询：
#1）针对字段title，满足'ssas'或者'sasa',至少一个满足
#2）针对字段tags（数组字段），如果b字段包含'pingpang',则提升评分。

PUT index_a/_bulk
{"index":{"_id":1}}
{"title":"ssas is very nb", "tags":["pingpang", "basketball"]}
{"index":{"_id":2}}
{"title":"which is sasa","tags":["football"]}
{"index":{"_id":3}}
{"title":"which is ssas","tags":["basktball","football"]}
{"index":{"_id":4}}
{"title":"just for testing", "tags":["pingpang"]}
{"index":{"_id":5}}
{"title":"just for testing", "tags":["basketball"]}
{"index":{"_id":6}}
{"title":"just for testing", "tags":["football"]}

# 方法1
POST index_a/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "bool": {
            "should": [
              {
                "match": {
                  "title": "ssas"
                }
              },
              {
                "match": {
                  "title": "sasa"
                }
              }
            ],
            "minimum_should_match": 1
          }
        }
      ],
      "should": [
        {
          "match": {
            "tags": {
              "query": "pingpang",
              "boost": 2
            }
          }
        }
      ]
    }
  }
}
# 方法2(感谢球友：Shawn.cx 阿炳提出bug，已修复 20200313）
POST index_a/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "terms": {
            "title": [
              "ssas",
              "sasa"
            ]
          }
        }
      ],
      "should": [
        {
          "match": {
            "tags": {
              "query": "pingpang",
              "boost": 2
            }
          }
        }
      ]
    }
  }
}

* 这题没啥好讲的，通过 `should` + `minimum_should_match` 或者 `terms` 保证命中，然后用 `should` 做加分就完了
  1. [参考链接-boolean-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-bool-query.html)，[参考链接-boost](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/mapping-boost.html)
  2. 页面路径-boolean-query：Query DSL =》 Compound queries =》 Boolean
  3. 页面路径-boost：Mapping =》 Mapping parameters =》 boost