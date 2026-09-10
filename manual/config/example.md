# Example SeaSearch Configuration

## Initial Seasearch with Local Disk as Storage Backend

``` sh
  INIT_SS_ADMIN_USER=admin
  INIT_SS_ADMIN_PASSWORD=password
  SS_DATA_PATH=./data
```

## Initial Seasearch with S3 as Storage Backend

=== "AWS"
    ``` sh
    INIT_SS_ADMIN_USER=admin
    INIT_SS_ADMIN_PASSWORD=password
    SS_DATA_PATH=./data
    SS_STORAGE_TYPE=s3
    S3_KEY_ID=<your-s3-key-id>
    S3_SECRET_KEY=<your-s3-secret-key>
    S3_SS_BUCKET=<your-seasearch-bucket>
    SS_S3_REGION=us-east-1
    S3_USE_HTTPS=true
    S3_USE_V4_SIGNATURE=true
    ```
=== "Exoscale"
    ``` sh
    INIT_SS_ADMIN_USER=admin
    INIT_SS_ADMIN_PASSWORD=password
    SS_DATA_PATH=./data
    SS_STORAGE_TYPE=s3
    S3_KEY_ID=<your-s3-key-id>
    S3_SECRET_KEY=<your-s3-secret-key>
    S3_SS_BUCKET=<your-seasearch-bucket>
    S3_HOST=sos-de-fra-1.exo.io
    S3_PATH_STYLE_REQUEST=true
    ```
=== "Hetzner"
    ``` sh
    INIT_SS_ADMIN_USER=admin
    INIT_SS_ADMIN_PASSWORD=password
    SS_DATA_PATH=./data
    SS_STORAGE_TYPE=s3
    S3_KEY_ID=<your-s3-key-id>
    S3_SECRET_KEY=<your-s3-secret-key>
    S3_SS_BUCKET=<your-seasearch-bucket>
    S3_HOST=fsn1.your-objectstorage.com
    S3_PATH_STYLE_REQUEST=true
    S3_USE_HTTPS=true
    ```
=== "Other Public Hosted S3 Storag"
    ```sh
    INIT_SS_ADMIN_USER=admin
    INIT_SS_ADMIN_PASSWORD=password
    SS_DATA_PATH=./data
    SS_STORAGE_TYPE=s3
    S3_KEY_ID=<your-s3-key-id>
    S3_SECRET_KEY=<your-s3-secret-key>
    S3_SS_BUCKET=<your-seasearch-bucket>
    S3_HOST=<access endpoint for storage provider>
    SS_S3_REGION=<region name for storage provider>
    S3_USE_HTTPS=true
    ```
=== "Self-hosted S3 Storage"
    ```sh
    INIT_SS_ADMIN_USER=admin
    INIT_SS_ADMIN_PASSWORD=password
    SS_DATA_PATH=./data
    SS_STORAGE_TYPE=s3
    S3_KEY_ID=<your-s3-key-id>
    S3_SECRET_KEY=<your-s3-secret-key>
    S3_SS_BUCKET=<your-seasearch-bucket>
    S3_HOST=<your s3 api endpoint host>:<your s3 api endpoint port>
    S3_USE_HTTPS=true
    S3_PATH_STYLE_REQUEST=true
    S3_USE_HTTPS=true
    ```
