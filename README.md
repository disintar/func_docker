﻿# Dockerfile for FunC testing

---

This fork extend default build with node, so you can create your dapps gh action more easy

| image                                                                    | tag     | size  |
|--------------------------------------------------------------------------|---------|-------|
| [disintar/toncli-local](https://hub.docker.com/r/disintar/toncli-local/) | slim    | 69mb  |
| [disintar/toncli-local](https://hub.docker.com/r/disintar/toncli-local/) | m1-slim | 67mb  |

---

This docker image should come in handy to run [toncli](https://github.com/disintar/toncli) with
the [new tests support](https://github.com/disintar/toncli/blob/master/docs/advanced/func_tests_new.md).  
Setting it all up manually could be cumbersome otherwise.  
Inspired by [Dockerfile for the Open Network Node](https://github.com/ton-blockchain/ton/tree/master/docker).  
Built on Ubuntu 20.04 so should be WSL docker compatible.

## Pre-built images

Pre-built images availabe.
Mult-arch supported *x86_64 (amd64)* and *arm64/v8* **(M1 compatible!)**.  
[https://hub.docker.com/r/trinketer22/func_docker/](https://hub.docker.com/r/trinketer22/func_docker/)

- main is the image from master(this) branch.
- slim is extremely small image (< 50MB compressed) from slim branch.

### Install pre-built images

Run:`docker pull trinketer22/func_docker` to install main image.  
or  
Run `docker pull trinketer22/func_docker:slim` to install slim image.

Run `docker tag trinketer22/func_docker toncli-local` to give an alias `toncli-local` for the
image `trinketer22/func_docker`.

Then you may go straight to [Usage](#use).

## Build

To build an image run: `docker build . -t toncli-local [ optional --build-arg ]`  
Where *toncli-local* would be an image name.

In most cases that's it.
However, if you need something special, there are custom build arguments available.

### Custom build arguments

- **TON_GIT** specifies git repo url to fetch sources from. [SpyCheese](https://github.com/SpyCheese/ton) by default.
- **TON_BRANCH** specifies git branch to fetch from. **Set to toncli-local by default** so would likely require change
  if alternate *TON_GIT* is set.
- **BUILD_DEBUG** is self-explaintatory. By default *Release* binaries are built. Set *BUILD_DEBUG=1* to build debug
  binaries.
- **CUSTOM_CMAKE** Overrides build process cmake flags. Use it at your own risk.

Example of building debug binaries from [ton-blockchain/ton](https://github.com/ton-blockchain/ton) testnet branch

```console
docker build . -t toncli-local \
--build-arg TON_GIT=https://github.com/ton-blockchain/ton \
--build-arg TON_BRANCH=testnet \
--build-arg BUILD_DEBUG=1
```

## Use

You're going to need to pass your workdir as a volume to make things happen

### Creating project

Run

 ``` console
 docker run --rm -it \
 -v ~/Dev:/code \
 toncli-local start --name test_project wallet 
 ```

You're going to see the toncli project structure in *~/Dev/test_project*  
`README.md  build  fift  func  project.yaml  tests`

### Building

Run

  ``` console
  docker run --rm -it \
  -v ~/Dev/test_project:/code \
  toncli-local build
  ```

### Running tests

   ``` console
   docker run --rm -it \
   -v ~/Dev/test_project:/code \
   toncli-local run_tests
   ``` 

### Deploying contract

Now here is the tricky part.  
**Toncli** stores deployment info in it's config directory instead of your project directory.  
So we're going to have to create another volume for that to persist.

Run

  ``` console
  docker run --rm -it \
  -v ~/Dev/test_project:/code \
  -v /path/to/toncli_conf_dir/:/root/.config \
  toncli-local update_libs
  ```

After that you should go through standard toncli initialization dialog and pass absolute paths to the binaries

- /usr/local/bin/func
- /usr/local/bin/fift
- /usr/local/bin/lite-client

Don't get confused those path's are inside the docker image and not your local system.  
After that you should get an initialized toncli directory on your local system at */path/to/toncli_conf_dir/toncli*.  
Looking like:

  ``` console
  config.ini
  fift-libs
  func-libs
  test-libs
  ``` 

Now you can use it in the deploy or any other process like so.

Run

  ``` console
  docker run --rm -it \
  -v /path/to/project:/code \
  -v /path/to/toncli_conf_dir/:/root/.config \
  toncli-local deploy --net testnet
  ```

**wallet** directory would be created inside your local config dir with all the usefull deployment information

### General usage

 ``` console
 docker run --rm -it \
 -v <code_volume> \
 -v [optional config volume] \
 <docker image name> \
 <toncli command you want to run>
 ```
