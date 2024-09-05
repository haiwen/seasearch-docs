# SeaSearch 配置项目

官方配置可以参考：[https://zincsearch-docs.zinc.dev/environment-variables/](https://zincsearch-docs.zinc.dev/environment-variables/)

以下配置说明，为我们扩展的配置项，所有配置，都是以环境变量的方式设置的。

## 扩展配置

```
GIN_MODE gin框架的日志模式，默认为 release
ZINC_WAL_ENABLE 是否启用 WAL，默认启用
ZINC_STORAGE_TYPE
ZINC_MAX_OBJ_CACHE_SIZE 启用 s3，oss时，本地最大缓存文件大小
ZINC_SHARD_LOAD_OBJS_GOROUTINE_NUM 索引加载并行度，在启用s3和Oss时，能提升索引载速度

ZINC_SHARD_NUM zincsearch 原有默认为 3，由于 seaseach 都是每个资料库一个索引，为了提升加载效率，改为默认为 1

s3相关，仅在 ZINC_STORAGE_TYPE=s3 时生效
ZINC_S3_ACCESS_ID
ZINC_S3_USE_V4_SIGNATURE
ZINC_S3_ACCESS_SECRET
ZINC_S3_ENDPOINT
ZINC_S3_USE_HTTPS
ZINC_S3_PATH_STYLE_REQUEST
ZINC_S3_AWS_REGION

oss相关，仅在 ZINC_STORAGE_TYPE=oss 时生效
ZINC_OSS_ACCESS_ID
ZINC_OSS_ACCESS_SECRET
ZINC_OSS_BUCKET
ZINC_OSS_ENDPOINT

集群相关
ZINC_SERVER_MODE 默认 none 为单机部署，可选 cluster,集群时必须为 cluster
ZINC_CLUSTER_ID 集群id，需要全局唯一
ZINC_ETCD_ENDPOINTS etcd 地址
ZINC_ETCD_ENDPOINTS etcd key前缀 默认 /zinc
ZINC_ETCD_USERNAME  etcd 用户名
ZINC_ETCD_PASSWORD  etcd 密码

日志相关
ZINC_LOG_OUTPUT 是否将日志输出到文件，默认 是
ZINC_LOG_DIR 日志目录，建议配置，默认为当前目录下的 log 子目录
ZINC_LOG_LEVEL 日志级别，默认 debug

```

## proxy 配置

```
ZINC_CLUSTER_PROXY_LOG_DIR=./log 
ZINC_CLUSTER_PROXY_HOST=0.0.0.0
ZINC_CLUSTER_PROXY_PORT=4082
ZINC_SERVER_MODE=proxy #必须为proxy
ZINC_ETCD_ENDPOINTS=127.0.0.1:2379
ZINC_ETCD_PREFIX=/zinc
ZINC_MAX_DOCUMENT_SIZE=1m #bulk和multisearch 对单个最大document的限制，默认1m
ZINC_CLUSTER_MANAGER_ADDR=127.0.0.1:4081 #manager 地址
```

## cluster-manger 配置

```
ZINC_CLUSTER_MANAGER_LOG_DIR=./log
ZINC_CLUSTER_MANAGER_HOST=0.0.0.0
ZINC_CLUSTER_MANAGER_PORT=4081
ZINC_CLUSTER_MANAGER_ETCD_ENDPOINTS=127.0.0.1:2379
ZINC_CLUSTER_MANAGER_ETCD_PREFIX=/zinc
```
