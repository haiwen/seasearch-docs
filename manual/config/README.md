# SeaSearch Configuration
For the official ZincSearch configuration, refer to: [ZincSearch Official Documentation](https://zincsearch-docs.zinc.dev/environment-variables/).

The following configuration options are the ones we’ve extended. All configurations are set via environment variables.

## Local Storage
- `SS_DATA_PATH`: Local storage path (default ./data). This is a required option and will be used as the SeaSearch system storage path (replaces the original `ZINC_DATA_PATH`).
  
## Object Storage
- `SS_STORAGE_TYPE`: Type of storage medium, default is disk. Possible options are s3, oss.
- `SS_MAX_OBJ_CACHE_SIZE`: When using object storage, the maximum local cache size. Default is 10GB.
- `SS_DATA_PATH`: Local storage path (default ./data). This is a required option and will be used for local cache storage when using object storage.

## S3
These configurations are only effective when `SS_STORAGE_TYPE=s3`.

- `SS_S3_ACCESS_ID`: S3 Access ID
- `SS_S3_USE_V4_SIGNATURE`: Use S3 V4 signature (default false)
- `SS_S3_ACCESS_SECRET`: S3 Access Key
- `SS_S3_ENDPOINT`: S3 endpoint
- `SS_S3_BUCKET`: S3 bucket
- `SS_S3_USE_HTTPS`: Use HTTPS to access S3
- `SS_S3_PATH_STYLE_REQUEST`: Use S3 path style request
- `SS_S3_AWS_REGION`: S3 Region

## OSS
These configurations are only effective when `SS_STORAGE_TYPE=oss`.

- `SS_OSS_ACCESS_ID`: OSS Access ID
- `SS_OSS_ACCESS_SECRET`: OSS Access Key
- `SS_OSS_BUCKET`: OSS storage bucket
- `SS_OSS_ENDPOINT`: OSS endpoint
  
## Logging
- `SEAFILE_LOG_TO_STDOUT`: Whether to output logs to standard output as part of the Seafile component (default false).
- `SEATABLE_LOG_TO_STDOUT`: Whether to output logs to standard output as part of the Seatable component (default false).
- `SS_LOG_DIR`: Log directory (default is a log subdirectory in the current directory).
- `SS_LOG_LEVEL`: Log level (default is debug).

## Example SeaSearch Configuration
### Enabling Local Disk as Storage Backend
```
export ZINC_FIRST_ADMIN_USER=admin
export ZINC_FIRST_ADMIN_PASSWORD=password
export SS_DATA_PATH=./data
```
### Enabling OSS as Storage Backend
```
export ZINC_FIRST_ADMIN_USER=admin
export ZINC_FIRST_ADMIN_PASSWORD=password
export SS_DATA_PATH=./data
export SS_STORAGE_TYPE=oss
export SS_OSS_ACCESS_ID=admin
export SS_OSS_ACCESS_SECRET=password
export SS_OSS_BUCKET=seaserach_bucket
export SS_OSS_ENDPOINT=oss-cn-beijing.aliyuncs.com
```
### Enabling S3 as Storage Backend
```
export ZINC_FIRST_ADMIN_USER=admin
export ZINC_FIRST_ADMIN_PASSWORD=password
export SS_DATA_PATH=./data
export SS_STORAGE_TYPE=s3
export SS_S3_ACCESS_ID=admin
export SS_S3_ACCESS_SECRET=password
export SS_S3_BUCKET=seaserach_bucket
export SS_S3_REGION=us-east-1
```
