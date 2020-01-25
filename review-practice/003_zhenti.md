### 0、真题来源

https://www.elastic.co/cn/training/certification/faq

### 1、部署实战题

![1](imgs/deploy_001.png)
==
![1](imgs/deploy_001_02.png)
==
```
cluster.name: cluster1 

# node1
node.name: node1
node.master: true 
node.data: false 
node.ingest: false 
node.ml: false 
cluster.remote.connect: false 

discovery.seed_hosts: ["node1", "node2", "node3"]
cluster.initial_master_nodes: ["node1"]

#node2
node.name: node2
node.master: false 
node.data: true 
node.ingest: false 
node.ml: false 
cluster.remote.connect: false 

#node3
node.name: node3
node.master: false 
node.data: false 
node.ingest: true 
node.ml: false 
cluster.remote.connect: false 
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

