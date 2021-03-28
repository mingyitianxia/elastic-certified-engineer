## 文本分析 text analysis

同义词/mapping 过滤，其实效果差不多，最大的区别在于如果是1 对 1的映射的话，这俩写起来的复杂程度差不多，但是如果是很多词元做同义词替换的话还是别用mapping了，映射写起来太多了

## 第1题，文本分词

1. 索引一些文档，使得搜索时"king's man"，"kings man"和"kings' man"得分相同
2. 索引一些文档，使得搜索时"oa"，"oA"，"OA"，"Oa"，"0a"，"0A"得分相同

## 第1题，题解

1. "king[']?s[']? man"的得分相同
```bash
PUT mapping_index
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "analysis": {
      "analyzer": {
        "mapping_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "mapping_filter"
          ]
        }
      },
      "char_filter": {
        "mapping_filter": {
          "type": "mapping",
          "mappings": [
            "' => "
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "mapping_field": {
        "type": "text",
        "analyzer": "mapping_analyzer"
      }
    }
  }
}
```

1. 各种"oa"得分相同
```bash
POST synonym_index
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "analysis": {
      "analyzer": {
        "tokenizer": "standard",
        "char_filter": [
          "synonym_filter"
        ]
      },
      "char_filter": {
        "synonym_filter": {
          "type": "synonym",
          "lenient": true,
          "synonyms": [
            "oa Oa OA oA 0a 0A"
          ]
        }
      }
    }
  }
}
```

## 第1题，题解说明

* mapping/synonym 算是分词器里比较难的部分了，他们看起来差不太多，但是实际在配置中还是会存在一些区别的
  * "mapping char filter" 是一种 `char_filter`， 而 "synonym filter" 是 `filter`，所以在做 analyzer 的配置的时候这俩要分别写在`char_filter`和`filter`里面
  * "mapping char filter" 需要维护一个mapping列表，里面详细描述了哪个/些词元需要被替换成哪个/些词元，这些词元是一一对应的，所以一行写多个ES是不认的。"synonym filter" 也维护了一个同义词列表，但不同的是，这个列表里可以是多对一的（多个词都互为同义词，或者他们都和某个词同义）