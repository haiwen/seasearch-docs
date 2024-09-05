# 安装 faiss

要在一台机器上编译或者运行 SeaSearch，需要这台机器安装 faiss 库。下面是具体安装步骤，适用于 x86 linux 机器，流程采用的操作系统为 debian 12，使用 apt 作为包管理器

## 前提条件

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

## 安装 Intel MKL库 （可选，仅支持x86 cpu）

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

## 非 x86 cpu 安装BLAS

非x86  cpu无法安装 MKL，可安装 OpenBLAS 实现

```
sudo apt install -y libatlas-base-dev libatlas3-base
```

## 编译 faiss

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
