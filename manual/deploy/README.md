# Deploy

Note: SeaSearch only supports deployment via docker now.

## Modify .env file

First, you need to specify the environment variables used by the SeaSearch image in the relevant `.env` file. Some environment variables can be found in [here](../config/README.md). Please add and modify the values (i.e., `<...>`) ​​of the following fields in the `.env` file


```shell
# If seasearch uses a separate configuration file such as seasearch.yml, you need to write it into COMPOSE_FILE
COMPOSE_FILE='docker-compose.yml,seasearch.yml'

# other environment variables in .env file

SEASEARCH_DATA_PATH=<persistent-volume-path-of-seasearch>
ZINC_FIRST_ADMIN_USER=<admin-username>  
ZINC_FIRST_ADMIN_PASSWORD=<admin-password>
```

## SeaSearch.yml

```yml
services:  
  seasearch:  
    image: ${SEASEARCH_IMAGE:-seafileltd/seasearch:latest}
    container_name: seasearch  
    volumes:  
      - ${SEASEARCH_DATA_PATH}:/opt/seasearch/data  
    ports:  
      - "4080:4080" 
    environment: 
      # Note: if new environment variables are added in .env, they also need to be set synchronously here
      - ZINC_FIRST_ADMIN_USER=${ZINC_FIRST_ADMIN_USER}
      - ZINC_FIRST_ADMIN_PASSWORD=${ZINC_FIRST_ADMIN_PASSWORD}
    networks:
      - frontend-net
      - backend-scheduler-net


networks:
  frontend-net:
    name: frontend-net
  backend-scheduler-net:
    name: backend-scheduler-net

```

Restart the services

```shell
docker-compose down
docker-compose up
```

Browse seasearch services in [http://127.0.0.1:4080/](http://127.0.0.1:4080/).
