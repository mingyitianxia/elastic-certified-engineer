### 自定义分词插件，让king's和kings有相同的评分

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
