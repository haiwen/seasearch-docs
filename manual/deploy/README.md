# Deploy

Note: SeaSearch only supports deployment via docker now.

## Download the seasearch.yml

```bash
wget https://haiwen.github.io/seasearch-docs/repo/seasearch.yml
```

## Modify .env file

First, you need to specify the environment variables used by the SeaSearch image in the relevant `.env` file. Some environment variables can be found in [here](../config/README.md). Please add and modify the values (i.e., `<...>`) ​​of the following fields in the `.env` file.


```shell
# If seasearch uses a separate configuration file such as seasearch.yml, you need to write it into COMPOSE_FILE
COMPOSE_FILE='docker-compose.yml,seasearch.yml'

# other environment variables in .env file
# For Apple's chip (M2, e.g.), you should use the images with -nomkl tags (i.e., seafileltd/seasearch-nomkl:latest)
SEASEARCH_IMAGE=seafileltd/seasearch:latest

SEASEARCH_DATA_PATH=<persistent-volume-path-of-seasearch>
ZINC_FIRST_ADMIN_USER=<admin-username>  
ZINC_FIRST_ADMIN_PASSWORD=<admin-password>
```

Note: if new environment variables are added in .env, they also need to **be set synchronously** in the `seasearch.yml`

## Restart the Service

```shell
docker-compose down
docker-compose up
```

Browse seasearch services in [http://127.0.0.1:4080/](http://127.0.0.1:4080/).
