# SeaSearch Configuration

## Single-Node Configurations

### Basic Configurations

```shell
# log mode of gin framework，default release
ZINC_WAL_ENABLE=true

# type of storage's engine, i.e., s3
ZINC_STORAGE_TYPE=

# the number of shards, since seaseach has one index per database, in order to improve loading efficiency, the default value is changed to 1
ZINC_SHARD_NUM=1
```

### S3 Storage Configurations

To enable s3 storage configurations, the term `ZINC_STORAGE_TYPE` has to be set as `ZINC_STORAGE_TYPE=s3`.

```shell
# the maximum local cache file size
ZINC_MAX_OBJ_CACHE_SIZE=

# S3 relative informations
ZINC_S3_ACCESS_ID=<your s3 access id>
ZINC_S3_USE_V4_SIGNATURE=<your s3 signature>
ZINC_S3_ACCESS_SECRET=<your s3 access secret>
ZINC_S3_ENDPOINT=<your s3 endpoint>
ZINC_S3_USE_HTTPS=<your s3 tls enabled>
ZINC_S3_PATH_STYLE_REQUEST=<your s3 style request path>
ZINC_S3_AWS_REGION=<your s3 AWS region>
```

## Cluster Configurations

### Basic Configurations

```shell
# default none for standalone deployment, optional to cluster, must be cluster for cluster deployment
ZINC_SERVER_MODE=

# cluster id，need to be globally unique
ZINC_CLUSTER_ID=<your cluster id>

ZINC_ETCD_ENDPOINTS=<your etcd address> 
ZINC_ETCD_USERNAME=<your etcd username>
ZINC_ETCD_PASSWORD=<your etcd password>
```

### Proxy Configurations
If the current node is a proxy node, the term `ZINC_SERVER_MODE` has to be set as **proxy** and the `ZINC_ETCD_ENDPOINTS` has to be pointed (i.e., =127.0.0.1:2379).

```shell
ZINC_CLUSTER_PROXY_LOG_DIR=/opt/seasearch/data/log 
ZINC_CLUSTER_PROXY_HOST=0.0.0.0
ZINC_CLUSTER_PROXY_PORT=4082
ZINC_ETCD_PREFIX=<yout etcd perfix, default /zinc>
ZINC_MAX_DOCUMENT_SIZE=1m # Bulk and multisearch limit on the maximum single document，default 1m 
ZINC_CLUSTER_MANAGER_ADDR=127.0.0.1:4081 # manager address
```

### Cluster-manger Configurations

```shell
ZINC_CLUSTER_MANAGER_LOG_DIR=/opt/seasearch/data/log
ZINC_CLUSTER_MANAGER_HOST=0.0.0.0
ZINC_CLUSTER_MANAGER_PORT=4081
ZINC_CLUSTER_MANAGER_ETCD_ENDPOINTS=<your etcd endpoints>
ZINC_CLUSTER_MANAGER_ETCD_PREFIX=<yout etcd perfix, default /zinc>
```

## Logs Configurations

```shell
ZINC_LOG_OUTPUT=true #whether to output logs to files, default yes
ZINC_LOG_DIR=/opt/seasearch/data/log #log directory
ZINC_LOG_LEVEL=debug #log level，default debug
```
