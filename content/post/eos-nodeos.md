+++
title = "eos学习与开发-02: nodeos配置"
description = "eos学习与开发"
tags = [
    "BlockChain",
    "EOS"
]
date = "2018-07-27"
categories = [
    "Development",
    "Technology",
]
thumbnail = "images/eos/dpos_bft_opt.jpg"
+++

nodeos可以通过cli选项配置，也可以通过配置文件config.ini配置。二者完全匹配，比如 --plugin eosio::chain_api_plugin vs. plugin=eosio::chain_api_plugin.

<!--more-->

##  单节点配置

启动单节点测试网络时，用cli选项配置：
```nodeos -e -p eosio --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin ```.

用config.ini配置：

``` 
    # Enable production on a stale chain, since a single-node test chain is pretty much always stale
    enable-stale-production = true
    # Enable block production with the testnet producers
    producer-name = eosio
    # Load the block producer plugin, so you can produce blocks
    plugin = eosio::producer_plugin
    # As well as API and HTTP plugins
    plugin = eosio::chain_api_plugin
    plugin = eosio::http_plugin
    # This will be used by the validation step below, to view history
    plugin = eosio::history_api_plugin
```

启动时删除已经存在的所有区块：```nodeos --delete-all-blocks```

设置最大交易时间：```max-transaction=3000```

config.ini的位置：

-   Mac OS: ~/Library/Application Support/eosio/nodeos/config
-   Linux: ~/.local/share/eosio/nodeos/config

###  生产节点

在config.ini中设置生产者的私钥用来签名区块。(下面的公钥及私钥代表了eosio帐户)

```
signature-provider = EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV=KEY:5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

###  非生产节点

在config.ini中设置p2p连接地址：```p2p-peer-address=106.10.42.238:9876```，可以设置多个地址。参考网页：
``` 
https://docs.google.com/spreadsheets/d/1K_un5Vak3eDh_b4Wdh43sOersuhs0A76HMCfeQplDOY/edit#gid=0
```

##  多节点配置

搭建一个单主机多节点的测试网络。如下图所示。

![](/images/eos/terms/nodeos.png)

### 1,启动keosd

``` 
$keosd  #端口：8900

$ cleos wallet keys
[
  "EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV", #eosio
  "EOS6svKHUmqCbB2crGc3sWHiYp5Ly1PNB82AzerverVJryZ5kt1H8", #key2
  "EOS7WRsrqfvT9WmD7FBXSGg8goaofp7ho8o8mQaHmsirbhgFB3MHp", #inita
  "EOS8Z7ei9yXTcEmWKTw3tsD8KMKs9Pv4uvC7NjyuBpQfnyicbQcLu"  #key1
]
```

### 2,创建default wallet

wallet password:```PW5HyE8aBw4xE4iVbUCX5qahGEvEXSzc9s7oqZ9CKLPJZzsVdffaL```

导入eosio的private-key: 
```
$cleos wallet import --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

### 3,启动第一个生产节点
```
$nodeos --enable-stale-production --producer-name eosio --plugin eosio::chain_api_plugin --plugin eosio::net_api_plugin
```

blocks目录中的内容：（文件大小不变）
``` 
$ ls -l nodeos/data/blocks/reversible
total 24
-rw-rw-r--  1 open  staff  356515840 Jul 28 18:13 shared_memory.bin
-rw-rw-r--  1 open  staff       3376 Jul 28 18:13 shared_memory.meta

```

state目录中的内容：(文件大小不变)
``` 
$ ls -l nodeos/data/state
total 10352
-rw-r--r--  1 open  staff        1313 Jul 28 18:13 forkdb.dat #nodeos运行时，没有这个文件。
-rw-rw-r--  1 open  staff  1073741824 Jul 28 18:13 shared_memory.bin
-rw-rw-r--  1 open  staff        3376 Jul 28 18:13 shared_memory.meta
```

或者使用配置文件config.ini: 

``` 
bnet-endpoint = 0.0.0.0:4321
bnet-follow-irreversible = 0
bnet-no-trx = false
bnet-peer-log-format = ["${_name}" ${_ip}:${_port}]

config-dir = "/Users/open/BlockChain/leos/gauss/config"
data-dir = "/Users/open/BlockChain/leos/gauss/data"

blocks-dir = "blocks"
chain-state-db-size-mb = 1024
reversible-blocks-db-size-mb = 340
contracts-console = false

https-client-validate-peers = 1
http-server-address = 127.0.0.1:8888

access-control-allow-credentials = false
max-body-size = 1048576
verbose-http-errors = false

p2p-listen-endpoint = 0.0.0.0:9876
p2p-max-nodes-per-host = 1
agent-name = "EOS Gauss Agent"

allowed-connection = any
max-clients = 25
connection-cleanup-period = 30
network-version-match = 0
sync-fetch-span = 100
max-implicit-request = 1500
use-socket-read-watermark = 0
peer-log-format = ["${_name}" ${_ip}:${_port}]

enable-stale-production = true
pause-on-startup = false
max-transaction-time = 30
max-irreversible-block-age = -1

producer-name = eosio 
signature-provider = EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV=KEY:5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
keosd-provider-timeout = 5
txn-reference-block-lag = 0

wallet-dir = "."
unlock-timeout = 900

plugin = eosio::chain_plugin
plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::http_plugin
plugin = eosio::history_api_plugin
plugin = eosio::wallet_api_plugin
```

```
$nodeos
```

### 4，启动第二个生产节点

-   加载eosio.bios合约
``` 
cleos set contract eosio build/contracts/eosio.bios
Reading WAST/WASM from build/contracts/eosio.bios/eosio.bios.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 1592225a963bc9f11d7b4aea80c0c591812be20834cc00328fddac6ab4a13f2b  3720 bytes  8584 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001621260037f7e7f0060057f7e7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":"0e656f73696f3a3a6162692f312e30050c6163636f756e745f6e616d65046e616d650f7065...
```

-   创建帐户inita

``` 
$cleos create key #for accnount inita
Private key: 5KPk7kd7ssSW6oTSP6UgmXfN2QjYyFUyRNEgif3yugKAkZt7idq
Public key: EOS7WRsrqfvT9WmD7FBXSGg8goaofp7ho8o8mQaHmsirbhgFB3MHp

$cleos wallet import --private-key 5KPk7kd7ssSW6oTSP6UgmXfN2QjYyFUyRNEgif3yugKAkZt7idq
imported private key for: EOS7WRsrqfvT9WmD7FBXSGg8goaofp7ho8o8mQaHmsirbhgFB3MHp

$cleos create account eosio inita EOS7WRsrqfvT9WmD7FBXSGg8goaofp7ho8o8mQaHmsirbhgFB3MHp EOS7WRsrqfvT9WmD7FBXSGg8goaofp7ho8o8mQaHmsirbhgFB3MHp
executed transaction: 1062496858d03f1de23323bb819d9a418dd49bfece42594b46f9ad1950bd80bd  200 bytes  1675 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"inita","owner":{"threshold":1,"keys":[{"key":"EOS7WRsrqfvT9WmD7FBXSGg8goa...

```

-   启动第二个nodeos节点：

``` 
$nodeos --producer-name inita --plugin eosio::chain_api_plugin --plugin eosio::net_api_plugin --http-server-address 127.0.0.1:8889 \
--p2p-listen-endpoint 127.0.0.1:9877 --p2p-peer-address 127.0.0.1:9876 --config-dir node2 --data-dir node2 --signature-provider \
EOS7WRsrqfvT9WmD7FBXSGg8goaofp7ho8o8mQaHmsirbhgFB3MHp=KEY:5KPk7kd7ssSW6oTSP6UgmXfN2QjYyFUyRNEgif3yugKAkZt7idq
```

此时，第二个节点并不是生产节点，它只同步第一个节点的块。

-   为了使第二个节点变成生产节点，必须在bios节点把帐户inita注册为生产者，并且更新bios节点的生产者排程。

``` 
$cleos push action eosio setprods "{ \"schedule\": [{\"producer_name\": \"inita\",\"block_signing_key\": \"EOS7WRsrqfvT9WmD7FBXSGg8goaofp7ho8o8mQaHmsirbhgFB3MHp\"}]}" -p eosio@active
executed transaction: 200723a709ab88d25be2916c8dbbb324b2ea23478632a92a687c374330321ee0  136 bytes  2333 us
#         eosio <= eosio::setprods              {"schedule":[{"producer_name":"inita","block_signing_key":"EOS7WRsrqfvT9WmD7FBXSGg8goaofp7ho8o8mQaHm...

```

这样，第二个节点成为了生产节点，而第一个节点只接受区块不生产区块。

-   查看节点信息

``` 
$cleos get info  # 第一个节点，默认端口#8888
{
  "server_version": "40a20769",
  "chain_id": "cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f",
  "head_block_num": 17821,
  "last_irreversible_block_num": 17820,
  "last_irreversible_block_id": "0000459cbb90ad07a47fcc68c21c3ed6fbaf492dde6c5c5d5649729d26fab786",
  "head_block_id": "0000459d42502a6e909470c6d511eb6c936e95a091105bfa684a1bae64b324f2",
  "head_block_time": "2018-07-27T12:06:58.000",
  "head_block_producer": "inita",
  "virtual_block_cpu_limit": 200000000,
  "virtual_block_net_limit": 1048576000,
  "block_cpu_limit": 199900,
  "block_net_limit": 1048576
}

$cleos --url http://127.0.0.1:8889 get info #第二个节点，指定端口#8889
{
  "server_version": "40a20769",
  "chain_id": "cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f",
  "head_block_num": 18062,
  "last_irreversible_block_num": 18061,
  "last_irreversible_block_id": "0000468d6913ac0bcf61c8bf6f2dea51d24d846df108a6a437e3a2e2859a4eeb",
  "head_block_id": "0000468ef039ffbfd9100b019bb0785bf70985e71203b3907a905512e560e3e2",
  "head_block_time": "2018-07-27T12:08:58.500",
  "head_block_producer": "inita",
  "virtual_block_cpu_limit": 200000000,
  "virtual_block_net_limit": 1048576000,
  "block_cpu_limit": 199900,
  "block_net_limit": 1048576
}
```

##  只读模式

每一个nodeos实例都在内存中维护了数据库，所以智能合约能够读写数据。nodeos节点通过提供http rpc api给客户，以读取这些数据。

有三种方式访问数据：

-   speculative:包含有未确认的交易；低延迟，不可靠；
-   head:仅包含最佳nodeos节点里的创建并签名了的区块；
-   irreversible:仅包含已经确认的交易；

##  总结

在mac上，eosio的root为：```~/Library/Application Support/eosio```。所以，在启动nodeos时，```--config```， ```--data-dir```要使用绝对路径，
否则，以eosio的root目录为相对目录。

我们将节点目录放在~/BlockChain/leos下，建两个节点：gauss,love。

``` 
$ tree leos -L 3
leos
├── gauss
│   ├── config
│   │   └── config.ini
│   └── data
│       ├── blocks
│       ├── default.wallet
│       └── state
└── love
    ├── config
    │   └── config.ini
    └── data
        ├── blocks
        └── state
```

-   1，启动keosd

默认钱包目录：```~/eosio-wallet```

-   2，启动第一个节点(生产节点)

``` 
$nodeos --config "/Users/open/BlockChain/leos/gauss/config/config.ini" --data-dir "/Users/open/BlockChain/leos/gauss/data"
```

-   3，启动第二个节点（非生产节点）

``` 
$nodeos --config "/Users/open/BlockChain/leos/love/config/config.ini" --data-dir "/Users/open/BlockChain/leos/love/data"
```

-   4，更改生产节点：将第二个节点改为生产节点。

``` 
$cleos push action eosio setprods "{ \"schedule\": [{\"producer_name\": \"inita\",\"block_signing_key\": \"EOS7WRsrqfvT9WmD7FBXSGg8goaofp7ho8o8mQaHmsirbhgFB3MHp\"}]}" -p eosio@active
Error 3070002: Runtime Error Processing WASM
```
**运行命令出错？？？**
