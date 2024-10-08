# Introduction

**SeaSearch** is a lightweight search engine will replace ElasticSearch as the default search engine, built on open source search engine ([ZincSearch](https://zincsearch-docs.zinc.dev/)) implemented in Go language. 

Why use **SeaSearch**:

- **Problems of ElasticSearch**:
    - Not designed for large number of indexes (like one index per library)
        - Need to search entire storage when searching inside a library
        - Need to filter out results that the user has permissions to acces
    - Can become slow when you have ~billion of files to search
    - Heavyweight Java program
    - Upgrade often requires rebuilding index

- **How about SeaSearch**:
    - Lightweight and can support one index per library
    - API compatible with ElasticSearch
    - Architecture Highlights
        - Cloud-native: can use S3 as storage (for single-node or cluster)
        - Shared storage: in a cluster, nodes use S3 as shared storage and store index metadata in etcd
        - Failover: node switching is easy thanks to shared storage architecture. ES replicates data between the nodes so consistency is more complex.