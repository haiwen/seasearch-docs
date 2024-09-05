# 安装 SeaSearch

原版的 SeaSearch 采用纯 go 语言编写，直接通过 Go 编译工具即可编译。在我们引入向量检索功能时，用到了 faiss 库，这个库需要以 CGO 的方式调用，所以对 SeaSearch 的编译会产生影响。

## 安装 faiss

要在一台机器上编译或者运行 SeaSearch，需要这台机器安装 faiss 库。下面是具体安装步骤，适用于 x86 linux 机器，流程采用的操作系统为 debian 12，使用 apt 作为包管理器

### 前提条件

通过包管理器安装，如果连接速度慢，可以尝试更换源

ubuntu 参考：[https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

debian 参考：[https://mirrors.tuna.tsinghua.edu.cn/help/debian/](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)

换源之后，执行

```
sudo apt update
```

C++ 编译器，支持C++17及以上

可以通过 apt 安装

```
sudo apt install -y gcc
```

Cmake，3.23.1 以上，如果源不是最新，可以从 ppa 或者源码安装

```
sudo apt install -y cmake
```

wget swig gnupg libomp

```
sudo apt install -y wget swig gnupg libomp-dev
```

nodeJs;

```
sudo apt-get update && sudo apt-get install -y ca-certificates curl gnupg
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
sudo apt update && sudo apt install nodejs -y
```

### 安装 Intel MKL库 （可选，仅支持x86 cpu）

faiss 依赖 BLAS，并且推荐使用 intel MKL性能最佳

```
wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
| gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" |sudo tee /etc/apt/sources.list.d/oneAPI.list

sudo apt update

sudo apt install -y intel-oneapi-mkl-devel
```

执行完毕之后，MKL库就安装完毕了，再配置一个环境变量：

```
export MKL_PATH=/opt/intel/oneapi/mkl/latest/lib/intel64
```

### 非 x86 cpu 安装BLAS

非x86  cpu无法安装 MKL，可安装 OpenBLAS 实现

```
sudo apt install -y libatlas-base-dev libatlas3-base
```

### 编译 faiss

下载 faiss 源码，ssh方式：

```
git clone git@github.com:facebookresearch/faiss.git
```

或者 http方式：

```
git clone https://github.com/facebookresearch/faiss.git
```

进入 faiss 目录，如果安装了 MKL，执行：

```
cmake -B build -DFAISS_ENABLE_GPU=OFF \
    -DFAISS_ENABLE_C_API=ON \
    -DFAISS_ENABLE_PYTHON=OFF \
    -DBLA_VENDOR=Intel10_64_dyn  \
    -DBUILD_SHARED_LIBS=ON \
    "-DMKL_LIBRARIES=-Wl,--start-group;${MKL_PATH}/libmkl_intel_lp64.a;${MKL_PATH}/libmkl_gnu_thread.a;${MKL_PATH}/libmkl_core.a;-Wl,--end-group" \
    .
```

如果未安装 MKL，执行：

```
cmake -B build -DFAISS_ENABLE_GPU=OFF \
    -DFAISS_ENABLE_C_API=ON \
    -DFAISS_ENABLE_PYTHON=OFF \
    -DBUILD_SHARED_LIBS=ON=ON \
    -DBUILD_TESTING=OFF \    
    .
```

执行编译

```
make -C build
```

安装头文件：

```
sudo make -C build install
```

将编译好的动态链接库，拷贝到系统路径，这里的 /tmp/faiss 是faiss源码路径，替换为真正的路径即可：

```
sudo cp /tmp/faiss/build/c_api/libfaiss_c.so /usr/lib
```

完整安装脚本可以参考 SeaSearch 项目目录下的 /ci/install\_faiss.sh

## 编译 SeaSearch

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

## 编译 seasearch proxy 和 cluster manger

在集群下，需要编译部署 seasearch proxy 和 cluster manager

编译 proxy:

```
go build -o seasearch-proxy ./cmd/zinc-proxy/main.go
```

编译 cluster manager：

```
go build -o cluster-manager ./cmd/cluster-manager/main.go
```


## 发布

项目根目录下有 Dokcerfile 文件，可以根据此文件构建 docker 镜像

注意：构建此 docker 镜像，需要确保能正常访问 github，否则无法下载 faiss 源码会导致构建失败, 并且仅支持 x86 cpu，arm 需要设置 platform 参数模拟 x86

```
docker build -f ./Dockerfile .
```

## Mac 中存在的安装问题

### faiss 安装

faiss 可通过 brew install faiss 安装

### fatal error: 'faiss/c\_api/AutoTune\_c.h' file not found

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
