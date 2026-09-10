# Vector search example

This example indexes papers whose content is represented by vectors. A paper can have multiple vector documents; the same `paper-id` can therefore appear more than once in the search results.

## Creating the SeaSearch Index and Vector Index

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
      "dims": 768,
      "vec_index_type": "flat"
    },
    "paper-id": {
      "type": "keyword"
    }
  }
}
```
With the above request, an index named paper is created, and a flat vector index is established for the content-vec field.

## Indexing Data

Use the bulk API to insert multiple vector documents.

```http
POST /es/_bulk
Content-Type: application/x-ndjson
```

```json
{ "index": { "_index": "paper" } }
{ "paper-id": "001", "content-vec": [10.2, 10.40, 9.5, 22.2, ...] }
{ "index": { "_index": "paper" } }
{ "paper-id": "001", "content-vec": [10.2, 10.41, 9.5, 22.2, ...] }
{ "index": { "_index": "paper" } }
{ "paper-id": "002", "content-vec": [10.2, 11.40, 9.5, 22.2, ...] }
{ "index": { "_index": "paper" } }
{ "paper-id": "003", "content-vec": [10.2, 12.40, 9.5, 22.2, ...] }
{ "index": { "_index": "paper" } }
{ "paper-id": "005", "content-vec": [10.2, 10.37, 9.5, 22.2, ...] }
{ "index": { "_index": "paper" } }
{ "paper-id": "006", "content-vec": [10.2, 10.38, 9.5, 22.2, ...] }
```

Save the document IDs returned by the bulk response if you need to update or delete individual vector documents later.

## Searching Data

The query below searches for the ten nearest vectors and returns the `paper-id` field.

```http
POST /api/paper/_search/vector
Content-Type: application/json
```

```json
{
  "query_field": "content-vec",
  "k": 10,
  "return_fields": ["paper-id"],
  "vector": [10.2, 10.40, 9.5, 22.2, ...]
}
```

This search returns the documents corresponding to the most similar vectors and provides the `paper-id`. A paper may contain multiple vectors, so the same `paper-id` can appear multiple times in the results.

## Maintain vector data

### Update a document directly

After a document is successfully indexed, SeaSearch returns its document ID. You can use this ID to update the document directly. Replace the placeholder ID with the ID returned by the indexing request.

```http
POST /es/_bulk
Content-Type: application/x-ndjson
```

```json
{ "update": { "_id": "<document-id-for-paper-005>", "_index": "paper" } }
{ "paper-id": "005", "content-vec": [10.2, 10.39, 9.5, 22.2, ...] }
```

### Query Then Update

If you did not save the returned document ID, you can first use SeaSearch's full-text search functionality to query the document or documents corresponding to a specific `paper-id`.

```http
POST /es/paper/_search
Content-Type: application/json
```

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {"paper-id": "003"}
        }
      ]
    }
  }
}
```

Using this query, you can retrieve the document associated with the `paper-id` along with its document ID.

### Replace all vector documents for a paper

A paper can contain multiple vector documents. If you know the document ID for an individual vector, you can update it directly. If the paper content is re-segmented or it is difficult to determine which existing vector each new vector replaces, replace all vector documents for the paper:

1. Query all documents for the paper by `paper-id` and record their document IDs.
2. Delete those documents.
3. Index the latest vector documents.

The delete and index operations can be included in the same bulk request:

```http
POST /es/_bulk
Content-Type: application/x-ndjson
```

```json
{ "delete": { "_id": "<document-id-for-paper-001-vector-1>", "_index": "paper" } }
{ "delete": { "_id": "<document-id-for-paper-001-vector-2>", "_index": "paper" } }
{ "index": { "_index": "paper" } }
{ "paper-id": "001", "content-vec": [10.2, 10.42, 9.5, 22.2, ...] }
{ "index": { "_index": "paper" } }
{ "paper-id": "001", "content-vec": [10.2, 10.43, 9.5, 22.2, ...] }
```
