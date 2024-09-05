
# 编译 SeaSearch

faiss 已经安装完毕，可以开始编译 SeaSearch了

首先下载 SeaSearch源码：

```
git clone git@github.com:seafileltd/seasearch.git
```

或者 http方式：

```
git clone https://github.com/seafileltd/seasearch.git
```

编译前端静态文件

```
cd web
npm config set registry https://registry.npmmirror.com
npm install
npm run build
```

安装 go 语言环境 Go 1.20 以上

参考 [https://go.dev/doc/install](https://go.dev/doc/install)

需要确保启用了 CGO

```
export CGO_ENABLED=1
```

可选，更换 go 源：

```
go env -w  GOPROXY=https://goproxy.cn,direct
```

之后在项目根目录执行：

```
go build -o seasearch ./cmd/zincsearch/
```

以上步骤执行完毕，可以在项目的根目录下面得到最终的 seasearch 二进制文件了。

一般来说无需手动指定头文件和动态链接库位置，如果编译提示找不到头文件，或者找不到动态运行库，可以在编译时通过环境变量指定位置：

```
CGO_CFLAGS=-I /usr/local/include #你的C
CGO_LDFLAGS=-I /usr/lib  
```

如果运行时，提示找不到 动态链接库，可以通过：

```
LD_LIBRARY_PATH=/usr/lib #指定动态链接库目录
```

# 编译 seasearch proxy 和 cluster manger

在集群下，需要编译部署 seasearch proxy 和 cluster manager

编译 proxy:

```
go build -o seasearch-proxy ./cmd/zinc-proxy/main.go
```

编译 cluster manager：

```
go build -o cluster-manager ./cmd/cluster-manager/main.go
```


# 发布

项目根目录下有 Dokcerfile 文件，可以根据此文件构建 docker 镜像

注意：构建此 docker 镜像，需要确保能正常访问 github，否则无法下载 faiss 源码会导致构建失败, 并且仅支持 x86 cpu，arm 需要设置 platform 参数模拟 x86

```
docker build -f ./Dockerfile .
```

# Mac 中存在的安装问题

## faiss 安装

faiss 可通过 brew install faiss 安装

## fatal error: 'faiss/c\_api/AutoTune\_c.h' file not found

执行如下命令解决：

source: [https://github.com/DataIntelligenceCrew/go-faiss/issues/7](https://github.com/DataIntelligenceCrew/go-faiss/issues/7)

```
cd faiss
export CMAKE_PREFIX_PATH=/opt/homebrew/opt/openblas:/opt/homebrew/opt/libomp:/opt/homebrew
cmake -B build -DFAISS_ENABLE_GPU=OFF -DFAISS_ENABLE_C_API=ON -DBUILD_SHARED_LIBS=ON -DFAISS_ENABLE_PYTHON=OFF .
make -C build
sudo make -C build install
sudo cp build/c_api/libfaiss_c.dylib /usr/local/lib/libfaiss_c.dylib
```
