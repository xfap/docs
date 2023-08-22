---
title: Installation
type: docs
weight: 1
---

## Python Package
Install via pip, then use as [documented here]({{< ref "Python.md" >}} )
```
pip install lingodb
```

## Docker Image
Either use the 
* [prebuilt docker image](https://github.com/lingo-db/lingo-db/pkgs/container/lingo-db)
* or build the docker image yourself using `make build-docker`

The docker image then contains all the command line tools under `/build/lingodb/`

## Building from source
1. Ensure you have a machine with sufficient compute power and space (16 GiB RAM, >16 GiB disk space, preferably many cores).
1. Initialize submodules: `git submodule update --init --recursive`
1. Install Dependencies
    1. oneTBB: [Build from source](https://github.com/oneapi-src/oneTBB/blob/master/INSTALL.md) or get from [Intel](https://www.intel.com/content/www/us/en/developer/articles/tool/oneapi-standalone-components.html#onetbb)
    1. Apache Arrow 12.0.1 : either install [prebuilt C++ packages](https://arrow.apache.org/install/) or build it from source (see below)
    1. standard build tools, including `cmake` and `Ninja`
1. Build LLVM/MLIR: `make build_llvm`
1. Build LingoDB
    * Debug Version : `make build-debug` (will create binaries under `build/lingodb-debug`)
    * Release Version : `make build-release` (will create binaries under `build/lingodb-release`)

### Building Apache Arrow from Source
```sh
cd arrow
mkdir build
cmake cpp  -B build -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_LIBDIR=lib -DARROW_CSV=ON -DARROW_COMPUTE=ON
cmake --build build
sudo cmake --install build
```

