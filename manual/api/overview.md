# Overview
SeaSearch is developed based on ZincSearch and is compatible with ElasticSearch (ES) APIs. The concepts used in the API are similar to those in ElasticSearch, so users can directly refer to the [ElasticSearch API documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html) and [ZincSearch API documentation](https://zincsearch-docs.zinc.dev/api-es-compatible/) for most API calls. This document introduces the commonly used APIs to help users quickly understand the main concepts and basic usage flow. It will also explain the modifications we made to the ZincSearch API and highlight the differences from the upstream API.

The ES-compatible APIs provided by SeaSearch can be accessed by adding the /es/ prefix in the URL. For example, the ES API URL is:
```
GET /my-index-000001/_search
```
The corresponding SeaSearch API URL is:
```
GET /es/my-index-000001/_search
```
