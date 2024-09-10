# Installation of SeaSearch

The original version of SeaSearch is written in pure Go language and can be compiled directly through the Go compilation tool. When we introduced the vector search function, we used the faiss library, which needs to be called in CGO mode, so it will affect the compilation of SeaSearch.

## Installation of faiss

To compile or run SeaSearch on a machine, you need to install the faiss library on that machine. The following are the specific installation steps, which are applicable to x86 linux machines. The operating system used in the process is debian 12, using apt as the package manager.

### Prerequisites

Install through the package manager. If the connection speed is slow, you can try changing the source

ubuntu reference：[https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

debian reference：[https://mirrors.tuna.tsinghua.edu.cn/help/debian/](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)

After changing the source, run

```
sudo apt update
```

C++ compiler, supports C++17 and above

Can be installed via apt

```
sudo apt install -y gcc
```

Cmake, 3.23.1 or above, if the source is not the latest, you can install it from ppa or source code

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

### Install Intel MKL library (optional, only supports x86 CPU)

faiss relies on BLAS, and Intel MKL is recommended for best performance.

```
wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
| gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" |sudo tee /etc/apt/sources.list.d/oneAPI.list

sudo apt update

sudo apt install -y intel-oneapi-mkl-devel
```

After the execution is completed, the MKL library is installed. Then configure an environment variable:

```
export MKL_PATH=/opt/intel/oneapi/mkl/latest/lib/intel64
```

### Install BLAS on non-x86 CPUs

MKL cannot be installed on non-x86 CPUs, but can be instead by installing OpenBLAS

```
sudo apt install -y libatlas-base-dev libatlas3-base
```

### compile faiss

Download faiss source code by ssh:

```
git clone git@github.com:facebookresearch/faiss.git
```

or by https：

```
git clone https://github.com/facebookresearch/faiss.git
```

If MKL is installed, go to the faiss directory and run:

```
cmake -B build -DFAISS_ENABLE_GPU=OFF \
    -DFAISS_ENABLE_C_API=ON \
    -DFAISS_ENABLE_PYTHON=OFF \
    -DBLA_VENDOR=Intel10_64_dyn  \
    -DBUILD_SHARED_LIBS=ON \
    "-DMKL_LIBRARIES=-Wl,--start-group;${MKL_PATH}/libmkl_intel_lp64.a;${MKL_PATH}/libmkl_gnu_thread.a;${MKL_PATH}/libmkl_core.a;-Wl,--end-group" \
    .
```

If MKL is not installed, run:

```
cmake -B build -DFAISS_ENABLE_GPU=OFF \
    -DFAISS_ENABLE_C_API=ON \
    -DFAISS_ENABLE_PYTHON=OFF \
    -DBUILD_SHARED_LIBS=ON=ON \
    -DBUILD_TESTING=OFF \    
    .
```

Installing the C++ library

```
make -C build
```

Installing the headers

```
sudo make -C build install
```

Copy the compiled dynamic link library to the system path. Here /tmp/faiss is the faiss source code path. Replace it with the real path:

```
sudo cp /tmp/faiss/build/c_api/libfaiss_c.so /usr/lib
```

For the complete installation script, please refer to /ci/install\_faiss.sh in the SeaSearch project directory.

## Compile SeaSearch

Faiss has been installed, you can start compiling SeaSearch

Download the SeaSearch source code using ssh：

```
git clone git@github.com:seafileltd/seasearch.git
```

or by https：

```
git clone https://github.com/seafileltd/seasearch.git
```

Compile frontend static files

```
cd web
npm config set registry https://registry.npmmirror.com
npm install
npm run build
```

Install the go language environment Go 1.20 or above

reference [https://go.dev/doc/install](https://go.dev/doc/install)

You need to make sure CGO is enabled

```
export CGO_ENABLED=1
```

Optional, replace the go source:

```
go env -w  GOPROXY=https://goproxy.cn,direct
```

Run in the project root directory：

```
go build -o seasearch ./cmd/zincsearch/
```

After completing the above steps, you can get the final seasearch binary file in the root directory of the project.

Generally, there is no need to manually specify the location of header files and dynamic link libraries. If the compilation prompt says that the header file cannot be found, or the dynamic runtime library cannot be found, you can specify the location through environment variables during compilation:

```
CGO_CFLAGS=-I /usr/local/include # Your default installation path for C header files
CGO_LDFLAGS=-I /usr/lib  
```

If the runtime prompts that the dynamic link library cannot be found, you can use:

```
LD_LIBRARY_PATH=/usr/lib # Specify the dynamic link library directory
```

## Compile seasearch proxy and cluster manger

In a cluster, you need to compile and deploy seasearch proxy and cluster manager

Compile proxy:

```
go build -o seasearch-proxy ./cmd/zinc-proxy/main.go
```

Compile cluster manager：

```
go build -o cluster-manager ./cmd/cluster-manager/main.go
```


## Publish

There is a Dokcerfile file in the project root directory, and you can build a docker image based on this file

Note: To build this docker image, you need to ensure that you can access github normally, otherwise you will not be able to download the faiss source code, which will cause the build to fail. It only supports x86 cpu, and arm needs to set the platform parameter to simulate x86.

```
docker build -f ./Dockerfile .
```

## Installation issues on Mac

### faiss installation

faiss can be installed via brew install faiss.

### fatal error: 'faiss/c\_api/AutoTune\_c.h' file not found

Execute the following command to solve:

source: [https://github.com/DataIntelligenceCrew/go-faiss/issues/7](https://github.com/DataIntelligenceCrew/go-faiss/issues/7)

```
cd faiss
export CMAKE_PREFIX_PATH=/opt/homebrew/opt/openblas:/opt/homebrew/opt/libomp:/opt/homebrew
cmake -B build -DFAISS_ENABLE_GPU=OFF -DFAISS_ENABLE_C_API=ON -DBUILD_SHARED_LIBS=ON -DFAISS_ENABLE_PYTHON=OFF .
make -C build
sudo make -C build install
sudo cp build/c_api/libfaiss_c.dylib /usr/local/lib/libfaiss_c.dylib
```
