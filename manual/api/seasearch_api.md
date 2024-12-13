# SeaSearch API Manual
## Overview
SeaSearch is developed based on ZincSearch and is compatible with ElasticSearch (ES) APIs. The concepts used in the API are similar to those in ElasticSearch, so users can directly refer to the ElasticSearch API documentation and ZincSearch API documentation for most API calls. This document introduces the commonly used APIs to help users quickly understand the main concepts and basic usage flow. It will also explain the modifications we made to the ZincSearch API and highlight the differences from the upstream API.

The ES-compatible APIs provided by SeaSearch can be accessed by adding the /es/ prefix in the URL. For example, the ES API URL is:
```
GET /my-index-000001/_search
```
The corresponding SeaSearch API URL is:
```
GET /es/my-index-000001/_search
```

## API Authentication
SeaSearch uses HTTP Basic Auth for authentication. API requests must include the corresponding basic auth token in the header.

To generate a basic auth token, combine the username and password with a colon (e.g., aladdin:opensesame), and then base64 encode the resulting string (e.g., YWxhZGRpbjpvcGVuc2VzYW1l).

You can generate a token using the following command, for example with aladdin:opensesame:

```
echo -n 'aladdin:opensesame' | base64
YWxhZGRpbjpvcGVuc2VzYW1l
```
Note: Basic auth is not secure. If you need to access SeaSearch over the public internet, it is strongly recommended to use HTTPS (e.g., via reverse proxy such as Nginx).
```
"Authorization": "Basic YWRtaW46MTIzNDU2Nzg="
```

### Administrator User
SeaSearch uses accounts to manage API permissions. When the program starts for the first time, an administrator account must be configured through environment variables.

Here is an example of setting the administrator account via shell:
```
set ZINC_FIRST_ADMIN_USER=admin
set ZINC_FIRST_ADMIN_PASSWORD=Complexpass#123
```
💡 In most scenarios, you can use the administrator account to provide access for applications. Only when you need to integrate multiple applications with different permissions, you should create regular users.

### Regular Users
You can create/update users via the API:
```
[POST] /api/user
{ 
    "_id": "prabhat",
    "name": "Prabhat Sharma",
    "role": "admin", // or user
    "password": "Complexpass#123"
}
```
To get all users:
```
[GET] /api/user
```
To delete a user:
```
[DELETE] /api/user/${userId}
```

## Index Management
In SeaSearch, users can create any number of indexes. An index is a collection of documents that can be searched, and a document can contain multiple searchable fields. Users specify the fields contained in the index via mappings and can customize the analyzers available to the index through settings. Each field can specify either a built-in or custom analyzer. The analyzer is used to split the content of a field into searchable tokens.

### Create Index
To create a SeaSearch index, you can configure the mappings and settings at the same time. For more details about mappings and settings, refer to the following sections.

ElasticSearch API: [Create Index](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)

### Configure Mappings
Mappings define the types and attributes of fields in a document. Users can configure the mapping via the API.

SeaSearch supports the following field types:

- text
- keyword
- numeric
- bool
- date
- vector

Other types, such as flattened, object, nested, etc., are not supported, and mappings do not support modifying existing fields (new fields can be added).

ElasticSearch Mappings API: [Put Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html)

ElasticSearch Mappings Explanation: [Mapping Types](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)

### Configure Settings
Index settings control the properties of the index. The most commonly used property is `analysis`, which allows you to customize the analyzers for the index. The analyzers defined here can be used by fields in the mappings.

ElasticSearch Settings API: [Update Settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html)

ElasticSearch related explanation:
- [Analyzer Concepts](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-concepts.html)
- [Specifying Analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html)

### Analyzer Support
Analyzers can be configured as default when creating an index, or they can be set for specific fields. (See the previous section for related concepts from the ES documentation.)

SeaSearch supports the following analyzers, which can be found here: [ZincSearch Documentation](https://zincsearch-docs.zinc.dev/api/index/analyze/). The concepts such as tokenization and token filters are consistent with ES and support most of the commonly used analyzers and tokenizers in ES.

### Chinese Analyzer
To enable the Chinese analyzer in the system, set the environment variable `ZINC_PLUGIN_GSE_ENABLE=true`.

If you need more comprehensive support for Chinese word dictionaries, set `ZINC_PLUGIN_GSE_DICT_EMBED = BIG`.

`GSE` is a standard analyzer, so you can directly assign the Chinese analyzer to fields in the mappings:
```
PUT /es/my-index/_mappings
{
  "properties": {
    "content": { 
        "type": "text",
        "analyzer": "gse_standard"
      }
  }
}
```
If users have custom tokenization habits, they can specify their dictionary files by setting the environment variable `ZINC_PLUGIN_GSE_DICT_PATH=${DICT_PATH}`, where `DICT_PATH` is the actual path to the dictionary files. The `user.txt` file contains the dictionary, and the `stop.txt` file contains stop words. Each line contains a single word.

GSE will load the dictionary and stop words from this path and use the user-defined dictionary to segment Chinese sentences.

### Document Operations
An index stores multiple documents. Users can perform CRUD operations (Create, Read, Update, Delete) on documents via the API. In SeaSearch, each document has a unique ID.

💡 Due to architectural design, SeaSearch’s performance for single document CRUD operations is much lower than that of ElasticSearch. Therefore, we recommend using batch operations whenever possible.

ElasticSearch Document APIs contain many additional parameters that are not meaningful to SeaSearch and are not supported. All query parameters are unsupported.

#### Create Document
ElasticSearch API: [Index Document](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)

#### Update Document
ElasticSearch’s update API supports partial updates to fields. SeaSearch only supports full document updates and does not support updating data via script or detecting if an update is a no-op.

If the document does not exist during an update, SeaSearch will create the corresponding document.

ElasticSearch API: [Update Document](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)

#### Delete Document 
Delete a document by its ID.

ElasticSearch API: [Delete Document](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete.html)

#### Get Document by ID
```
[GET] /api/${indexName}/_doc/${docId}
```

#### Batch Operations
It is recommended to use batch operations to update indexes.

ElasticSearch API: [Bulk Document API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)

### Search Documents
#### Query DSL
To perform full-text search, use the DSL. For usage, refer to:

[Query DSL Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

We do not support all query parameter options provided by ES. Unsupported parameters include: indices_boost, knn, min_score, retriever, pit, runtime_mappings, seq_no_primary_term, stats, terminate_after, version.

Search API: [Search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)

#### Delete by Query
To delete documents based on a query, use the delete-by-query operation. Like search, we do not support some ES parameters.

ElasticSearch API: [Delete by Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html)

#### Multi-Search
Multi-search supports searching multiple indexes and running different queries on each index.

ElasticSearch API: [Multi-Search API Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-multi-search.html)

We extended the multi-search to support using the same scoring information across different indexes for more accurate score calculation. To enable this, set `unify_score=true` in the query.

`unify_score` is meaningful only in this scenario: when searching the same query across multiple indexes. For example, in Seafile, when globally searching across all accessible repositories, each repository corresponds to an index. Enabling unify_score ensures consistent scoring across different repositories, providing more accurate search results.
```
[POST] /es/_msearch?unify_score=true
{"index": "t1"}
{"query": {"bool": {"should": [{"match": {"filename": {"query": "数据库", "minimum_should_match": "-25%"}}}, {"match": {"filename.ngram": {"query": "数据库", "minimum_should_match": "80
```
