POST /_analyze
{
  "text":"The policeman had their confesion",
  "analyzer":""
}


# 1 Create the indices `hamlet-1` and `hamlet-2`, each with two 
#    primary shards and no replicas
# Add some documents to `hamlet-1` by running the following command

DELETE hamlet-1
DELETE hamlet-2

PUT hamlet-1
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 2
  }
}

PUT hamlet-2
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 2
  }
}

PUT hamlet-1/_doc/_bulk
{"index":{"_index":"hamlet-1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet-1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet-1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet-1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}

# Add some documents to `hamlet-2` by running the following command

PUT hamlet-2/_doc/_bulk
{"index":{"_index":"hamlet-2","_id":4}}
{"line_number":"2.1.1","speaker":"LORD POLONIUS","text_entry":"Give him this money and these notes, Reynaldo."}
{"index":{"_index":"hamlet-2","_id":5}}
{"line_number":"2.1.2","speaker":"REYNALDO","text_entry":"I will, my lord."}
{"index":{"_index":"hamlet-2","_id":6}}
{"line_number":"2.1.3","speaker":"LORD POLONIUS","text_entry":"You shall do marvellous wisely, good Reynaldo,"}
{"index":{"_index":"hamlet-2","_id":7}}
{"line_number":"2.1.4","speaker":"LORD POLONIUS","text_entry":"Before you visit him, to make inquire"}

#2 Create the alias `hamlet` that maps both `hamlet-1` and `hamlet-2` 
# Verify that the documents grouped by `hamlet` are 8

POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "hamlet-1", "alias" : "hamlet" } },
          { "add" : { "index" : "hamlet-2", "alias" : "hamlet" } }
    ]
}

GET hamlet/_count

#3 # Configure `hamlet-1` to be the write index of the `hamlet` alias
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "hamlet-1",
        "alias": "hamlet",
        "is_write_index": true
      }
    },
    {
      "add": {
        "index": "hamlet-2",
        "alias": "hamlet"
      }
    }
  ]
}

# 4 Add a document to `hamlet`, so that the document (i) has id "8", 
#    (ii) has "_doc" type, (iii) has a field `text_entry` with value 
#    "With turbulent and dangerous lunacy?", (iv) has a field 
#    `line_number` with value "3.1.4", (v) has a field `speaker` 
#    with value "KING CLAUDIUS"
POST hamlet/_doc/8
{
  "text_entry":"With turbulent and dangerous lunacy?",
  "line_number":"3.1.4",
  "speaker":"KING CLAUDIUS"
}


# Create a script named `control_reindex_batch` and save it into the 
#    cluster state. The script checks whether a document has the 
#    field `reindexBatch`, and (i) in the affirmative case, it 
#    increments the field value by a script parameter named 
#    `increment`, (ii) otherwise, the script adds the field to the 
#    document setting its value to "1"


POST _scripts/control_reindex_batch
{
  "script": {
    "lang": "painless",
  "source": """
      if (ctx._source['reindexBatch'] != null) {
        ctx._source['reindexBatch'] += params.increment;
      } else {
        ctx._source['reindexBatch'] = 1;
      }
    """
  }
}

# Create the index `hamlet-new` with 2 primary shards and no 
#    replicas
# Reindex `hamlet` into `hamlet-new`, while satisfying the following 
#    criteria: (i) apply the `control_reindex_batch` script with the 
#    `increment` parameter set to "1", (ii) reindex using two
#    parallel slices


PUT hamlet-new
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 0
  }
}

POST _reindex?slices=2
{
  "source": {
    "index": "hamlet"
  },
  "dest": {
    "index": "hamlet-new"
  },
  "script": {
        "id": "control_reindex_batch",
        "params": {
          "increment": 1
        }
      }
}

GET hamlet-new/_search


# Create a pipeline named `split_act_scene_line`. The pipeline 
#    splits the value of `line_number` using the dots as a 
#    separator, and stores the split values into three 
#    new fields named `number_act`, `number_scene`, and 
#    `number_line`, respectively
To verify that an ingest pipeline works as expected, you can rely on the _simulate pipeline API.
# Test the pipeline on the following document{
  "_source": {
    "line_number": "1.2.3"
  }
}

PUT _ingest/pipeline/split_act_scene_line
{
  "description": "split pipeline",
  "processors": [
    {
      "split": {
        "field": "line_number",
        "separator": "\\."
      }
    }
  ]
}

//感谢：https://t.zsxq.com/7EQJ2fm hello郎 提出的bug
POST _ingest/pipeline/_simulate
{
  "pipeline" : {
   "description": "split pipeline",
  "processors": [
    {
      "split": {
        "field": "line_number",
        "separator": "\\."
      }
    },
    {
      "set": {
        "field": "number_act",
        "value": "{{line_number.0}}"
      }
    },
     {
      "set": {
        "field": "number_scene",
        "value": "{{line_number.1}}"
      }
    },
     {
      "set": {
        "field": "number_line",
        "value": "{{line_number.2}}"
      }
    },
     {
        "script": {
          "source": """
            ctx.a1 = ctx.line_number.0;
            ctx.a2 = ctx.line_number.1;
            ctx.a3 = ctx.line_number.2;
          """
        }
      }
  ]
  },
  "docs" : [
    { "_source": { "line_number": "1.2.3"} }
  ]
}



PUT hamlet-new/_doc/111
{
  "line_number": "1.2.3"
}

GET hamlet-new/_doc/111
#Satisfied with the outcome? Go update your documents, then!
# Update all documents in `hamlet-new` by using the 
#    `split_act_scene_line` pipeline

POST hamlet-new/_update_by_query?pipeline=split_act_scene_line

POST hamlet-new/_search
