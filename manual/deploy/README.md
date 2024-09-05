# 启动 SeaSearch

## 启动单机

对于开发环境而言，只需要按照官方说明，配置 启动帐号和启动密码两个 环境变量即可。

编译 SeaSearch 参考： [Setup](../setup/README.md)

对于开发环境，直接配置环境变量，并启动二进制文件即可；

以下命令会首先创建一个 data文件夹，作为默认的存储路径，之后以 admin 以及 Complexpass#123作为初始用户，启动一个 SeaSearch 程序，并默认监听4080端口：

```
mkdir data
ZINC_FIRST_ADMIN_USER=admin ZINC_FIRST_ADMIN_PASSWORD=Complexpass#123 GIN_MODE=release ./SeaSearch
```

如果需要重置数据，删除整个 data 目录再重启即可，这会清理所有元数据以及索引数据。

## 启动集群

1. 启动 etcd

2. 启动 SeaSearch 节点，节点会自动向 etcd 注册心跳。

3. 启动 cluster-manager，然后通过 API 或者 直接向 etcd 设置 cluster-info，设置SeaSearch 节点的地址。并且同时，cluster-manager 开始根据节点心跳对分片进行分配。

4. 启动 SeaSearch-proxy，此时就可以对外提供服务了。
