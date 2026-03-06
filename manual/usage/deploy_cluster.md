# Deploy SeaSearch in cluster (AMD64 CPU only)

## Cluster architecture

A SeaSearch cluster deployment consists of SeaSearch nodes (**recommended more than three nodes for high availability**), and a cluster gateway node. These nodes coordinate internal state and index allocation through etcd: 

![grafik](../media/seasearch_cluster_architecture.png)

- The SeaSearch node is responsible for reading and writing the index. The index is temporarily stored on the local disk as a cache. If the required data is not in the cache, it is read from S3. When a SeaSearch node goes offline, its indexes are reassigned to the remaining nodes, which then retrieve the latest data from S3.
- The cluster gateway node mainly consists of two processes: the cluster-manager process and the proxy process. The former monitors the health of the Seasearch nodes and assigns the most suitable node to new requests. The latter queries etcd for the metadata information of the most suitable node assigned by the cluster-manager and uses this metadata information to forward user requests to that node.

## Prerequisite

### Prepare S3 storage for storing indexes

You need to create a bucket on S3 storage provider for storing indexes

### Prepare etcd cluster

You need to prepare [etcd](https://etcd.io/docs/v3.6/install/) cluster (**recommended more than three nodes for high availability**) to store SeaSearch cluster metadata information. In most cases, you can install and configure etcd for each node as follows:

- Installation:
    ```sh
    apt update
    apt install -y etcd-server etcd-client
    service etcd start


    mkdir /opt/etcd-data/
    chmod 777 /opt/etcd-data/
    ```
- Configuraion (`vim /etc/default/etcd`):
    ```conf
    # etcd-1 ip: 172.16.1.1
    # etcd-2 ip: 172.16.1.2
    # etcd-3 ip: 172.16.1.3

    ETCD_NAME="etcd-1"
    ETCD_DATA_DIR="/opt/etcd-data/"
    ETCD_LISTEN_PEER_URLS="http://172.16.1.1:2380"
    ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://172.16.1.1:2379"
    ETCD_INITIAL_ADVERTISE_PEER_URLS="http://172.16.1.1:2380"
    ETCD_INITIAL_CLUSTER="etcd-1=http://172.16.1.1:2380,etcd-2=http://172.16.1.2:2380,etcd-3=http://172.16.1.3:2380"
    ETCD_INITIAL_CLUSTER_TOKEN="xxx" # your etcd token
    ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379,http://172.16.1.1:2379"
    ETCD_PROXY="off"
    ETCD_AUTO_COMPACTION_RETENTION="1"
    ```

    Then restart etcd with

    ```sh
    service etcd restart
    ```

## Deploy SeaSearch server (recommended more than three nodes)

The following assumptions and conventions are used in the rest of this document (applicable to **all nodes**, including cluster-gateway node):

* `/opt/seasearch` is the directory for storing SeaSearch docker compose files. If you decide to put SeaSearch in a different directory, adjust all paths accordingly.

### Download the yml and env file

You can download the `.yml` and `.env` files by following commands:

```bash
mkdir /opt/seasearch
cd /opt/seasearch
wget https://seasearch-manual.seafile.com/1.0/repo/cluster/seasearch/seasearch.yml
wget -O .env https://seasearch-manual.seafile.com/1.0/repo/cluster/seasearch/env
```

### Modify `.env` file

```env
INIT_SS_ADMIN_USER=<admin-username>  
INIT_SS_ADMIN_PASSWORD=<admin-password>
SS_CLUSTER_ID= # e.g., 1, 2, 3...., each node must be distinct and cannot be modified after initialization.
SS_ETCD_ENDPOINTS= # separated by commas, e.g., '192.168.0.1,192.168.0.2,192.168.0.3'
SS_ETCD_USERNAME=
SS_ETCD_PASSWORD=
SS_ETCD_PREFIX=/seasearch
```

### Start SeaSearch server

```
docker compose up -d
```

## Deploy cluster gateway (single node)

### Download the yml and env file

You can download the `.yml` and `.env` files by following commands:

```bash
mkdir /opt/seasearch
cd /opt/seasearch
wget https://seasearch-manual.seafile.com/1.0/repo/cluster/gateway/cluster-gateway.yml
wget -O .env https://seasearch-manual.seafile.com/1.0/repo/gateway/env
```

### Modify `.env` file

```env
SS_ETCD_ENDPOINTS= # separated by commas, e.g., '192.168.0.1,192.168.0.2,192.168.0.3'
SS_ETCD_USERNAME=
SS_ETCD_PASSWORD=
SS_ETCD_PREFIX=/seasearch
```

### Start cluster gateway node

```sh
docker compose up -d
```

### Register all SeaSearch nodes

You can register SeaSearch nodes by a command with

```sh
docker exec -it seasearch-cluster-gateway <SS_CLUSTER_ID_1>:<ip1>:4080 <SS_CLUSTER_ID_2>:<ip2>:4080 ...
```

For example, there are three SeaSearch nodes:
- Cluster ID 1 with IP 192.168.0.1
- Cluster ID 2 with IP 192.168.0.2
- Cluster ID 3 with IP 192.168.0.3

```sh
docker exec -it seasearch-cluster-gateway 1:192.168.0.1:4080 2:192.168.0.2:4080 3:192.168.0.3:4080
```

Now, you can access SeaSearch cluster at `http://<your cluster-gateway IP>:4082/` and login by the `INIT_SS_ADMIN_USER` and `INIT_SS_ADMIN_PASSWORD` defined in the `.env` file.
