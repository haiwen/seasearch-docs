

# API 介绍



SeaSearch 通过 Http Basic Auth 进行权限校验，API 请求需要在 header 中携带对应的 token。



生成 basic auth 可以通过这个工具: [http://web.chacuo.net/safebasicauth](http://web.chacuo.net/safebasicauth)



## 用户管理



### 管理员用户



SeaSearch 通过账户来管理API权限等，程序在第一次启动时，需要通过环境变量配置一个管理员帐号



以下是 管理员帐号示例：

```plaintext
set ZINC_FIRST_ADMIN_USER=admin
set ZINC_FIRST_ADMIN_PASSWORD=Complexpass#123
```


### 普通用户



可以通过API来创建/更新用户：

```plaintext
[POST] /api/user

{ 
    "_id": "prabhat",
    "name": "Prabhat Sharma",
    "role": "admin", // or user
    "password": "Complexpass#123"
}
```


获取所有用户：

```plaintext
[GET] /api/user
```


删除用户：

```plaintext
[DELETE] /api/user/${userId}
```



## 索引相关



### 创建索引



创建一个 SeaSearch 索引，并且在此时可以同时设置 mappings 以及 settings。



我们也可以直接通过其他请求设置 settings 或者 mapping，如果 index不存在，则会自动创建。



SeaSearch 文档：[https://zincsearch-docs.zinc.dev/api/index/create/#update-a-exists-index](https://zincsearch-docs.zinc.dev/api/index/create/#update-a-exists-index)



参考 ES api文档：[https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)



### 配置 mappings



mappings 定义了 document 中，字段的规则，例如类型，格式等。



可以通过单独的 API 来配置 mapping:



SeaSearch api: [https://zincsearch-docs.zinc.dev/api-es-compatible/index/update-mapping/](https://zincsearch-docs.zinc.dev/api-es-compatible/index/update-mapping/)



ES 相关说明：[https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)



### 配置 settings



settings 设置了 index 的 analyzer 分片等相关设置。



SeaSearch api: [https://zincsearch-docs.zinc.dev/api-es-compatible/index/update-settings/](https://zincsearch-docs.zinc.dev/api-es-compatible/index/update-settings/)



ES 相关说明：



  * analyzer 相关概念：[https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-concepts.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-concepts.html)

  * 如何指定 analyzer：[https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html)






### analyzer 支持



analzyer 可以在创建索引索引时配置 default ，也可以针对某个字段进行设置。（参考上一节中 settings ES 的文档了解相关概念。）



SeaSearch 支持的 analyzer可以在这个页面中找到：[https://zincsearch-docs.zinc.dev/api/index/analyze/](https://zincsearch-docs.zinc.dev/api/index/analyze/) 里面的 tokenize, token filter 等概念和 ES 是一致的，且支持 ES 大部分常用的 analyzer 和 tokenizer 等。



支持的常规analyzer



  * standard 默认的 analyzer，如果没有指定，则采用此 analyzer，按词切分，小写处理

  * simple 按照非字母切分（符号被过滤），小写处理

  * keyword 不分词，直接将输入当作输出 

  * stop 小写处理，停用词过滤器 (the、a、is等）

  * web 由 buluge 实现，匹配 邮箱、url 等。处理小写，使用停用词过滤器

  * regexp/pattern 正则表达式，默认\W+(非字符分割)，支持设置 小写、停用词

  * whitespace 按照空格切分，不转小写






多语言 analzyer：

语言| analyzer  
---|---  
阿拉伯语| ar  
丹麦语| da  
德语| de  
英语| english  
西班牙语| es  
波斯语| fa  
亚洲地区国家| cjk  
芬兰语| fi  
法语| fr  
印地语| hi  
匈牙利语| hu  
意大利语| it  
荷兰语| nl  
挪威语| no  
葡萄牙语| pt  
罗马尼亚语| ro  
俄语| ru  
瑞典语| sv  
土耳其语| tr  
索拉尼| ckb

  
  
中文 analzyer：



  * gse_standard 使用最短路径算法来分词

  * gse_search 搜索引擎的分词模式，提供尽可能多的关键词






中文 analyzer 使用的是 [gse](https://github.com/go-ego/gse) 这个库实现分词，是 python 结巴库的 Golang 实现，默认是没有启用的，需要通过环境变量来启用

```plaintext
ZINC_PLUGIN_GSE_ENABLE=true
# true 启用中文分词支持,默认false

ZINC_PLUGIN_GSE_DICT_EMBED=BIG 
# BIG：使用gse内置词库与停用词；否则，使用 SeaSearch 内置的简单词库，默认 small

ZINC_PLUGIN_GSE_ENABLE_STOP=true
# true 使用停用词，默认 true

ZINC_PLUGIN_GSE_ENABLE_HMM=true
# 使用 HMM 模式用于搜素分词，默认为 true

ZINC_PLUGIN_GSE_DICT_PATH=./plugins/gse/dict
# 使用用户自定义词库与停用词，需要将内容放在配置的这个路径下，并且词库命名为 user.txt
停用词命名为 stop.txt
```


## 全文检索



### document CRUD



创建 document：



SeaSearch ：[https://zincsearch-docs.zinc.dev/api-es-compatible/document/create/](https://zincsearch-docs.zinc.dev/api-es-compatible/document/create/)



ES api 说明：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)



更新 document ：



SeaSearch：[https://zincsearch-docs.zinc.dev/api-es-compatible/document/update/](https://zincsearch-docs.zinc.dev/api-es-compatible/document/update/)



ES api 说明：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)



删除 document：



SeaSearch： [https://zincsearch-docs.zinc.dev/api-es-compatible/document/delete/](https://zincsearch-docs.zinc.dev/api-es-compatible/document/delete/)



ES api 说明：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete.html)



根据 id获取 document：

```plaintext
[GET] /api/${indexName}/_doc/${docId}
```


### 批量进行操作



应该尽量使用批量操作更新索引



SeaSearch文档： [https://zincsearch-docs.zinc.dev/api-es-compatible/document/bulk/#request](https://zincsearch-docs.zinc.dev/api-es-compatible/document/bulk/#request)



ES api说明：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)



### 搜索



api示例：



[https://zincsearch-docs.zinc.dev/api-es-compatible/search/search/](https://zincsearch-docs.zinc.dev/api-es-compatible/search/search/)



全文搜索使用 DSL，使用方法可以参考：



[https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)



delete-by-query：根据 query进行删除：

```plaintext
[POST] /es/${indexName}/_delete_by_query

{
  "query": {
    "match": {
      "name": "jack"
    }
  }
}
```


ES api 文档：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html)



multi-search，支持对不同 index 执行不同的 query：



SeaSearch 文档：[https://zincsearch-docs.zinc.dev/api-es-compatible/search/msearch/](https://zincsearch-docs.zinc.dev/api-es-compatible/search/msearch/)



ES api 文档：[https://www.elastic.co/guide/en/elasticsearch/reference/current/search-multi-search.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-multi-search.html)



我们对 multi-search 做了扩展，使它支持在搜索不同的索引时，使用相同的统计信息，以使得得分计算更加精确，在请求中设置 query：unify_score=true 即可开启。

```plaintext
[POST] /es/

{"index": "t1"}
{"query": {"bool": {"should": [{"match": {"filename": {"query": "数据库", "minimum_should_match": "-25%"}}}, {"match": {"filename.ngram": {"query": "数据库", "minimum_should_match": "80%"}}}], "minimum_should_match": 1}}, "from": 0, "size": 10, "_source": ["path", "repo_id", "filename", "is_dir"], "sort": ["_score"]}
{"index": "t2"}
{"query": {"bool": {"should": [{"match": {"filename": {"query": "数据库", "minimum_should_match": "-25%"}}}, {"match": {"filename.ngram": {"query": "数据库", "minimum_should_match": "80%"}}}], "minimum_should_match": 1}}, "from": 0, "size": 10, "_source": ["path", "repo_id", "filename", "is_dir"], "sort": ["_score"]}
```


## 向量检索



我们为 SeaSearch 扩展开发了向量检索的功能，以下是相关API介绍。



### 创建向量索引



使用向量检索功能，需要提前创建向量索引，可以通过 mapping 的方式建立。



我们创建一个索引，设置写入的文档数据的向量字段叫 "vec"，索引类型是 flat, 向量维度是 768

```plaintext
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


参数说明：

```plaintext
${indexName} zincIndex 索引名称

type  固定为 vector，表示向量索引
dims  向量维度
m     ivf_pq 索引所需参数，需要能被 dims整除
nbits ivf_pq 索引所需参数，默认为 8
vec_index_type 索引类型，支持 flat, ivf_pq 两种
```


### 写入包含向量的document



写入包含向量 document 与写入普通document 在 API层面并无差异，可自行选择合适的方式。



下面以 bluk API 为例

```plaintext
[POST] /es/_bulk

body:

{ "index" : { "_index" : "index1" } } 
{"name": "jack1","vec":[10.2,10.41,9.5,22.2]}
{ "index" : { "_index" : "index1" } } 
{"name": "jack2","vec":[10.2,11.41,9.5,22.2]}
{ "index" : { "_index" : "index1" } } 
{"name": "jack3","vec":[10.2,12.41,9.5,22.2]}
```


注意 _bulk API 严格要求每一行的格式，数据不能超过一行，详细请参考 [ES bulk](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)



修改和删除，也可以使用 bulk，删除 document 之后，其对应的向量数据同样会被删除



### 检索向量



通过传入一个 向量，搜索系统中N个相似的向量，并返回对应文档信息：

```plaintext
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


API 响应格式与 全文检索格式相同。



以下是参数说明：

```plaintext
${indexName} zincIndex 索引名称

query_field 要检索 index 中的哪个字段，字段必须为 vector 类型
k 要返回的 K 个最相似的向量数量
return_fields 单独返回的字段名称
vector 用于查询的向量
nprobe 仅对 ivf_pq 索引类型生效，要查询的聚蔟数量，数量越高，越精确
_source 用于控制是否返回 _source 字段，支持 bool或者一个数组，描述需要返回哪些字段
```


### 重建索引



立即对索引进行重建，适用于不等待后台自动检测的情况

```plaintext
[POST] /api/:target/:field/_rebuild
```


### 查询 recall



对于 ivf_pq 类型的向量，可以对其数据进行 recall 检查

```plaintext
[POST] /api/:target/_recall
{
    "field":"vec_001", # 要测试的字段
    "k":10, 
    "nprobe":5, # nprobe 数量
    "query_count":1000 # 进行测试的次数
}
```


# 向量检索使用示例



接下来实际演示如何 索引一批 papers，每个 paper 可能包含多个需要被索引的向量，我们希望通过 向量检索，得到最相似的 N 个向量，从而得到其对应的 paper-id。



## 创建 SeaSearch 索引与向量索引



首先是设定 向量索引的 mapping，在设定mapping时，index 和向量索引 会自动创建



由于 paper-id 只是一个普通的字符串，我们无需进行 analyze, 所以我们设置其类型为 keyword：

```plaintext
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


通过以上请求，我们创建了一个名为 paper 的 index，并为索引的 title-vec 字段，建立了 flat 类型的向量索引。



## 索引数据



我们通过 _bulk API 批量向 SeaSearch 写入这些 paper 数据

```plaintext
[POST] /es/_bulk

{ "index" : {"_index" : "paper" } } 
{"paper-id": "001","
{ "
{"paper-id": "002","title-vec":[10.2,11.40,9.5,22.2....]}
{ "
{"paper-id": "003","title-vec":[10.2,12.40,9.5,22.2....]}
....
```


## 检索数据



现在我们可以用向量检索：

```plaintext
[POST] /api/paper/_search/vector

{
    "query_field":"title-vec",
    "k":10,
    "return_fields":["paper-id"],
    "vector":[10.2,10.40,9.5,22.2....]
}
```


可以检索出最相似的向量对应的 document，并得到 paper-id。由于一个 paper 可能包含多个 向量，如果某个 paper 的多个向量都与查询的 向量 非常相似，那么这个 paper-id 可能出现在结果中多次。



## 维护向量数据



### 直接更新document



在一个 document 成功导入之后，SeaSearch会返回其 doc id，我们可以根据 doc id 直接更新一个document：

```plaintext
[POST] /es/_bulk

{ "update" : {"_id":"23gZX9eT6QM","_index" : "paper" } } 
{"paper-id": "005","vec":[10.2,1.43,9.5,22.2...]}
```


### 先查询再更新



如果没有保存返回的 doc id，可以先利用 SeaSearch 的全文检索功能，查询 paper-id 对应的docuemnts：

```plaintext
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


通过 DSL，我们可以直接检索到 paper-id 对应的 document 以及其 doc id。



### 全量更新 paper



一个 paper 包含多个向量，如果某个向量需要更新，那么我们直接更新这个向量对应的 document即可，但是在实际应用中，区分一个 paper的内容哪些是新增的，哪些是更新的，是不太容易的。



我们可以采用全量更新的方式：



  * 首先通过 DSL 查询出一个 paper 所有的 document

  * 删除所有的 document

  * 导入最新的 paper 数据






第2和第3步，可以在一个 批量 操作中进行。



下面的例子将演示删除 paper 001 的 document，并重新导入；同时，直接更新 paper 005 和 paper 006，因为它们只有一个向量：

```plaintext
[POST] /es/_bulk

{ "delete" : {"_id":"23gZX9eT6Q8","_index" : "paper" } } 
{ "delete" : {"_id":"23gZX9eT6Q0","_index" : "
{ "delete" : {"_id":"23gZX9eT6Q3","_index" : "
{ "index" : {"_index" : "
{"paper-id": "001","vec":[10.2,1.41,9.5,22.2...]}
{ "
{"
{ "
{"
{ "update" : {"_id":"23gZX9eT6QM","_index" : "paper" } } 
{"paper-id": "005","vec":[10.2,1.43,9.5,22.2...]}
{ "update" : {"_id":"23gZX9eT6QY","_index" : "paper" } } 
{"paper-id": "006","vec":[10.2,1.43,9.5,22.2...]}
```

