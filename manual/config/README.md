# SeaSearch Configuration

The official configuration can be referenced：[https://zincsearch-docs.zinc.dev/environment-variables/](https://zincsearch-docs.zinc.dev/environment-variables/)

The following configuration instructions are for our extended configuration items. All configurations are set in the form of environment variables.

## Extended configuration

```
GIN_MODE, log mode of gin framework，default release
ZINC_WAL_ENABLE, whether to enable WAL，defaule enabled
ZINC_STORAGE_TYPE
ZINC_MAX_OBJ_CACHE_SIZE, when s3 and oss are enabled, the maximum local cache file size
ZINC_SHARD_LOAD_OBJS_GOROUTINE_NUM, index loading parallelism, when S3 and oss are enabled, can improve the index loading speed

ZINC_SHARD_NUM zincsearch the original default value is 3. Since seaseach has one index per database, in order to improve loading efficiency, the default value is changed to 1

S3 related, only valid when ZINC_STORAGE_TYPE=s3
ZINC_S3_ACCESS_ID
ZINC_S3_USE_V4_SIGNATURE
ZINC_S3_ACCESS_SECRET
ZINC_S3_ENDPOINT
ZINC_S3_USE_HTTPS
ZINC_S3_PATH_STYLE_REQUEST
ZINC_S3_AWS_REGION

OSS related, only valid when ZINC_STORAGE_TYPE=oss
ZINC_OSS_ACCESS_ID
ZINC_OSS_ACCESS_SECRET
ZINC_OSS_BUCKET
ZINC_OSS_ENDPOINT

cluster related
ZINC_SERVER_MODE, default none for standalone deployment, optional to cluster, must be cluster for cluster deployment
ZINC_CLUSTER_ID, cluster id，need to be globally unique
ZINC_ETCD_ENDPOINTS, etcd address
ZINC_ETCD_ENDPOINTS, etcd key prefix, default /zinc
ZINC_ETCD_USERNAME,  etcd username
ZINC_ETCD_PASSWORD,  etcd password

log related
ZINC_LOG_OUTPUT, whether to output logs to files, default yes
ZINC_LOG_DIR, log directory, recommended configuration, default is the log subdirectory under the current directory
ZINC_LOG_LEVEL, log level，default debug

```

## proxy configuration

```
ZINC_CLUSTER_PROXY_LOG_DIR=./log 
ZINC_CLUSTER_PROXY_HOST=0.0.0.0
ZINC_CLUSTER_PROXY_PORT=4082
ZINC_SERVER_MODE=proxy # must be proxy
ZINC_ETCD_ENDPOINTS=127.0.0.1:2379
ZINC_ETCD_PREFIX=/zinc
ZINC_MAX_DOCUMENT_SIZE=1m # Bulk and multisearch limit on the maximum single document，default 1m 
ZINC_CLUSTER_MANAGER_ADDR=127.0.0.1:4081 # manager address
```

## cluster-manger configuration

```
ZINC_CLUSTER_MANAGER_LOG_DIR=./log
ZINC_CLUSTER_MANAGER_HOST=0.0.0.0
ZINC_CLUSTER_MANAGER_PORT=4081
ZINC_CLUSTER_MANAGER_ETCD_ENDPOINTS=127.0.0.1:2379
ZINC_CLUSTER_MANAGER_ETCD_PREFIX=/zinc
```
