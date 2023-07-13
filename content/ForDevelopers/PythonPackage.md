---
title: Python Package
---

Currently LingoDB is distributed as two seperate python packages: 
* `lingodb-bridge`: bundles LingoDB as a binary and implements a basic integration using pybind11
* `lingodb`: a python-only library that wraps `lingodb-bridge` and provides a nice interface (and much more in the future)

## Working on `lingo-db`
If you only plan to adapt/extend the python implementation, you do not have to build the `lingodb-bridge` package yourselve.
First install the current version of the `lingodb-bridge` package.
```sh
pip install lingodb-bridge
```
Then, install the package in *development mode* so that you can just change the code (`tools/python/lingodb`) and directly test the changes:
```sh
cd tools/python
python -m pip install -e .
```
For building a release package:
```sh
cd tools/python
python -m build .
```

## Building `lingodb-bridge`
Building a python binary wheel is non-trivial but becomes easy with our [docker image](https://github.com/orgs/lingo-db/packages/container/package/python-buildimg). Just execute the following commands at the repository's root:
```sh
docker pull ghcr.io/lingo-db/python-buildimg:latest
docker run --rm -v "$(pwd):/repo" -v "$(pwd):/built-packages" python-buildimg create_package.sh PYVERSION
```
where `PYVERSION` is one of:
* `cp39-cp39`: for Python 3.9
* `cp310-cp310`: for Python 3.11
* `cp311-cp311`: for Python 3.12

This will then create a wheel in the current directory that can be installe, e.g.:
```
pip install lingodb_bridge-0.0.0-cp310-cp310-manylinux_2_28_x86_64.whl
```

### Building the docker image
For building the docker image, just execute the following command at the repo's root directory.
```sh
docker build -t python-buildimg -f "tools/python/Dockerfile" .
```

