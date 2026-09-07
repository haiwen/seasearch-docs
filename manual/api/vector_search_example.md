# Vector search example

This example indexes papers whose content is represented by vectors. A paper can have multiple vector documents; the same `paper-id` can therefore appear more than once in the search results.

The example uses four-dimensional vectors so that every request body is complete and can be copied directly into an API client.

## Create the index and vector field

The mapping request creates the `paper` index and its vector index. The `paper-id` field is a `keyword` because it is used as an identifier and does not need text analysis.

```http
PUT /es/paper/_mapping
Content-Type: application/json
```

```json
{
  "properties": {
    "content-vec": {
      "type": "vector",
      "dims": 4,
      "vec_index_type": "flat"
    },
    "paper-id": {
      "type": "keyword"
    }
  }
}
```

## Index paper vectors

Use the bulk API to insert multiple vector documents. This sample gives paper `001` two vectors and gives papers `002`, `003`, `005`, and `006` one vector each.

```http
POST /es/_bulk
Content-Type: application/x-ndjson
```

```json
{ "index": { "_index": "paper" } }
{ "paper-id": "001", "content-vec": [10.2, 10.40, 9.5, 22.2] }
{ "index": { "_index": "paper" } }
{ "paper-id": "001", "content-vec": [10.2, 10.41, 9.5, 22.2] }
{ "index": { "_index": "paper" } }
{ "paper-id": "002", "content-vec": [10.2, 11.40, 9.5, 22.2] }
{ "index": { "_index": "paper" } }
{ "paper-id": "003", "content-vec": [10.2, 12.40, 9.5, 22.2] }
{ "index": { "_index": "paper" } }
{ "paper-id": "005", "content-vec": [10.2, 10.37, 9.5, 22.2] }
{ "index": { "_index": "paper" } }
{ "paper-id": "006", "content-vec": [10.2, 10.38, 9.5, 22.2] }
```

Save the document IDs returned by the bulk response if you need to update or delete individual vector documents later.

## Search the paper vectors

The query below is closest to the first vector in the sample data. It returns the `paper-id` field for the ten nearest vectors, or fewer if the index contains fewer than ten documents.

```http
POST /api/paper/_search/vector
Content-Type: application/json
```

```json
{
  "query_field": "content-vec",
  "k": 10,
  "return_fields": ["paper-id"],
  "vector": [10.2, 10.40, 9.5, 22.2]
}
```

Because paper `001` has two vector documents, its `paper-id` can appear twice when both vectors are among the nearest results.

## Maintain vector data

### Update a document directly

Bulk update replaces the complete document. Replace the placeholder IDs with IDs returned by the indexing request.

```http
POST /es/_bulk
Content-Type: application/x-ndjson
```

```json
{ "update": { "_id": "<document-id-for-paper-005>", "_index": "paper" } }
{ "paper-id": "005", "content-vec": [10.2, 10.39, 9.5, 22.2] }
```

### Find a document ID, then update it

If you did not save a document ID, use full-text search to find the document by `paper-id`:

```http
POST /es/paper/_search
Content-Type: application/json
```

```json
{
  "query": {
    "term": {"paper-id": "003"}
  }
}
```

Use the returned `_id` in a bulk update request like the one above.

### Replace all vectors for a paper

When a paper contains multiple vectors, it is often simpler to replace all of its vector documents:

1. Query all documents for the paper and record their document IDs.
2. Delete those documents.
3. Index the latest vectors.

The delete and re-index operations can be sent in one bulk request. The following request also updates papers `005` and `006`, which each have one vector:

```http
POST /es/_bulk
Content-Type: application/x-ndjson
```

```json
{ "delete": { "_id": "<document-id-for-paper-001-vector-1>", "_index": "paper" } }
{ "delete": { "_id": "<document-id-for-paper-001-vector-2>", "_index": "paper" } }
{ "index": { "_index": "paper" } }
{ "paper-id": "001", "content-vec": [10.2, 10.42, 9.5, 22.2] }
{ "index": { "_index": "paper" } }
{ "paper-id": "001", "content-vec": [10.2, 10.43, 9.5, 22.2] }
{ "update": { "_id": "<document-id-for-paper-005>", "_index": "paper" } }
{ "paper-id": "005", "content-vec": [10.2, 10.39, 9.5, 22.2] }
{ "update": { "_id": "<document-id-for-paper-006>", "_index": "paper" } }
{ "paper-id": "006", "content-vec": [10.2, 10.40, 9.5, 22.2] }
```
