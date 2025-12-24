# lwsf-wow

> Wownero fork of lwsf - Light Wallet Server Frontend

> This project is correctly incomplete, so the instructions here are for
> the truly adventurous.

## Table of Contents

  - [Introduction](#introduction)
  - [About this project](#about-this-project)
  - [License](#license)
  - [Compiling lwsf-wow from source](#compiling-lwsf-wow-from-source)
    - [Dependencies](#dependencies)
    - [Cloning the repository](#cloning-the-repository)
    - [Build Instructions](#build-instructions)
  - [Running lwsf-wow](#running-lwsf-wow)


## Introduction

Wownero is a privacy-focused, CPU-mineable, decentralized cryptocurrency based on Monero. This is the Wownero fork of lwsf (Light Wallet Server Frontend).


## About this project

The tl;dr is that you if you import this project into your wallet,
you can choose at runtime between a Wownero "full" wallet that does
all of the transaction scanning locally, or a Wownero "light" wallet
that does all of the scanning remotely. A single C++ `virtual`
interface has two different implementations for Wownero wallet
processing.

**The longer version -**

This is a client library for the [Wownero light-wallet REST API](https://codeberg.org/wownero/wownero-lws)
that implements the `wallet2_api.h` interface. Downstream projects/wallets can write code that calls into
the `wallet2_api.h` functions to manage a Wownero wallet, and select
a "standard" wownero wallet by calling
`Monero::WalletManagerFactory::getWalletManager()` or select a
light-wallet by calling `lwsf::WalletManagerFactory::getWalletManager()`.

The `lwsf` manager requires the user to specify a [wownero-lws](https://codeberg.org/wownero/wownero-lws) server,
whereas the `Monero` manager requires a wownerod RPC. The
light-wallet implementation can have quicker sync time, whereas the
wownerod implementation will have better privacy guarantees. If you
run your own LWS server, the privacy is identical.

The projects use different file formats for storing data, so
you have to manually import a wallet via seed if you want to
change backends.


## License

See [LICENSE](LICENSE).


## Compiling lwsf from source

### Dependencies

The following table summarizes the tools and libraries required to build.

| Dep          | Min. version  | Vendored | Debian/Ubuntu pkg    | Arch pkg     | Void pkg           | Fedora pkg          | Optional | Purpose         |
| ------------ | ------------- | -------- | -------------------- | ------------ | ------------------ | ------------------- | -------- | --------------- |
| GCC          | 7             | NO       | `build-essential`    | `base-devel` | `base-devel`       | `gcc`               | NO       |                 |
| CMake        | 3.5           | NO       | `cmake`              | `cmake`      | `cmake`            | `cmake`             | NO       |                 |
| Boost        | 1.66          | NO       | `libboost-all-dev`   | `boost`      | `boost-devel`      | `boost-devel`       | NO       | C++ libraries   |
| wownero      | master branch | NO       |                      |              |                    |                     | NO       | Wownero libs    |
| OpenSSL      | basically any | NO       | `libssl-dev`         | `openssl`    | `libressl-devel`   | `openssl-devel`     | NO       | sha256 sum      |
| libzmq       | 4.2.0         | NO       | `libzmq3-dev`        | `zeromq`     | `zeromq-devel`     | `zeromq-devel`      | NO       | ZeroMQ library  |
| libunbound   | 1.4.16        | NO       | `libunbound-dev`     | `unbound`    | `unbound-devel`    | `unbound-devel`     | NO       | DNS resolver    |
| libsodium    | ?             | NO       | `libsodium-dev`      | `libsodium`  | `libsodium-devel`  | `libsodium-devel`   | NO       | cryptography    |
| Doxygen      | any           | NO       | `doxygen`            | `doxygen`    | `doxygen`          | `doxygen`           | YES      | Documentation   |
| Graphviz     | any           | NO       | `graphviz`           | `graphviz`   | `graphviz`         | `graphviz`          | YES      | Documentation   |

Follow the guide from the [Wownero project](https://git.wownero.com/wownero/wownero) for
dependency installation and cloning the Wownero repository. Stop
after you've finished the cloning section for Wownero, as `lwsf-wow`
will build Wownero separately for you.

### Cloning the repository

Clone recursively to pull-in needed submodule(s):

```bash
git clone --recursive https://codeberg.org/wownero/lwsf-wow.git
cd lwsf-wow && git submodule init && git submodule update
```

### Build instructions

Wownero uses the CMake build system. Create a folder for the build,
and specify both the Wownero source directory and
lwsf-wow source directory:

```bash
mkdir build_lwsf && cd build_lwsf
cmake -DCMAKE_BUILD_TYPE=Release -DMONERO_SOURCE_DIR=/home/user/wownero_source /home/user/lwsf-wow_source
cmake --build .
```

## Running lwsf-wow

When complete, you should have `lwsf-ledger` executable in
`/home/user/build_lwsf/src` and a `lwsf-api.a` file in the
same folder. Run the `-h` option on `lwsf-ledger` to test
against a wownero-lws server.

### lwsf-api.a

This file is a little tricky to use outright as it doesn't
contain the entire code needed. The remainder of the code
is in `.a` files in the Wownero source directory. The best
way to include `lwsf-wow` into another project is to use Cmake
with the [`FetchContent` function](https://cmake.org/cmake/help/latest/module/FetchContent.html).
After running those few lines, you will have a `lwsf-api`
target in Cmake:

```cmake
project(your_project)
include(FetchContent)
FetchContent_Declare(lwsf SOURCE_DIR "${your_project_SOURCE_DIR}/external/lwsf-wow")

if (NOT lwsf_POPULATED)
  FetchContent_MakeAvailable(lwsf)
endif ()

add_library(your_library ...)
target_link_libraries(your_library lwsf-api)
```

## Syncing with Upstream

This project tracks the upstream [lwsf](https://github.com/vtnerd/lwsf) repository.
To sync changes from upstream:

```bash
git fetch upstream
git merge upstream/main
# Resolve any conflicts if needed
git push origin main
```



