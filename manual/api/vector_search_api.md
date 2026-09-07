# Vector search API

SeaSearch supports vector search over embeddings generated from text, images, or other data. A vector search workflow has three steps:

1. Create a mapping with a vector field.
2. Index documents that contain vectors with the same dimension as the mapping.
3. Search the vector field for the nearest vectors.

For a complete workflow, see [Vector search example](vector_search_example.md).

## Create a vector index

Create a vector field with the mapping API. The `dims` value must match the length of every vector stored in the field and every query vector.

SeaSearch supports three vector index types:

| Index type | Search speed | Memory usage | Indexing speed | Recommended use |
| --- | --- | --- | --- | --- |
| `flat` | Slow | 100% | — | Small datasets and exact search |
| `ivf_pq` | Fast | 10% - 20% | Medium | Large datasets |
| `hnsw` | Fastest | 25% | Slow | High-performance, high-accuracy search |

- `flat` stores the input vectors directly and computes distances against all stored vectors. It is usually the best choice for small datasets.
- `ivf_pq` uses an inverted file index and product quantization to reduce memory usage and search time. See the [IVF-PQ overview](https://towardsdatascience.com/similarity-search-with-ivfpq-9c6348fd4db3/) for background.
- `hnsw` uses a hierarchical navigable small world graph. It provides high search performance and accuracy at the cost of additional memory.

!!! note
    SeaSearch 1.0.x supports `flat` and `ivf_pq`. Use `hnsw` only with SeaSearch 1.1.0 or later.

### Choose an index type

To choose the index type, you can consider the following factors:

1. Although `flat` index provides exact search results, it is usually not necessary for most use cases. Both `hnsw` and `ivf_pq` indexes can automatically fall back to a exact search mode when the dataset is small. In default configuration, `hnsw` performs a exact search when the number of vectors is fewer than 10K, while `ivf_pq` performs a exact search when the number of vectors is fewer than 100K. This avoids the overhead of maintaining and searching complex index structures for small datasets.
2. In most scenarios, you do not need to explicitly use the `flat` index. As a general guideline, use `hnsw` when the number of vectors is less than 10M because it provides high search performance and accuracy with reasonable memory usage. For larger datasets, `ivf_pq` is recommended because it significantly reduces memory consumption and provides efficient search performance at large scale.

### Mapping request

```http
PUT /es/{index_name}/_mapping
Content-Type: application/json
```

```json
{
  "properties": {
    "vec": {
      "type": "vector",
      "dims": 4,
      "vec_index_type": "flat"
    },
    "category": {
      "type": "keyword"
    }
  }
}
```

### Mapping parameters

| Parameter | Description |
| --- | --- |
| `index_name` | The name of the index. It is specified in the `{index_name}` path parameter of the mapping request. |
| `type` | Must be `vector`, indicating that the field stores vectors (embeddings) and can be searched with the vector search API. |
| `dims` | The dimension of each vector. |
| `vec_index_type` | The vector index type: `flat`, `ivf_pq`, or `hnsw` (SeaSearch 1.1.0 and later). |
| `m` | Required for `ivf_pq`. It must be greater than 0 and divide `dims` evenly. |
| `nbits` | Required for `ivf_pq`. It must be greater than 0; `4` or `8` are recommended. |

## Index documents containing vectors

Indexing a document with a vector uses the same document APIs as indexing a regular document. The following example uses the bulk API:

```http
POST /es/_bulk
Content-Type: application/x-ndjson
```

```json
{ "index": { "_index": "index1" } }
{ "name": "jack1", "category": "science", "vec": [10.2, 10.41, 9.5, 22.2] }
{ "index": { "_index": "index1" } }
{ "name": "jack2", "category": "technology", "vec": [10.2, 11.41, 9.5, 22.2] }
```

## Search vectors

You can search the index for the `k` most similar vectors to an input vector and return fields from the documents that contain those vectors:

```http
POST /api/{index_name}/_search/vector
Content-Type: application/json
```

```json
{
  "query_field": "vec",
  "k": 7,
  "return_fields": ["name"],
  "vector": [10.2, 10.40, 9.5, 22.2],
  "filter_query": {
    "term": {"category": "science"}
  }
}
```

The response uses the same format as full-text search. Results are sorted by vector distance, with the nearest vectors first.

### Search parameters

| Parameter | Required | Description |
| --- | --- | --- |
| `query_field` | Yes | The mapped field to search. It must be a vector field. |
| `k` | No | The number of nearest vectors to return. The default is `10`. |
| `return_fields` | No | Fields to return in the `fields` section of each hit. |
| `vector` | Yes | The query vector. Its length must equal the mapped `dims` value. |
| `nprobe` | No | For `ivf_pq`, the number of clusters to search. A larger value generally improves recall and increases latency. The default is `1`. |

### Filtering and source options

| Parameter | Required | Description |
| --- | --- | --- |
| `filter_query` | No | A full-text query object used to filter candidate documents before vector search. |
| `_source` | No | Controls which ordinary document source fields are returned in the `_source` section of each hit. |

## Recall Query

Use the recall API to evaluate an `ivf_pq` index and tune its parameters. The response contains a floating-point value between `0` and `1`; values closer to `1` indicate more accurate approximate search.

```http
POST /api/{index_name}/_recall
Content-Type: application/json
```

```json
{
  "field": "vec",
  "k": 10,
  "nprobe": 5,
  "query_count": 1000
}
```

| Parameter | Description |
| --- | --- |
| `field` | The vector field to evaluate. |
| `k` | The number of nearest vectors per query. The default is `10`. |
| `nprobe` | The number of IVF clusters to probe. The default is `5`. |
| `query_count` | The number of test queries. A larger value gives a more stable estimate. The default is `100`. |

You can tune the following parameters to improve recall:

- Increase `m` to improve accuracy, with additional computational and memory cost.
- Increase `nbits` to represent the original vectors more accurately, with lower compression.
- Increase `nprobe` to search more clusters, with additional query latency.
- `k` represents the number of most similar vectors returned in each query. If `k` is too low, it may generate random errors, leading to an inaccurate evaluation of recall.
