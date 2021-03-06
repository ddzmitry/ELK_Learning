# ELK_Learning



## installing 
```
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.9.2
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.9.2
```
### Checking
```
curl -X GET "localhost:9200/_cat/nodes?v&pretty"
```


#### Summary
> Nodes store the data in Elastic
> A cluster collection of nodes
> Data stored in documents, which are JSON
> Documents grouped with indices


## Managing documents 

#### Queries

#### Get Stats about clustrer
```
> Cluster Health 
GET /_cluster/health 
> Get Nodes
GET /_cat/nodes?v
GET /_nodes
> Get Indices
GET /_cat/indices?v
```
### Curl
```
url -XGET "http://localhost:9200/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match_all": {}  }}'
```

### Add Index
```
PUT /pages
```
### Get Shards
```
GET /_cat/shards?v
```

### Adding node in configs
```
# In new instance change node name
node.name: node-2
# Start it 
bin/elasticsearch
# Get nodes (now its 2)
GET /_cat/nodes?v
```
### Adding node by changing config in command line (not best option)
```
bin/elasticsearch -Enode.name=node-3 -Epath.data=./node-3/data -Epath.logs=./node-3/logs
```

### Master node
```
# config
node.master: true|false
```
### Data node (store and search data)
```
# config
node.data: true|false
```
### Ingest node (enables node to run ingest pipelines)
> Ingest pipelines are a series of steps(processors) that are performed when indexing documents
```
# config
node.ingest: true|false
```
#### Abbreviations 
```
GET /_cat/nodes?v

node.role :dilmrt  - data,ingest,master


```
### Create Delete Index 
```
DELETE /pages
# Specify shards and replicas
PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}
```
### Adding Data to index
```
POST /products/_doc
{
  "name":"Coffee Maker",
  "price" : 64,
  "in_stock": 10
}
# Add by key
PUT /products/_doc/100
{
  "name":"Toaster",
  "price" : 49,
  "in_stock": 4

}
```
### Getting Data from index
```
GET /products/_doc/100
```

### Updating data by id
```
POST /products/_update/100
{
  "doc": {
    "in_stock" :3
  }
}

# Add new field
POST /products/_update/100
{
  "doc": {
    "tags" : ["electronics"]
  }
}

```
### Scripted Updates to Change Data
> This script will decrease in_stock data by 1 (in_stock--)
```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
```
> Set Value (hard coded)
```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}
```
> Set Value using script and parameters ( params.qty is a variable that can be updated to apply to script)
```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.qty",
    "params": {
      "qty" : 4
    }
  }
}
# Will decrease value in_stock by 4 
```

### Upserts (Update docs)
```
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock -= params.qty",
    "params": {
      "qty" : 4
    }
  },
  "upsert": {
    "name" : "Blender",
    "price" : 399 ,
    "in_stock" : 5
  }
}
```

### Replacing Doc 
```
GET /products/_doc/100
# Will "re-put" value
PUT /products/_doc/100
{
  "name" : "Toaster",
  "price" : 50,
  "in_stock" : 10

}
```

### Delete Doc
```
DELETE /products/_doc/100
```
### Number of Shards
```
shard_num = hash(_routing) % num_primary_shards
```
### update by sequence 
```
First GET
GET /products/_doc/100
>> OUT 
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 18,
  "_seq_no" : 22,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Toaster",
    "price" : 50,
    "in_stock" : 10
  }
}



Second post 
POST /products/_update/100?if_primary_term=1&if_seq_no=22
{
  "doc": {
    "in_stock" : 123
  }
}
If you run second query again it will fail because sequence number had changed to 23

```

### Update By query
```
# Run for all documents and decrease value by one 
# Update many
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}

>> OUT (query goes to coordinating node)
{
  "took" : 994,
  "timed_out" : false,
  "total" : 3,
  "updated" : 3, - total records
  "deleted" : 0,
  "batches" : 1, - how many batches it took to retrieve document (using scroll API)
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}


```
### To force update and ignore conflicts 
```
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}
> Check updates
# Get All
GET /products/_search
{
    "query": {
    "match_all": {}
  }
}
```

### Delete by query
```
Delete by Condition 

POST /products/_delete_by_query
{
  "query": {
    "match_all" : {}
  }
}
```

### Batch Processing - Add values via bulk request by id and pass values (have to be inline)
> link https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html
#### Post with bulk
```
# Batch Processing

POST /_bulk
{ "index" : { "_index" : "products", "_id" : 200 } }
{ "name": "Espresso Machine", "price": 199 , "in_stock":5 }
{ "create": { "_index" : "products", "_id" : 201 } }
{ "name": "Milk Frother", "price": 149, "in_stock": 14 }

```
#### Update with bulk
```
POST /_bulk
{ "update": { "_index" : "products", "_id" : 201 } }
{ "doc" : {"name": "THE Milk Frother", "price": 180, "in_stock": 250 } }
```
#### Delete and update with bulk with index provided 
```
# We specify index right in the request , so we dont have to write it in body
POST /products/_bulk
{ "update": { "_id" : 201 } }
{ "doc" : { "in_stock": 10 } }
{ "delete": {  "_id" : 200 } }
```

### Importing with curl
```
url -H "Content-Type: application/x-ndjson" -XPOST http://localhost:9200/products/_bulk  --data-binary "@data.json"

# Check shards to make sure documents there 
GET /_cat/shards?v

```

## Section 4 Mapping and Analysis
####  Basic Analyzer
```
POST /_analyze
{
  "text": "This is a text to analyze",
  "analyzer": "standard"
}

POST /_analyze
{
  "text": "This is a text to analyze ANALUZE",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}
```
#### Analyze By keyword - by exact match works well for emails
```
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS!",
  "analyzer": "keyword"
}

```

#### Coersion will work with type that can be converted to a float (different types will create both text and float mapping)
```
PUT /coercian_test/_doc/1
{
  "price" : 7.4
}
PUT /coercian_test/_doc/2
{
  "price" : "7.4"
}

# will fail adding doc
PUT /coercian_test/_doc/3
{
  "price" : "7.4m"
}

GET /coercian_test/_doc/2
```
#### NB Elastic Search dont use coersian , it forigves if we put two different types inside of field of the index 

#### Array - Elastic will accept array as input value , because it only cares about text
```
# Verify by analyze API
POST /_analyze
{
  "text": ["Strings are simply","merged together"],
  "analyzer": "standard"
}
# Output
{
  "tokens" : [
    {
      "token" : "strings",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "are",
      "start_offset" : 8,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "simply",
      "start_offset" : 12,
      "end_offset" : 18,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "merged",
      "start_offset" : 19,
      "end_offset" : 25,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "together",
      "start_offset" : 26,
      "end_offset" : 34,
      "type" : "<ALPHANUM>",
      "position" : 4
    }
  ]
}

```
#### NB All types in array can't be mixed (as long as they can be coersed it will work )
> [Boolean,{Object}] - WILL NOT WORK

```
      "start_offset" : 26,
      "end_offset" : 34,
      Separates words like on a single line
```

### Adding Explicit Mapping
```
PUT /reviews 
{
  "mappings": {
    "properties": {
      "rating" : { "type": "float"},
      "content" : {"type": "text"},
      "product_id" : {"type": "integer"},
      "author" : {
        "properties": {
          "first_name" : {"type": "text"},
          "last_name" : {"type": "text"},
          "email" : {"type": "keyword"}
        }
      }
      
    }
  }
}

# Add Data
PUT /reviews/_doc/1
{
  "rating" : 5.0 ,
  "content" : "Great course! Bo really taught me a lot about ELK",
  "product_id" : 123,
  "author" : {
    "first_name" : "Joe",
    "last_name" : "Doe",
    "email" : "johndoe@example.com"
  }
}

```

### Retrieving Mappings 
```
GET /reviews/_mapping
# By field
GET /reviews/_mapping/field/author.email
GET /reviews/_mapping/field/rating
```

### Adding Mappings to existing index 
```
GET /reviews/_mapping

# Add datefield mapping
PUT /reviews/_mapping
{
  "properties": {
    "created_at" : {
      "type" : "date"
    }
  }
  
}
```