### 0、真题来源

https://www.elastic.co/cn/training/certification/faq

### 1、部署实战题

![1](imgs/deploy_001.png)
==
![1](imgs/deploy_001_02.png)
==
```
PUT  earthquakes
{
  "mappings":{
    "properties":{
      "magnitude":{
        "type":"long"
      }
    }
  }
}

POST /earthquakes/_search
{
  "size": 0,
  "aggs": {
    "mag_over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "avg_mags": {
          "avg": {
            "field": "magnitude.keyword"
          }
        }
      }
    },
    "max_mag_of_month": {
      "max_bucket": {
        "buckets_path": "mag_over_time>avg_mags"
      }
    }
  }
}
```

### 2、Mapping定义实战题
![1](imgs/deploy_002.png)
==
```
PUT task2
{
  "mappings": {
    "properties": {
      "address": {
        "properties": {
          "city": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          },
          "country": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          },
          "state": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          }
        }
      },
      "comment": {
        "type": "text",
        "analyzer": "standard", 
        "fields": {
          "english": {
            "type": "text",
           "analyzer": "english"
          },
          "dutch": {
            "type": "text",
           "analyzer": "dutch"
          }
        }
      },
      "username": {
        "type": "keyword"
      }
    }
  }
}

POST task2/_doc/123
{
  "username":"",
  "address":{
    "city":"Mountain View",
    "state":"California",
    "country":"United States of America"
  },
  "comment":"To be prepare for the exam, you shoule be able to complete all the exam object"
}

GET task2/_mapping
```

### 4、查询实战题
![1](imgs/deploy_003.png)
==
```
POST movie_data/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "match": {
            "tags": "based on comic book"
          }
        },
        {
          "match": {
            "tags": "marvel cinematic universe"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "budget": {
        "order": "desc"
      }
    },
    {
      "release_date": {
        "order": "desc"
      }
    }
  ]
}
```

