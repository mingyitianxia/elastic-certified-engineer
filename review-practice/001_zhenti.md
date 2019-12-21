### 1、自定义分词插件，让king's和kings有相同的评分

```
PUT test002
{
  "mapping":{
   "Properties":{
     "title":{
       "type":"text",
       "analyzer":"simple"
     }
   }   
  }
}

POST test002/_bulk
{"index":{"_id":1}}
{"title":"king's"}
{"index":{"_id":2}}
{"title":"kings"}

POST test002/_msearch
{}
{"query":{"match_phrase":{"title":"kings"}}}
{}
{"query":{"match_phrase":{"title":"king's"}}}
```

### 2、有一个文档，内容类似dog & cat， 要求索引这条文档，并且使用match_phrase query，查询dog & cat或者dog and cat都能match。

https://elasticsearch.cn/article/6133

```
DELETE whitespace_example
PUT /whitespace_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_whitespace": {
          "tokenizer": "whitespace",
           "char_filter": [
            "my_char_filter"
          ]
        }
      },
        "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            "& => and"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title":{
        "type":"text",
        "analyzer": "rebuilt_whitespace"
      }
    }
  }
}

POST whitespace_example/_bulk
{"index":{"_id":1}}
{"title":"dog & cat"}
{"index":{"_id":2}}
{"title":"dog and cat"}

POST whitespace_example/_msearch
{}
{"query":{"match_phrase":{"title":"dog & cat"}}}
{}
{"query":{"match_phrase":{"title":"dog and cat"}}}
```
