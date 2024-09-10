

# API introduction

SeaSearch uses Http Basic Auth for permission verification, and the API request needs to carry the corresponding token in the header.

Generate basic auth through this tool: [http://web.chacuo.net/safebasicauth](http://web.chacuo.net/safebasicauth)

## User management

### Administrator user

SeaSearch manages API permissions through accounts. When the program is started for the first time, an administrator account needs to be configured through environment variables.

The following is an example of an administrator account:

```
set ZINC_FIRST_ADMIN_USER=admin
set ZINC_FIRST_ADMIN_PASSWORD=xxx
```

### Normal user

Users can be created/updated via the API:

```
[POST] /api/user

{ 
    "_id": "prabhat",
    "name": "Prabhat Sharma",
    "role": "admin", // or user
    "password": "xxx"
}
```

get all users：

```
[GET] /api/user
```

delete user：

```
[DELETE] /api/user/${userId}
```

## Index related

### create index

Create a SeaSearch index, and you can set both mappings and settings at the same time.

We can also set settings or mapping directly through other requests. If the index does not exist, it will be created automatically.

SeaSearch documentation：[https://zincsearch-docs.zinc.dev/api/index/create/#update-a-exists-index](https://zincsearch-docs.zinc.dev/api/index/create/#update-a-exists-index)

ES documentation：[https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)

### Configure mappings

Mappings define the rules for fields in a document, such as type, format, etc.

Mapping can be configured via a separate API:

SeaSearch api: [https://zincsearch-docs.zinc.dev/api-es-compatible/index/update-mapping/](https://zincsearch-docs.zinc.dev/api-es-compatible/index/update-mapping/)

ES related instructions：[https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)


### Configure settings

Settings set the analyzer sharding and other related settings of the index.

SeaSearch api: [https://zincsearch-docs.zinc.dev/api-es-compatible/index/update-settings/](https://zincsearch-docs.zinc.dev/api-es-compatible/index/update-settings/)

ES related instructions：

  * analyzer related concepts：[https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-concepts.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-concepts.html)

  * How to specify an analyzer：[https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html)

### Analyzer support

Analyzer can configure the default when creating an index, or set it for a specific field. (Refer to the settings ES documentation in the previous section to understand the relevant concepts.)

The analyzers supported by SeaSearch can be found on this page: [https://zincsearch-docs.zinc.dev/api/index/analyze/](https://zincsearch-docs.zinc.dev/api/index/analyze/). The concepts such as tokenize and token filter are consistent with ES, and most of the commonly used analyzers and tokenizers in ES are supported.

Supported general analyzers

  * standard, the default analyzer. If not specified, this analyzer is used to split words and lowercase them.

  * simple, split according to non-letters (symbols are filtered), lowercase

  * keyword, no word segmentation, directly treat input as output

  * stop, lowercase, stop word filter (the, a, is, etc.)

  * web, implemented by Bluge, matching email addresses, urls, etc. Handling lowercase, using stop word filters

  * regexp/pattern, regular expression, default is \W+ (non-character segmentation), supports lowercase and stop words

  * whitespace, split by space, do not convert to lowercase


### Luanguages analyzers

| Country        | Shortened form |
| -------------- | -------------- |
| arabic         | ar             |
| Asia Countries | cjk            |
| sorani         | ckb            |
| danish         | da             |
| german         | de             |
| english        | en             |
| spanish        | es             |
| persian        | fa             |
| finnish        | fi             |
| french         | fr             |
| hindi          | hi             |
| hungarian      | hu             |
| italian        | it             |
| dutch          | nl             |
| norwegian      | no             |
| portuguese     | pt             |
| romanian       | ro             |
| russian        | ru             |
| swedish        | sv             |
| turkish        | tr             |


Chinese analyzer:

  * gse_standard, use the shortest path algorithm to segment words

  * gse_search, the search engine's word segmentation mode provides as many keywords as possible

The Chinese analyzer uses the [gse](https://github.com/go-ego/gse) library to implement word segmentation. It is a Golang implementation of the Python stammer library. It is not enabled by default and needs to be enabled through environment variables.

```
ZINC_PLUGIN_GSE_ENABLE=true
# true: enable Chinese word segmentation support, default is false

ZINC_PLUGIN_GSE_DICT_EMBED=BIG 
# BIG: use the gse built-in vocabulary and stop words; otherwise, use the SeaSearch built-in simple vocabulary, the default is small

ZINC_PLUGIN_GSE_ENABLE_STOP=true
# true: use stop words, default true

ZINC_PLUGIN_GSE_ENABLE_HMM=true
# Use HMM mode for search word segmentation, default is true

ZINC_PLUGIN_GSE_DICT_PATH=./plugins/gse/dict
# To use a user-defined word library and stop words, you need to put the content in the configured path, and name the word library user.txt and the stop words stop.txt
```


## Full text search

### document CRUD

create document:

SeaSearch API: [https://zincsearch-docs.zinc.dev/api-es-compatible/document/create/](https://zincsearch-docs.zinc.dev/api-es-compatible/document/create/)

ES API：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)

update document:

SeaSearch API: [https://zincsearch-docs.zinc.dev/api-es-compatible/document/update/](https://zincsearch-docs.zinc.dev/api-es-compatible/document/update/)

ES API: [https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)

delete document：

SeaSearch API: [https://zincsearch-docs.zinc.dev/api-es-compatible/document/delete/](https://zincsearch-docs.zinc.dev/api-es-compatible/document/delete/)

ES API: [https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete.html)

Get document by id:

```
[GET] /api/${indexName}/_doc/${docId}
```

### Batch Operation

Batch operations should be used to update indexes whenever possible.

SeaSearch API： [https://zincsearch-docs.zinc.dev/api-es-compatible/document/bulk/#request](https://zincsearch-docs.zinc.dev/api-es-compatible/document/bulk/#request)

ES API：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)


### search

API examples:

[https://zincsearch-docs.zinc.dev/api-es-compatible/search/search/](https://zincsearch-docs.zinc.dev/api-es-compatible/search/search/)

Full-text search uses DSL. For usage, please refer to:

[https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

delete-by-query：Delete based on query

```
[POST] /es/${indexName}/_delete_by_query

{
  "query": {
    "match": {
      "name": "jack"
    }
  }
}
```

ES API: [https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html)

multi-search，supports executing different queries on different indexes:

SeaSearch API: [https://zincsearch-docs.zinc.dev/api-es-compatible/search/msearch/](https://zincsearch-docs.zinc.dev/api-es-compatible/search/msearch/)

ES API: [https://www.elastic.co/guide/en/elasticsearch/reference/current/search-multi-search.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-multi-search.html)

We have extended multi-search to support using the same statistics when searching different indexes to make the score calculation more accurate. You can enable it by setting query: unify_score=true in the request.

```
[POST] /es/_msearch?unify_score=true

{"index": "t1"}
{"query": {"bool": {"should": [{"match": {"filename": {"query": "test string", "minimum_should_match": "-25%"}}}, {"match": {"filename.ngram": {"query": "test string", "minimum_should_match": "80%"}}}], "minimum_should_match": 1}}, "from": 0, "size": 10, "_source": ["path", "repo_id", "filename", "is_dir"], "sort": ["_score"]}
{"index": "t2"}
{"query": {"bool": {"should": [{"match": {"filename": {"query": "test string", "minimum_should_match": "-25%"}}}, {"match": {"filename.ngram": {"query": "test string", "minimum_should_match": "80%"}}}], "minimum_should_match": 1}}, "from": 0, "size": 10, "_source": ["path", "repo_id", "filename", "is_dir"], "sort": ["_score"]}
```


## Vector search

We have developed a vector search function for the SeaSearch extension. The following is an introduction to the relevant API.

### Create vector search

To use the vector search function, you need to create a vector index in advance, which can be done through mapping.

We create an index and set the vector field of the document data to be written to be called "vec", the index type is flat, and the vector dimension is 768

```
[PUT] /es/${indexName}/_mapping

{
"properties":{
        "vec":{
            "type":"vector", 
            "dims":768,
            "m":64,
            "nbits":8,
            "vec_index_type":"flat"
        }
    }
}
```

Parameter Description:

```
${indexName} zincIndex, index name

type,  fixed to vector, indicating vector index
dims,  vector dimensions
m,     ivf_pq index required parameters, need to be divisible by dims
nbits, ivf_pq index required parameter, default is 8
vec_index_type, index type, supports two types: flat and ivf_pq
```

### Write a document containing a vector


There is no difference between writing a document containing a vector and writing a normal document at the API level. You can choose the appropriate method.

The following takes the bluk API as an example

```
[POST] /es/_bulk

body:

{ "index" : { "_index" : "index1" } } 
{"name": "jack1","vec":[10.2,10.41,9.5,22.2]}
{ "index" : { "_index" : "index1" } } 
{"name": "jack2","vec":[10.2,11.41,9.5,22.2]}
{ "index" : { "_index" : "index1" } } 
{"name": "jack3","vec":[10.2,12.41,9.5,22.2]}
```

Note that the _bulk API strictly requires the format of each line, and the data cannot exceed one line. For details, please refer to [ES bulk](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)

Modification and deletion can also be done using bulk. After deleting a document, its corresponding vector data will also be deleted


### Retrieval vector

By passing in a vector, we can search for N similar vectors in the system and return the corresponding document information:

```
[POST] /api/${indexName}/_search/vector

body:
{
    {
    "query_field":"vec",
    "k":7,
    "return_fields":["name"],
    "vector":[10.2,10.40,9.5,22.2.......],
    "_source":false
    }
}
```

The API response format is the same as the full-text search format.

The following is a description of the parameters:

```
${indexName} zincIndex, index name

query_field,    the field in the index to retrieve, the field must be of vector type
k,              the number of K most similar vectors to return
return_fields,  the name of the field to be returned individually
vector,         the vector used for query
nprobe,         only works for ivf_pq index type, the number of clusters to query, the higher the number, the more accurate
_source,        it is used to control whether to return the _source field, supports bool or an array, describing which fields need to be returned

```

### Rebuild index

Rebuild the index immediately, suitable for situations where you don't need to wait for background automatic detection.

```
[POST] /api/:target/:field/_rebuild
```

### query recall


For vectors of type ivf_pq, recall checks can be performed on their data.

```
[POST] /api/:target/_recall
{
    "field":"vec_001", # Fields to test
    "k":10, 
    "nprobe":5, # nprobe number
    "query_count":1000 # Number of times the test was performed
}
```

# Vector search usage examples

Next, we will demonstrate how to index a batch of papers. Each paper may contain multiple vectors that need to be indexed. We hope to obtain the most similar N vectors through vector retrieval, and thus obtain their corresponding paper-ids.

## Creating SeaSearch indexes and vector indexes

The first step is to set the mapping of the vector index. When setting the mapping, the index and vector index are automatically created.

Since paper-id is just a normal string, we don't need to analyze it, so we set its type to keyword:

```
[PUT] /es/paper/_mapping

{
"properties":{
        "title-vec":{
            "type":"vector", 
            "dims":768,
            "vec_index_type":"flat",
            "m":1
        },
        "paper-id":{
            "type":"keyword"
        }
    }
}
```

Through the above request, we created an index named paper and established a flat vector index for the title-vec field of the index.

## Index data

We write these paper data to SeaSearch in batches through the _bulk API.

```
[POST] /es/_bulk

{ "index" : {"_index" : "paper" } } 
{"paper-id": "001","title-vec":[10.2,10.40,9.5,22.2....]}
{ "index" : {"_index" : "paper" } } 
{"paper-id": "002","title-vec":[10.2,11.40,9.5,22.2....]}
{ "index" : {"_index" : "paper" } } 
{"paper-id": "003","title-vec":[10.2,12.40,9.5,22.2....]}
....


```

## Retrieving data

Now we can retrieve it using the vector:

```
[POST] /api/paper/_search/vector

{
    "query_field":"title-vec",
    "k":10,
    "return_fields":["paper-id"],
    "vector":[10.2,10.40,9.5,22.2....]
}
```

The document corresponding to the most similar vector can be retrieved, and the paper-id can be obtained. Since a paper may contain multiple vectors, if multiple vectors of a paper are very similar to the query vector, then this paper-id may appear multiple times in the results.

## Maintaining vector data

### Update the document directly

After a document is successfully imported, SeaSearch will return its doc id. We can directly update a document based on the doc id:

```
[POST] /es/_bulk

{ "update" : {"_id":"23gZX9eT6QM","_index" : "paper" } } 
{"paper-id": "005","vec":[10.2,1.43,9.5,22.2...]}
```

### Query first and then update

If the returned doc id is not saved, you can first use SeaSearch's full-text search function to query the documents corresponding to paper-id:

```
[POST] /es/paper/_search

{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {"paper-id":"003"}
                }
            ]
        }
    }
}
```

Through DSL, we can directly retrieve the document corresponding to the paper-id and its doc id.

### Fully updated paper

A paper contains multiple vectors. If a vector needs to be updated, we can directly update the document corresponding to the vector. However, in actual applications, it is not easy to distinguish which contents of a paper are newly added and which are updated.

We can adopt the method of full update:

  * First, query all documents of a paper through DSL

  * Delete all documents

  * Import the latest paper data

Steps 2 and 3 can be performed in one batch operation.

The following example will demonstrate deleting the document of paper 001 and re-importing it; at the same time, directly updating paper 005 and paper 006 because they only have one vector:

```
[POST] /es/_bulk


{ "index" : {"_index" : "paper" } } 
{"paper-id": "001","title-vec":[10.2,10.40,9.5,22.2....]}
{ "index" : {"_index" : "paper" } } 
{"paper-id": "002","title-vec":[10.2,11.40,9.5,22.2....]}
{ "index" : {"_index" : "paper" } } 
{"paper-id": "003","title-vec":[10.2,12.40,9.5,22.2....]}
....


```

