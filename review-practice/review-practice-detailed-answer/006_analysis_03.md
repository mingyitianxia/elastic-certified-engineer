DELETE hamlet_*
DELETE _template/*

# Create the index `hamlet_1` with one primary shard and no replicas
# Define a mapping for the default type "_doc" of `hamlet_1`, so 
#    that (i) the type has three fields, named `speaker`, 
#    `line_number`, and `text_entry`, (ii) `text_entry` is 
#    associated with the language "english" analyzer
# Add some documents to `hamlet_1` by running the following command


PUT hamlet_1
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "speaker":{"type": "text"},
      "line_number":{"type": "text"},
      "text_entry":{"type": "text","analyzer": "english"}
    }
  }
}



PUT hamlet_1/_bulk
{"index":{"_index":"hamlet_1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet_1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet_1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet_1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}


# Create the index `hamlet_2` with one primary shard and no replicas
# Add to `hamlet_2` a custom analyzer named `shy_hamlet_analyzer`, 
#    consisting of 
#    (i)   a char filter to replace the characters "Hamlet" with "
#         [CENSORED]",  
#    (ii)  a tokenizer to split tokens on whitespaces and columns,  
#    (iii) a token filter to ignore any token with less than 5  characters 
# Define a mapping for the default type "_doc" of `hamlet_2`, so 
#    that (i) the type has one field named `text_entry`, (ii) 
#    `text_entry` is associated with the `shy_hamlet_analyzer` 
#    created in the previous step

PUT hamlet_2
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 1,
    "analysis": {
      "analyzer": {
        "shy_hamlet_analyzer": {
          "type": "custom",
          "tokenizer": "whitespace",
          "char_filter": [
            "my_char_filter"
          ],
          "filter": [
            "my_condition"
          ]
        }},
        "char_filter": {
          "my_char_filter": {
            "type": "mapping",
            "mappings": [
              "Hamlet => [CENSORED]"
            ]
          }
        },
        "filter": {
          "my_condition": {
            "type": "condition",  
            "filter" : [ "lowercase" ],
            "script": {
              "source": "token.getTerm().length() < 5"
            }
          }
        }
      }
  },
  "mappings": {
    "properties": {
      "text_entry":{
        "type": "text",
        "analyzer": "shy_hamlet_analyzer"
      }
    }
  }
}

# Reindex the `text_entry` field of `hamlet_1` into `hamlet_2`
# Verify that documents have been reindexed to `hamlet_2` as 
#    expected - e.g., by searching for "censored" into the 
#    `text_entry` field

POST _reindex
{
"source": {
  "index": "hamlet_1"
},
"dest": {
  "index": "hamlet_2"
}
}
POST hamlet_2/_search

POST hamlet_2/_analyze
{
  "analyzer": "shy_hamlet_analyzer",
  "text": "Though yet of Hamlet our dear brothers death"
}

POST hamlet_2/_search
{
  "query": {
    "match": {
      "text_entry": "[CENSORED]"
    }
  }
}

