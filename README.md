# Measurement Kit common CI scripts

This repository contains common CI scripts used by Measurement Kit. On travis
CI (and possibly on any other Docker enabled CI system) we use the custom
[bassosimone/mk-debian](https://hub.docker.com/r/bassosimone/mk-debian)
docker image to run tests. This allows us to reproduce Travis builds locally.

## How to integrate into a project

```
git submodule add https://github.com/measurement-kit/ci-common .ci/common
```

## How to run from Travis CI

You can perform most builds you may want to run with this minimal
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
  - ./.ci/common/script/travis $BUILD_TYPE
```

The [script/travis](script/travis) will automatically trigger an `autotools`
or a `cmake` build depending on the content of the root directory. To see what
triggers which build, we encourage you to read such script.

You can locally run a specific Travis build, e.g. "vanilla", with:

```
./.ci/common/script/travis vanilla
```

Of course, this requires you to have the docker daemon running.

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
  - cmd: cmake -G "%CMAKE_GENERATOR%"
  - cmd: cmake --build . -- /nologo /property:Configuration=Release
  - cmd: ctest --output-on-failure -C Release -a
```

## How to generate a new docker image

```
docker build --no-cache debian
docker tag `docker images | head | awk '{print $3}'|sed -n 2p` \
    bassosimone/mk-debian:testing
docker push bassosimone/mk-debian
```
