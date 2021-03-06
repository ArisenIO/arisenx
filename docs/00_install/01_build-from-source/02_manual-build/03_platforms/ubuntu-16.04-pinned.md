---
content_title: Ubuntu 16.04 (pinned)
---

This section contains shell commands to manually download, build, install, test, and uninstall ARISEN and dependencies on Ubuntu 16.04.

[[info | Building ARISEN is for Advanced Developers]]
| If you are new to ARISEN, it is recommended that you install the [ARISEN Prebuilt Binaries](../../../00_install-prebuilt-binaries.md) instead of building from source.

Select a task below, then copy/paste the shell commands to a Unix terminal to execute:

* [Download ARISEN Repository](#download-arisen-repository)
* [Install ARISEN Dependencies](#install-arisen-dependencies)
* [Build ARISEN](#build-arisen)
* [Install ARISEN](#install-arisen)
* [Test ARISEN](#test-arisen)
* [Uninstall ARISEN](#uninstall-arisen)

[[info | Building ARISEN on another OS?]]
| Visit the [Build ARISEN from Source](../../index.md) section.

## Download ARISEN Repository
These commands set the ARISEN directories, install git, and clone the ARISEN repository.
```sh
# set ARISEN directories
export ARISEN_LOCATION=~/arisen/rsn
export ARISEN_INSTALL_LOCATION=$ARISEN_LOCATION/../install
mkdir -p $ARISEN_INSTALL_LOCATION
# install git
apt-get update && apt-get upgrade -y && DEBIAN_FRONTEND=noninteractive apt-get install -y git
# clone ARISEN repository
git clone https://github.com/arisenio/arisen.git $ARISEN_LOCATION
cd $ARISEN_LOCATION && git submodule update --init --recursive
```

## Install ARISEN Dependencies
These commands install the ARISEN software dependencies. Make sure to [Download the ARISEN Repository](#download-arisen-repository) first and set the ARISEN directories.
```sh
# install dependencies
apt-get install -y build-essential automake \
    libbz2-dev libssl-dev doxygen graphviz libgmp3-dev autotools-dev libicu-dev python2.7 python2.7-dev \
    python3 python3-dev autoconf libtool curl zlib1g-dev sudo ruby libusb-1.0-0-dev libcurl4-gnutls-dev \
    pkg-config apt-transport-https vim-common jq
# build cmake
export PATH=$ARISEN_INSTALL_LOCATION/bin:$PATH
cd $ARISEN_INSTALL_LOCATION && curl -LO https://cmake.org/files/v3.13/cmake-3.13.2.tar.gz && \
    tar -xzf cmake-3.13.2.tar.gz && \
    cd cmake-3.13.2 && \
    ./bootstrap --prefix=$ARISEN_INSTALL_LOCATION && \
    make -j$(nproc) && \
    make install && \
    rm -rf $ARISEN_INSTALL_LOCATION/cmake-3.13.2.tar.gz $ARISEN_INSTALL_LOCATION/cmake-3.13.2
# build clang
cd $ARISEN_INSTALL_LOCATION && git clone --single-branch --branch release_80 https://git.llvm.org/git/llvm.git clang8 && cd clang8 && git checkout 18e41dc && \
    cd tools && git clone --single-branch --branch release_80 https://git.llvm.org/git/lld.git && cd lld && git checkout d60a035 && \
    cd ../ && git clone --single-branch --branch release_80 https://git.llvm.org/git/polly.git && cd polly && git checkout 1bc06e5 && \
    cd ../ && git clone --single-branch --branch release_80 https://git.llvm.org/git/clang.git clang && cd clang && git checkout a03da8b && \
    cd tools && mkdir extra && cd extra && git clone --single-branch --branch release_80 https://git.llvm.org/git/clang-tools-extra.git && cd clang-tools-extra && git checkout 6b34834 && \
    cd $ARISEN_INSTALL_LOCATION/clang8/projects && git clone --single-branch --branch release_80 https://git.llvm.org/git/libcxx.git && cd libcxx && git checkout 1853712 && \
    cd ../ && git clone --single-branch --branch release_80 https://git.llvm.org/git/libcxxabi.git && cd libcxxabi && git checkout d7338a4 && \
    cd ../ && git clone --single-branch --branch release_80 https://git.llvm.org/git/libunwind.git && cd libunwind && git checkout 57f6739 && \
    cd ../ && git clone --single-branch --branch release_80 https://git.llvm.org/git/compiler-rt.git && cd compiler-rt && git checkout 5bc7979 && \
    mkdir $ARISEN_INSTALL_LOCATION/clang8/build && cd $ARISEN_INSTALL_LOCATION/clang8/build && \
    cmake -G 'Unix Makefiles' -DCMAKE_INSTALL_PREFIX=$ARISEN_INSTALL_LOCATION -DLLVM_BUILD_EXTERNAL_COMPILER_RT=ON -DLLVM_BUILD_LLVM_DYLIB=ON -DLLVM_ENABLE_LIBCXX=ON -DLLVM_ENABLE_RTTI=ON -DLLVM_INCLUDE_DOCS=OFF -DLLVM_OPTIMIZED_TABLEGEN=ON -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_BUILD_TYPE=Release .. && \
    make -j $(nproc) && \
    make install && \
    rm -rf $ARISEN_INSTALL_LOCATION/clang8
# build llvm
cd $ARISEN_INSTALL_LOCATION && git clone --depth 1 --single-branch --branch release_80 https://github.com/llvm-mirror/llvm.git llvm && \
    cd llvm && \
    mkdir build && cd build && \
    cmake -DLLVM_TARGETS_TO_BUILD=host -DLLVM_BUILD_TOOLS=false -DLLVM_ENABLE_RTTI=1 -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$ARISEN_INSTALL_LOCATION -DCMAKE_TOOLCHAIN_FILE=$ARISEN_LOCATION/scripts/pinned_toolchain.cmake -DCMAKE_EXE_LINKER_FLAGS=-pthread -DCMAKE_SHARED_LINKER_FLAGS=-pthread -DLLVM_ENABLE_PIC=NO .. && \
    make -j$(nproc) && \
    make install && \
    rm -rf $ARISEN_INSTALL_LOCATION/llvm
# build boost
cd $ARISEN_INSTALL_LOCATION && curl -LO https://dl.bintray.com/boostorg/release/1.71.0/source/boost_1_71_0.tar.bz2 && \
    tar -xjf boost_1_71_0.tar.bz2 && \
    cd boost_1_71_0 && \
    ./bootstrap.sh --with-toolset=clang --prefix=$ARISEN_INSTALL_LOCATION && \
    ./b2 toolset=clang cxxflags='-stdlib=libc++ -D__STRICT_ANSI__ -nostdinc++ -I$ARISEN_INSTALL_LOCATION/include/c++/v1 -D_FORTIFY_SOURCE=2 -fstack-protector-strong -fpie' linkflags='-stdlib=libc++ -pie' link=static threading=multi --with-iostreams --with-date_time --with-filesystem --with-system --with-program_options --with-chrono --with-test -q -j$(nproc) install && \
    rm -rf $ARISEN_INSTALL_LOCATION/boost_1_71_0.tar.bz2 $ARISEN_INSTALL_LOCATION/boost_1_71_0
# build mongodb
cd $ARISEN_INSTALL_LOCATION && curl -LO https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1604-3.6.3.tgz && \
    tar -xzf mongodb-linux-x86_64-ubuntu1604-3.6.3.tgz && rm -f mongodb-linux-x86_64-ubuntu1604-3.6.3.tgz && \
    mv $ARISEN_INSTALL_LOCATION/mongodb-linux-x86_64-ubuntu1604-3.6.3/bin/* $ARISEN_INSTALL_LOCATION/bin/ && \
    rm -rf $ARISEN_INSTALL_LOCATION/mongodb-linux-x86_64-ubuntu1604-3.6.3
# build mongodb c driver
cd $ARISEN_INSTALL_LOCATION && curl -LO https://github.com/mongodb/mongo-c-driver/releases/download/1.13.0/mongo-c-driver-1.13.0.tar.gz && \
    tar -xzf mongo-c-driver-1.13.0.tar.gz && cd mongo-c-driver-1.13.0 && \
    mkdir -p build && cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$ARISEN_INSTALL_LOCATION -DENABLE_BSON=ON -DENABLE_SSL=OPENSSL -DENABLE_AUTOMATIC_INIT_AND_CLEANUP=OFF -DENABLE_STATIC=ON -DENABLE_ICU=OFF -DENABLE_SNAPPY=OFF  -DCMAKE_TOOLCHAIN_FILE=$ARISEN_LOCATION/scripts/pinned_toolchain.cmake .. && \
    make -j$(nproc) && \
    make install && \
    rm -rf $ARISEN_INSTALL_LOCATION/mongo-c-driver-1.13.0.tar.gz $ARISEN_INSTALL_LOCATION/mongo-c-driver-1.13.0
# build mongodb cxx driver
cd $ARISEN_INSTALL_LOCATION && curl -L https://github.com/mongodb/mongo-cxx-driver/archive/r3.4.0.tar.gz -o mongo-cxx-driver-r3.4.0.tar.gz && \
    tar -xzf mongo-cxx-driver-r3.4.0.tar.gz && cd mongo-cxx-driver-r3.4.0 && \
    sed -i 's/\"maxAwaitTimeMS\", count/\"maxAwaitTimeMS\", static_cast<int64_t>(count)/' src/mongocxx/options/change_stream.cpp && \
    sed -i 's/add_subdirectory(test)//' src/mongocxx/CMakeLists.txt src/bsoncxx/CMakeLists.txt && \
    mkdir -p build && cd build && \
    cmake -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$ARISEN_INSTALL_LOCATION -DCMAKE_TOOLCHAIN_FILE=$ARISEN_LOCATION/scripts/pinned_toolchain.cmake .. && \
    make -j$(nproc) && \
    make install && \
    rm -rf $ARISEN_INSTALL_LOCATION/mongo-cxx-driver-r3.4.0.tar.gz $ARISEN_INSTALL_LOCATION/mongo-cxx-driver-r3.4.0
```

## Build ARISEN
These commands build the ARISEN software on the specified OS. Make sure to [Install ARISEN Dependencies](#install-arisen-dependencies) first.
```sh
export ARISEN_BUILD_LOCATION=$ARISEN_LOCATION/build
mkdir -p $ARISEN_BUILD_LOCATION
cd $ARISEN_BUILD_LOCATION && cmake -DCMAKE_BUILD_TYPE='Release' -DCMAKE_TOOLCHAIN_FILE=$ARISEN_LOCATION/scripts/pinned_toolchain.cmake -DCMAKE_INSTALL_PREFIX=$ARISEN_INSTALL_LOCATION -DBUILD_MONGO_DB_PLUGIN=true ..
cd $ARISEN_BUILD_LOCATION && make -j$(nproc)
```

## Install ARISEN
This command installs the ARISEN software on the specified OS. Make sure to [Build ARISEN](#build-arisen) first.
```sh
cd $ARISEN_BUILD_LOCATION && make install
```

## Test ARISEN
These commands validate the ARISEN software installation on the specified OS. This task is optional but recommended. Make sure to [Install ARISEN](#install-arisen) first.
```sh
$ARISEN_INSTALL_LOCATION/bin/mongod --fork --logpath $(pwd)/mongod.log --dbpath $(pwd)/mongodata
cd $ARISEN_BUILD_LOCATION && make test
```

## Uninstall ARISEN
These commands uninstall the ARISEN software from the specified OS.
```sh
xargs rm < $ARISEN_BUILD_LOCATION/install_manifest.txt
rm -rf $ARISEN_BUILD_LOCATION
```
