# Measurement Kit CI scripts

This repository contains CI scripts including the docker file used to build
the [bassosimone/mk-debian](https://hub.docker.com/r/bassosimone/mk-debian)
docker images used by MK on Travis-CI.

Historically it only contained scripts related to docker builds, hence its
name and the directory where we recommend to add it as a submodule.

## How to integrate into a project

```
git submodule add https://github.com/measurement-kit/docker-ci .ci/docker
```

## How to run from Travis CI

You can perform most CMake builds you may want to run with this minimal
albeit complete `.travis.yml` file:

```yaml
language: c++
services:
  - docker
sudo: required
matrix:
  include:
    - env: BUILD_TYPE="asan"
    - env: BUILD_TYPE="clang"
    - env: BUILD_TYPE="coverage"
    - env: BUILD_TYPE="lsan"
    - env: BUILD_TYPE="ubsan"
    - env: BUILD_TYPE="vanilla"
script:
  - ./.ci/docker/script/travis $BUILD_TYPE
```

See [script/cmake](script/cmake) for more details.

## How to run from AppVeyor

This is a minimal `.appveyor.yml` for building for x86 and x86 using CMake:

```yaml
image: Visual Studio 2017
environment:
  matrix:
    - CMAKE_GENERATOR: "Visual Studio 15 2017 Win64"
    - CMAKE_GENERATOR: "Visual Studio 15 2017"
build_script:
  - cmd: git submodule update --init --recursive
  - cmd: .\.ci\docker\script\appveyor.bat
```

## How to generate a new docker image

```
docker build --no-cache debian
docker tag `docker images | head | awk '{print $3}'|sed -n 2p` \
    bassosimone/mk-debian:testing
docker push bassosimone/mk-debian
```
