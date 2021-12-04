# Ubuntu18.04 编译安装 grpc

- 环境

  操作系统：Ubuntu18.04

  gRPC:v1.27.3

- 安装依赖

  ```bash
  apt-get install -y build-essential autoconf libtool pkg-config cmake libgflags-dev libgtest-dev clang-5.0 libc++-dev libssl-dev wget
  ```

- 克隆代码

  ```bash
  git clone -b v1.27.3 https://github.com/grpc/grpc
  cd grpc
  git submodule update --init --recursive
  ```

- Install absl

  ```bash
  mkdir -p "third_party/abseil-cpp/cmake/build"
  pushd "third_party/abseil-cpp/cmake/build"
  cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_POSITION_INDEPENDENT_CODE=TRUE ../..
  make -j4 install
  popd
  ```

- Install c-ares

  ```bash
  mkdir -p "third_party/cares/cares/cmake/build"
  pushd "third_party/cares/cares/cmake/build"
  cmake -DCMAKE_BUILD_TYPE=Release ../..
  make -j4 install
  popd
  ```

- Install protobuf

  ```bash
  mkdir -p "third_party/protobuf/cmake/build"
  pushd "third_party/protobuf/cmake/build"
  cmake -Dprotobuf_BUILD_TESTS=OFF -DCMAKE_BUILD_TYPE=Release ..
  make -j4 install
  popd
  ```

- Install zlib

  ```bash
  mkdir -p "third_party/zlib/cmake/build"
  pushd "third_party/zlib/cmake/build"
  cmake -DCMAKE_BUILD_TYPE=Release ../..
  make -j4 install
  popd
  ```

- Install gRPC

  ```bash
  mkdir -p "cmake/build"
  pushd "cmake/build"
  cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DgRPC_INSTALL=ON \
    -DgRPC_BUILD_TESTS=OFF \
    -DgRPC_CARES_PROVIDER=package \
    -DgRPC_ABSL_PROVIDER=package \
    -DgRPC_PROTOBUF_PROVIDER=package \
    -DgRPC_SSL_PROVIDER=package \
    -DgRPC_ZLIB_PROVIDER=package \
    ../..
  make -j4 install
  popd
  ```

- Build helloworld example using cmake

  ```bash
  mkdir -p "examples/cpp/helloworld/cmake/build"
  pushd "examples/cpp/helloworld/cmake/build"
  cmake ../..
  make
  popd
  ```
