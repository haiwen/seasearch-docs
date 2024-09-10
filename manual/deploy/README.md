# Launch SeaSearch

## Start a single machine

For the development environment, you only need to follow the official instructions to configure the two environment variables of the startup account and startup password.

Compile SeaSearch reference： [Setup](../setup/README.md)

For the development environment, simply configure the environment variables and start the binary file

The following command will first create a data folder as the default storage path, then start a SeaSearch program with admin and xxx as the initial users, and listen to port 4080 by default:

```
mkdir data
ZINC_FIRST_ADMIN_USER=admin ZINC_FIRST_ADMIN_PASSWORD=xxx GIN_MODE=release ./SeaSearch
```

If you need to reset the data, just delete the entire data directory and restart, which will clean up all metadata and index data.

## Start the cluster

1. Start etcd

2. Start the SeaSearch node, which will automatically register its heartbeat with etcd.

3. Start cluster-manager, then set the address of the SeaSearch node through the API or directly set cluster-info to etcd. At the same time, cluster-manager starts to allocate shards based on the node heartbeat.

4. Start SeaSearch-proxy, and you can now provide services to the outside world.
