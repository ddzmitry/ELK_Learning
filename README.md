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