## Vecoter Search
SeaSearch supports vector search. You can create an index containing a vector field. You can then save vectors (embeddings) generated from texts or images into the index. We'll introduce related APIs in this document.

### Creating a Vector Index
To use the vector search feature, you first have to create an index containing a vector field. This can be done via mapping API.

SeaSearch supports two types of vector index fields: `flat` and `ivfpq`.
- Flat index directly save the input vectors. When searching for a vector, it simply computes the distances between the input vector and the saved vectors and returns the top K vectors nearest to the input vector. It's recommended to use this type if you have less than 100K vectors in the index.
- [IVFPQ](https://towardsdatascience.com/similarity-search-with-ivfpq-9c6348fd4db3/) index uses a more efficient data structure to save the vectors. It requires less memory and less time to search for input vectors. If you have more than 100K vectors, you may consider using IVFPQ index.

For example, we create an index containing a vector field named "vec". The index type is set to `flat` and the vector dimension is `768`.

```
[PUT] /es/${indexName}/_mapping

body:
{
  "properties": {
    "vec": {
      "type": "vector",
      "dims": 768,
      "vec_index_type": "flat"
    }
  }
}
```

You can specify the following parameters for vector indexes:

- `${indexName}`: The name of the index.
- `type`: Fixed as `vector`, indicating that this is a vector index.
- `dims`: The vector dimension.
- `m`: A parameter required for the `ivf_pq` index, it must be divisible by dims. For example, the `dims` is `768`, the `m` could be `192`.
- `nbits`: A parameter required for the `ivf_pq` index, must grater than 0, we recommend `4` or `8`.
- `vec_index_type`: The index type. Supported types are `flat` and `ivf_pq`.

### Indexing Documents Containing Vectors

Indexing a document that includes vectors is identical at the API level to indexing a regular document. 
You may choose the method that best suits your needs. The following example uses the bulk API.

```
[POST] /es/_bulk

{ "index" : { "_index" : "index1" } }
{"name": "jack1", "vec": [10.2, 10.41, 9.5, 22.2]}
{ "index" : { "_index" : "index1" } }
{"name": "jack2", "vec": [10.2, 11.41, 9.5, 22.2]}
{ "index" : { "_index" : "index1" } }
{"name": "jack3", "vec": [10.2, 12.41, 9.5, 22.2]}
```

### Searching Vectors

You can search the index for the top K similar vectors to the input vector and return information from the documents that contain the vectors.

```
[POST] /api/${indexName}/_search/vector

{
  "query_field": "vec",
  "k": 7,
  "return_fields": ["name"],
  "vector": [10.2, 10.40, 9.5, 22.2, ...]  
}
```
The API response format is the same as that for full-text search.

You can specify the following parameters for vector search:

- `${indexName}`: The name of the index.
- `query_field`: The field in the index to search. This field must be of type vector.
- `k`: The number of most similar vectors to return.
- `return_fields`: The names of the fields to return separately.
- `vector`: The vector used for querying.
- `nprobe`: Applicable only to the `ivf_pq` index type; it specifies the number of clusters to search. The higher the number, the more accurate the results.


### Recall Query
For vector indexes of the `ivf_pq` type, you can evaluate the recall for search on the data. You can optimize and adjust your `ivf_pq` index parameters based on the results of this API.
It returns a floating point number between 0 and 1, representing the recall rate. The closer it is to 1, the more accurate the search results are.

```
[POST] /api/${indexName}/_recall

{
  "field": "vec_001",  
  "k": 10,
  "nprobe": 5,         
  "query_count": 1000
}
```

You can specify the following parameters for vector recall:

- `${indexName}`: The name of the index.
- `field`: The field in the index to test. This field must be of type vector.
- `k`: The number of most similar vectors to return, with default of `10`.
- `nprobe`: The number of probed clusters. The default is `5`.
- `query_count`: The higher this number, the more accurate the results will be, with a default of 100.

You can try adjusting the following vecotr index parameters to improve recall:

-  `m`: The larger the value of `m`, the higher the accuracy, but it will also increase the computational complexity and memory usage.
-  `nbits`: The larger the `nbits` value, the larger the encoding book, and the more accurately the original vector can be represented. However, this also means that more bits are needed to store each index, resulting in a lower compression rate.
-  `k`: Represents the number of most similar vectors returned in each query. If k is too low, it may generate random errors, leading to an inaccurate evaluation of recall.
-  `nprobe`: A higher value allows you to search for more clusters, improving the recall rate by expanding the search scope, but at the cost of increasing the query latency.

## Example of Using Vector Search

Below is an example demonstrating how to index a batch of papers. 
Each paper may contain multiple vectors that need to be indexed. 
We aim to retrieve the N most similar vectors via vector search and thereby obtain the corresponding `paper-id`.

### Creating the SeaSearch Index and Vector Index

First, set up the mapping for the vector index. When you define the mapping, both the index and the vector index will be created automatically. 
Since `paper-id` is just a plain string and does not require analysis, we set its type to `keyword`.

```
[PUT] /es/paper/_mapping

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
With the above request, an index named `paper` is created, and a `flat` vector index is established for the `content-vec` field.

### Indexing Data

We use the `_bulk` API to batch insert these `paper` data into SeaSearch.

```
[POST] /es/_bulk

{ "index" : {"_index" : "paper" } }
{"paper-id": "001", "content-vec": [10.2, 10.40, 9.5, 22.2, ...]}
{ "index" : {"_index" : "paper" } }
{"paper-id": "002", "content-vec": [10.2, 11.40, 9.5, 22.2, ...]}
{ "index" : {"_index" : "paper" } }
{"paper-id": "003", "content-vec": [10.2, 12.40, 9.5, 22.2, ...]}
```

### Searching Data
Now you can perform a vector search.

```
[POST] /api/paper/_search/vector

{
  "query_field": "content-vec",
  "k": 10,
  "return_fields": ["paper-id"],
  "vector": [10.2, 10.40, 9.5, 22.2, ...]
}
```

This search returns the documents corresponding to the most similar vectors and provides the `paper-id`. 
Note that a paper may contain multiple vectors, if several vectors of the same paper are very similar to the query vector, that `paper-id` might appear multiple times in the results.

### Maintaining Vector Data

#### Directly Updating a Document
After a document is successfully imported, SeaSearch returns its document ID. You can use this doc id to directly update the document.

```
[POST] /es/_bulk

{ "update" : {"_id": "23gZX9eT6QM", "_index": "paper" } }
{"paper-id": "005", "vec": [10.2, 1.43, 9.5, 22.2, ...]}
```

#### Query Then Update

If you did not save the returned doc id, you can first use SeaSearch's full-text search functionality to query the document(s) corresponding to a specific `paper-id`.

```
[POST] /es/paper/_search

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
Using DSL, you can retrieve the document associated with the `paper-id` along with its doc id.

#### Full Update for a Paper

A paper may contain multiple vectors. If a particular vector needs updating, you can update the document corresponding to that vector directly. 
However, in practice, it can be difficult to distinguish which parts of a paper are new versus which are updates.

A full update approach can be adopted as follows:
1.Use DSL to query all documents for a paper.
2.Delete all of the documents.
3.Import the latest paper data.

Steps 2 and 3 can be executed in a single bulk operation.

The following example demonstrates deleting the documents for paper `001` and re-importing them. 
At the same time, it directly updates paper `005` and paper `006` (since they each have only one vector):

```
[POST] /es/_bulk

{ "delete" : {"_id": "23gZX9eT6Q8", "_index": "paper" } }
{ "delete" : {"_id": "23gZX9eT6Q0", "_index": "paper" } }
{ "delete" : {"_id": "23gZX9eT6Q3", "_index": "paper" } }
{ "index" : {"_index": "paper" } }
{"paper-id": "001", "content-vec": [10.2, 1.41, 9.5, 22.2, ...]}
{ "index" : {"_index": "paper" } }
{"paper-id": "001", "content-vec": [10.2, 1.42, 9.5, 22.2, ...]}
{ "index" : {"_index": "paper" } }
{"paper-id": "001", "content-vec": [10.2, 1.43, 9.5, 22.2, ...]}
{ "update" : {"_id": "23gZX9eT6QM", "_index": "paper" } }
{"paper-id": "005", "content-vec": [10.2, 1.43, 9.5, 22.2, ...]}
{ "update" : {"_id": "23gZX9eT6QY", "_index": "paper" } }
{"paper-id": "006", "content-vec": [10.2, 1.43, 9.5, 22.2, ...]}
```
