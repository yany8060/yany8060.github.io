---
title: go-ethereum-简单搭建私有链
date: 2017-12-05 20:34:24
tags: [以太坊,区块链]
categories: [区块链]
---

### Geth
> https://ethereum.github.io/go-ethereum/install/

#### 安装 geth：
访问[https://geth.ethereum.org/downloads/](https://geth.ethereum.org/downloads/)，下载Geth for macOS。
![F674D81F-4D30-4C90-9D2C-5888F42F688C.png](http://upload-images.jianshu.io/upload_images/1419542-1cd247330ce95842.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
给geth做一个软连接到/usr/local/bin目录下，然后在命令行输入：geth version  显示如下边上安装成功
![image.png](http://upload-images.jianshu.io/upload_images/1419542-d6d5d014aeb2a3bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)
使用homebrew安装：
```
brew tap ethereum/ethereum
brew install ethereum
```

#### Geth工具介绍：
* Geth工具是Go Ethereum, 是以太坊的官方客户端（Go语言实现）
* Geth的命令行中包含了大多数的以太坊的命令，包括账户新建，账户之间的以太币转移，挖矿，获取余额，部署以太坊合约等

### 配置私链节点
新建文件夹，命名随意，在此文件夹下创建genesis.json文件和data文件夹
genesis.json内容如下
```
{
  "config": {
        "chainId": 0,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x20000",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```
![创世块.png](http://upload-images.jianshu.io/upload_images/1419542-6403c3c435533892.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 初始化：
在命令行下进入刚才创建的文件夹，输入如下命令：
`geth --datadir ./data/00 init genesis.json`
各参数代表的含义如下：
* init 表示初始化区块，后面跟着创世块的配置文件genesis.json
* datadir 数据存放的位置
```
privatechain
├── data
│   ├── geth
│   │   └── chaindata
│   │       ├── 000002.log
│   │       ├── CURRENT
│   │       ├── LOCK
│   │       ├── LOG
│   │       └── MANIFEST-000003
│   └── keystore
└── genesis.json
```


#### 启动节点：
`geth --datadir ./data/00 --networkid 15 console`
各参数代表的含义如下：
* networkid 设置当前区块链的网络ID，用于区分不同的网络，1表示公链
* console 表示启动命令行模式，可以在Geth中执行命令
![启动节点.png](http://upload-images.jianshu.io/upload_images/1419542-c1ac395ce5bdd206.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 控制台
* 创建账号：`personal.newAccount()`或者 `personal.newAccount("123456")`
* 查看节点信息:`admin.nodeInfo`
* 挖矿
  *  开始挖矿 `miner.start()`
  * 停止挖矿 `miner.stop()`
* 查看账号：`eth.accounts`
* 查看账号余额：`eth.getBalance("账号名称（eth.accounts[0]）")`

