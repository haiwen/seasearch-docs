# Overview

SeaSearch is multi-tenant search engine with full-text indexing and vector indexing.

SeaSearch is compatible with ElasticSearch (ES) APIs. The concepts used in the API are similar to those in ElasticSearch, so users can directly refer to the [ElasticSearch API documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html) for most API calls. This document includes the commonly used APIs to help users quickly understand the main concepts and basic usage flow.


The ES-compatible APIs provided by SeaSearch can be accessed by adding the /es/ prefix in the URL. For example, the ES API URL is:

```
GET /my-index-000001/_search
```

The corresponding SeaSearch API URL is:

```
GET /es/my-index-000001/_search
```
