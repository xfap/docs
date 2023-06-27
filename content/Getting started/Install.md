---
title: Installation
type: docs
weight: 1
---

## Python Package

## Docker Image

## Building from source

### Building Apache Arrow from Source
```sh
cd arrow
mkdir build
cmake cpp  -B build -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_LIBDIR=lib -DARROW_CSV=ON -DARROW_COMPUTE=ON
cmake --build build
sudo cmake --install build
```

