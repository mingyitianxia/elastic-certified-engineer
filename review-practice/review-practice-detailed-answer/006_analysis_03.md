## Analysis
分析（分词）

## GOAL: set the analyzer on data index against requirements
目标：按要求创建索引
建议docker-compose文件：`1e1k_base_cluster.yml`

## 第1题，为数据字段指定分词器

1. Create the index `hamlet_1` with one primary shard and no replicas
   1. 创建一个1分片0副本的索引`hamlet_1`
2. Define a mapping for the default type "_doc" of `hamlet_1`, so that
   1. 设置`hamlet_1`的索引配置，使它满足以下条件
   2. the type has three fields, named `speaker`, `line_number`, and `text_entry`,
      1. 它有3个字段`speaker`, `line_number` 和 `text_entry`
   3. `text_entry` is associated with the language "english" analyzer
      1. `text_entry`的分析器是"english"
3. Add some documents to `hamlet_1` by running the following command
   1. 通过下面的命令给`hamlet_1`插入数据
      ```bash
      PUT hamlet_1/_bulk
      {"index":{"_index":"hamlet_1","_id":0}}
      {"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
      {"index":{"_index":"hamlet_1","_id":1}}
      {"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
      {"index":{"_index":"hamlet_1","_id":2}}
      {"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
      {"index":{"_index":"hamlet_1","_id":3}}
      {"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
      ```

## 第1题，题解

1. 创建索引
    ```bash
    PUT hamlet_1
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      },
      "mappings": {
        "properties": {
          "speaker": {
            "type": "text"
          },
          "line_number": {
            "type": "text"
          },
          "text_entry": {
            "type": "text",
            "analyzer": "english"
          }
        }
      }
    }
    ```
1. 插入数据，略。

## 第1题，题解说明

* 这题主要考察的是索引配置中的分词器（`analyzer`）的设置，ES在处理文本数据时，会尝试通过分词器将文本数据拆散成词元，然后再对词元进行倒排索引
  1. [参考链接-analysis](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis.html)，[参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analyzer.html)
  2. 页面路径：Analysis
  3. 页面路径：Mapping =》 Mapping parameters =》 analyzer

## 第2题，自定义分词器

1. Create the index `hamlet_2` with one primary shard and no replicas
   1. 创建1分片0副本的索引`hamlet_2`
1. Add to `hamlet_2` a custom analyzer named `shy_hamlet_analyzer`, consisting of 
   1. 给`hamlet_2`添加一个满足下面要求的自定义分词器`shy_hamlet_analyzer`
   2. a char filter to replace the characters "Hamlet" with "[CENSORED]",
      1. 它有一个词元转换器可以把 "Hamlet" 替换成 "[CENSORED]"
   3. a tokenizer to split tokens on whitespaces and columns,  
      1. 它有一个词元提取器通过空格（`whitespaces`）和逗号（`columns`）把语句拆开
   4. a token filter to ignore any token with less than 5 characters
      1. 一个词元过滤器来剔除掉所有长度小于5个字符的词元
1. Define a mapping for the default type "_doc" of `hamlet_2`, so that
   1. 给`hamlet_2`的默认type设置以下索引设置
   2. the type has one field named `text_entry`,
      1. 它有一个字段叫`text_entry`
   3. `text_entry` is associated with the `shy_hamlet_analyzer` created in the previous step
      1. `text_entry`的分词器是上面设置的`shy_hamlet_analyzer`
1. Reindex the `text_entry` field of `hamlet_1` into `hamlet_2`
   1. 把`text_entry`字段从`hamlet_1` reindex 到 `hamlet_2`
2. Verify that documents have been reindexed to `hamlet_2` as expected - e.g., by searching for "censored" into the `text_entry` field
   1. 校验一下所有数据都被 reindex到`hamlet_2`了，比如通过在`text_entry`里搜“censored”

## 第2题，题解

1. 创建索引
    ```bash
    PUT hamlet_2
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0,
        "analysis": {
          "analyzer": {
            "shy_hamlet_analyzer": {
              "char_filter": [
                "hamlet_char_filter"
              ],
              "tokenizer": "hamlet_tokenizer",
              "filter": [
                "hamlet_filter"
              ]
            }
          },
          "char_filter": {
            "hamlet_char_filter": {
              "type": "mapping",
              "mappings": [
                "Hamlet => [CENSORED]"
              ]
            }
          },
          "tokenizer": {
            "hamlet_tokenizer": {
              "type": "pattern",
              "pattern": "[\\s,]"
            }
          },
          "filter": {
            "hamlet_filter": {
              "type": "length",
              "min": 5
            }
          }
        }
      },
      "mappings": {
        "properties": {
          "text_entry": {
            "type": "text",
            "analyzer": "shy_hamlet_analyzer"
          }
        }
      }
    }
    ```
1. reindex
    ```bash
    POST _reindex
    {
      "source": {
        "index": "hamlet_1",
        "_source": ["text_entry"]
      },
      "dest": {
        "index": "hamlet_2"
      }
    }
    ```
1. 数据校验
    ```bash
    POST hamlet_2/_search
    {
      "query": {
        "match": {
          "text_entry": "[CENSORED]"
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
          "value" : 1,
          "relation" : "eq"
        },
        "max_score" : 0.65592396,
        "hits" : [
          {
            "_index" : "hamlet_2",
            "_type" : "_doc",
            "_id" : "3",
            "_score" : 0.65592396,
            "_source" : {
              "text_entry" : "Though yet of Hamlet our dear brothers death"
            }
          }
        ]
      }
    }
    ```

## 第2题，题解说明

* 这题主要考察分词器里面的各种个性化配置
  * `analyzer`里主要包含`tokenizer`，`char_filter`，`filter`和`position_increment_gap`
    * `tokenizer`：词元提取器，通过什么方式把字符串拆开（比如本题中的空格和逗号）
    * `char_filter`：词元转换器，通过什么方式把解析出来的词元进行清洗
    * `filter`：词元过滤器，以什么规则对解析出来对词元进行过滤
    * `position_increment_gap`：词元位置打散参数，为了防止短语之间对词元太过接近而设置的打散步长
  * `reindex`，我们之前的使用大都是直接全量导数据，这里用到了它对字段对限制，指定字段的复制
  1. [参考链接-custom-analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-custom-analyzer.html)，[参考链接-pattern-tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-pattern-tokenizer.html)，[参考链接-mapping-char-filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-mapping-charfilter.html)，[参考链接-analyzer-length-filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-length-tokenfilter.html)
     1. 页面路径-custom-analyzer：Analysis =》 Analyzers =》 Custom Analyzer
     2. 页面路径-pattern-tokenizer：Analysis =》 Tokenizers
    =》 Pattern Tokenizer
     1. 页面路径-mapping-charfilter：Analysis =》 Character Filters =》 Mapping Char Filter
     2. 页面路径-analyzer-length-filter：Analysis =》 Token Filters =》 Length Token Filter
  2. [参考链接-reindex](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-reindex.html)
     1. 页面路径：Document APIs =》 =》 Reindex API