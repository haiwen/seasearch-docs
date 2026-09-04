# Search Document

## Basic Search

You have to use DSL to perform search. For usage, refer to:

[Query DSL Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

We do not support all query parameter options provided by ES. Unsupported parameters include: indices_boost, knn, min_score, retriever, pit, runtime_mappings, seq_no_primary_term, stats, terminate_after, version.

Search API: [Search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)

## Multi-Search

Multi-search supports searching multiple indexes and running different queries on each index.

ElasticSearch API: [Multi-Search API Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-multi-search.html)

```
[POST] /es/_msearch
{"index": "t1"}
{"query": {"bool": {"should": [{"match": {"filename": {"query": "数据库", "minimum_should_match": "-25%"}}}, {"match": {"filename.ngram": {"query": "数据库", "minimum_should_match": "80
```

## Unified Search

When you search multiple indexes with the same query, the "/es/_search" and "/es/_msearch" APIs will calculate scores for each indexes separately. This means you cannot simply sort the results from multiple indexes by relevance. This is inconvenient in some use cases. For example, in Seafile, we create an index for each library. When we search a query for all accessible libraries, it's necesary to sort all results from all libraries based on unified scores.

To support such scenarios, we provide a unified search API. Please note that the query for each index should be the same except for [filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#filter-context).

```
[POST] /api/unified_search
{
    "index_queries": [
        {
            "index": "index1",
            "query": {}
        },
        {
            "index": "index2",
            "query": {}
        }
      ]
}
```


## Delete by Query

To delete documents based on a query, use the delete-by-query operation. Like search, we do not support some ES parameters.

ElasticSearch API: [Delete by Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html)