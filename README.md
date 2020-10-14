# ELK_Learning

### installing 
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