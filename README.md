![deployment](https://img.shields.io/github/deployments/WebCryptojacking/WebCryptojacking.github.io/github-pages?label=deployment) ![Platform](https://img.shields.io/badge/platform-Linux-important) ![license](https://img.shields.io/github/license/WebCryptojacking/WebCryptojacking.github.io) ![size](https://img.shields.io/github/repo-size/WebCryptojacking/WebCryptojacking.github.io)

## 0. Table of Contents

* [Introduction](#1-introduction)
* [Code](#2-code)
* [Data](#3-data)
* [Patches](#4-patches)

## 1. Introduction

Web cryptojacking exploits users’ web browsers to stealthily mine cryptocurrencies, which has become a lucrative business. In recent years, certain cryptocurrencies enforce the web-resistant virtual machine (WRVM) on the mining process. For example, Monero has enforced the RandomX VM while Scala has enforced the Panthera VM. These WRVMs make Web cryptojacking seemingly unprofitable.

However, in this work, we refute the over-optimism of the community and demonstrate that web cryptojacking is still profitable. We pinpoint the performance bottleneck of WRVMs over the web to be the vector instruction translation, and propose to transform and streamline vector instructions by investigating their element propagation processes. This is realized by our symbolic-constrained migratory emulation, which significantly enhances the instruction emulation efficiency.

We implement the whole design into an open-source web miner dubbed PropaMiner. This repository contains the source code of PropaMiner. We also release the data involved in this work and the patches to mitigate our proposed attack methodology.

**Note that all measurements, attacks, and analyses throughout this work are conducted under a well-established IRB, and no personally identifiable information is collected.**

## 2. Code

### Build from source

PropaMiner consists of two components including WebRandomX and WRXProxy.

WebRandomX is a high-performance web miner. It implements the key mechanisms of our symbolic-constrained migratory emulation for the RandomX VM. The core of WebRandomX is built into a WASM binary so that one can load it and perform web mining after the page is loaded from the remote server (can be a normal page or a C&C server). We provide an example web page to bootstrap the binary. 

WRXProxy is a mining pool proxy running on a remote server (i.e., the C&C server). It collects mining tasks from the mining pool (binding to a RandomX wallet address of the adversary), and distributes the tasks to active WebRandomX instances. When a WebRandomX instance finds a nonce that satisfies the difficulty criteria, it will submit it to WRXProxy and WRXProxy then proxies the nonce to the mining pool. In this way, one can gain profits from the submitted nonces.

#### Prerequisites

* **Cloud VM server** (for building and hosting PropaMiner): at least 1 GB memory and 2 Gbps bandwidth
* **OS**: Ubuntu 22.04.4 LTS
* **Emscripten emcc**: 3.1.5
* **npm**: 8.5.1
* **Node.js**: 12.22.9

#### Compile and deploy WebRandomX

Clone and build the WebRandomX binary:

```shell
git clone git@github.com:WebCryptojacking/WebRandomX.git
cd WebRandomX
mkdir build && cd build
emcmake cmake -DARCH=native ..
make
```

The built binary *web-randomx.wasm* will locate at *WebRandomX/build/web-randomx.wasm*

Afterwards, install npm dependencies to build the bootstrap page and run the web server:

```shell
npm install
npm run dev
```

The bootstrap page can be visited at *[Your Server IP]:9999*.

**Note**: The proxy server address should be configured in `src/js/job.js`.

#### Build and deploy WRXProxy

To enable web mining on WebRandomX page, WRXProxy should be built and deployed on the server:

```shell
git clone git@github.com:WebCryptojacking/WRXProxy.git
cd WRXProxy && npm install
```

The RandomX wallet should be configured in *config.json*:

```json
{
  "miner": {
    "port": 80
  },
  "pool": {
    "host": "gulf.moneroocean.stream",
    "port": 10001
  },
  "info": {
    "wallet": "# Monero Wallet Address",
    "password": "# Monero Wallet Password"
  }
}
```

Then, simply run `npm run start` to start the WRXProxy

The WebRandomX page should communicate with the proxy and mine automatically.

### Test with our pre-complied PropaMiner

We provide our pre-complied PropaMiner [here](http://66.42.105.235:9999/).

<img width="1452" alt="example-web-miner" src="https://github.com/WebCryptojacking/WebCryptojacking.github.io/assets/149248579/37ca7d46-7eb9-42fd-847c-31be0176a49d">

## 3. Data

We provide our measurement data regarding the tested WRVM programs at [WebCryptojacking.github.io/dataset](https://github.com/WebCryptojacking/WebCryptojacking.github.io/tree/main/dataset)

Each file contains the RandomX instructions in a WRVM program, and each line in a file is a specific instruction organized as:

```
<op code>, <dst reg>, <src reg>, <mod>, <imm>
```

## 4. Patches

We release our patches for mitigating the threat of web cryptojacking enabled by PropaMiner at [WebCryptojacking.github.io/patch](https://github.com/WebCryptojacking/WebCryptojacking.github.io/tree/main/patch). It implements resetting and randomized shuffling on vectors to minimize the isomorphism of element propagations.
