# SeaSearch server changelog

## 1.0

### 1.0.5 (2026-09-10)

- Add log rotate
- [Fix] Content Length not being processed when proxy forwarding
- [Fix] No 404 response was returned when the proxy failed to obtain the index

### 1.0.4 (2026-05-22)

- [Fix] 500 errors during vector search
- SeaSearch cluster
- [Refact] Change Etcd key prefix default value to `/seasearch`
- [Fix] 406 error returned by the index deletion interface in the cluster
- [Fix] Process not exiting in time
- [Fix] Proxy modifying request length
- [Opt] Requires caching hashed passwords in memory

### 1.0.3 (2026-03-05)

- Support query vector from multi-indexes
- Add cache for list objects
- Forbit `_id` field while saving vector in documment indexes
- Conditional vector search

### 1.0.2 (2025-11-26)

- Support *float16* vector

### 1.0.1 (2025-07-09)

- Add inner stop words
- Adjust `stop.txt`
