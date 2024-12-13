## Document Operations
An index stores multiple documents. Users can perform CRUD operations (Create, Read, Update, Delete) on documents via the API. In SeaSearch, each document has a unique ID.

!!! tip 
Due to architectural design, SeaSearch’s performance for single document CRUD operations is much lower than that of ElasticSearch. Therefore, we recommend using batch operations whenever possible.

ElasticSearch Document APIs contain many additional parameters that are not meaningful to SeaSearch and are not supported. All query parameters are unsupported.

### Create Document
ElasticSearch API: [Index Document](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)

### Update Document
ElasticSearch’s update API supports partial updates to fields. SeaSearch only supports full document updates and does not support updating data via script or detecting if an update is a no-op.

If the document does not exist during an update, SeaSearch will create the corresponding document.

ElasticSearch API: [Update Document](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)

### Delete Document 
Delete a document by its ID.

ElasticSearch API: [Delete Document](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete.html)

### Get Document by ID
```
[GET] /api/${indexName}/_doc/${docId}
```

### Batch Operations
It is recommended to use batch operations to update indexes.

ElasticSearch API: [Bulk Document API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)
