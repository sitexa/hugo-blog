+++
title = "eos学习与开发-01:cleos"
description = "eos学习与开发"
tags = [
    "BlockChain",
    "EOS"
]
date = "2018-07-24"
categories = [
    "Development",
    "Technology",
]
thumbnail = "images/eos/dpos_bft_opt.jpg"
+++

学习区块链技术，主要学好3条公链就足够了，就是比特币、以太坊、EOS，因为他们分别是区块链1.0、2.0、3.0的代表。

学习比特币，让我们知道区块链的技术原理；学习以太坊，让我们学会怎么运用智能合约和DAPP； 学习EOS，让我们把区块链应用到各行各业。

<!--more-->

##  智能合约编程语言：WebAssembly 

EOS使用c++作为智能合约编程语言，编译生成字节码(.wasm)运行在浏览器上，相对于JavaScript而言具有更高的性能。
它有一个名字叫WebAssembly。它看上去象一个编译器，让浏览器能看懂c++并运行c++。

![](/images/eos/terms/webassembly.png)


##  EOSIO Accounts and Wallets Conceptual Overview

![](/images/eos/terms/wallets.png)

Wallet钱包可以看成是用来保存密钥对的仓库，wallets是由keosd管理的，cleos是存取wallets的客户端工具。

Account帐户可以理解为链上标识符，配置存取权限就可以访问链上资源。nodeos管理帐户的发布及其在区块链上的行为。同样，使用客户端工具cleos存取帐户信息。

钱包和帐户之间没有继承关系，帐户不知道钱包的信息，反之也是。相应地，nodeos和keosd之间也没有继承关系。

##  创建和管理wallet

``` 
$ cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5HyE8aBw4xE4iVbUCX5qahGEvEXSzc9s7oqZ9CKLPJZzsVdffaL"
```

``` 
$ cleos wallet create -n periwinkle
Creating wallet: periwinkle
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JTA9a4Qj1wNsrbpPvKyLta4Ca32PBzAxdnWcboXkX2XDNU4tnd"
```

``` 
$ cleos wallet list
Wallets:
[
  "default *",
  "periwinkle *"
]
```

``` 
$ cleos wallet lock -n periwinkle
Locked: 'periwinkle'
```

``` 
$ cleos wallet list
Wallets:
[
  "default *",
  "periwinkle"
]
```

``` 
$cleos wallet open -n periwinkle
$cleos wallet open 
$cleos wallet list
Wallets:
[
  "default",
  "periwinkle"
]
```

``` 
$cleos wallet unlock --password PW5HyE8aBw4xE4iVbUCX5qahGEvEXSzc9s7oqZ9CKLPJZzsVdffaL
$cleos wallet unlock -n periwinkle --password PW5JTA9a4Qj1wNsrbpPvKyLta4Ca32PBzAxdnWcboXkX2XDNU4tnd
$cleos wallet list
Wallets:
[
  "default *",
  "periwinkle *"
]
```

##  keosd

-   1,默认端口：8900 （v1.1.1)

``` 
$cleos wallet list
"/usr/local/eosio/bin/keosd" launched
Wallets:
[]
$lsof -i -P | grep 'keosd'
keosd     15842 open    8u  IPv4 0xc15a02b884ca325      0t0  TCP localhost:8900 (LISTEN)
```

-   2,指定端口：8899

``` 
$keosd --http-server-address=localhost:8899
$lsof -i -P|
keosd     15906 open    8u  IPv4 0xc15a02b8892b5e5      0t0  TCP localhost:8899 (LISTEN)
```

-   3,在eosio-wallet/config.ini中配置端口

``` 
http-server-address=127.0.0.1:8899
```

-   4，在eosio的配置文件config.ini中配置钱包服务

``` 
http-server-address=127.0.0.1:8888
plugin=wallet_api_plugin
```

##  cleos连接钱包服务

-   1,连接默认端口:**keosd的默认端口是8900**

``` 
cleos wallet list
$ cleos wallet list
Wallets:
[]
```

-   2,连接指定端口

``` 
# keosd running
$cleos --wallet-url=http://127.0.0.1:8899 wallet list 
# nodeos running
$cleos --wallet-url=http://127.0.0.1:8888 wallet list
```

-   3,创建钱包(default)

``` 
$ cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JkDCnmazHPLBSciCrVYdsEsFuHDr2hkKQ24TaJ2emrXWBTpdtU"
```

-   4,创建和导入密钥

``` 
$ cleos create key
Private key: 5JHkeDNJiDUQ5k464TxNpKrkGEPrjMD7KgGdMcjxSmcW5Yxcoha
Public key: EOS8Z7ei9yXTcEmWKTw3tsD8KMKs9Pv4uvC7NjyuBpQfnyicbQcLu
$ cleos create key
Private key: 5JHv3BSQzefVVuEKG1QtNzX64XRiTzyyGNta5iRpYTEyorC9sWH
Public key: EOS6svKHUmqCbB2crGc3sWHiYp5Ly1PNB82AzerverVJryZ5kt1H8
```

``` 
$ cleos wallet import --private-key 5JHkeDNJiDUQ5k464TxNpKrkGEPrjMD7KgGdMcjxSmcW5Yxcoha
imported private key for: EOS8Z7ei9yXTcEmWKTw3tsD8KMKs9Pv4uvC7NjyuBpQfnyicbQcLu
```

``` 
$ cleos wallet import --private-key 5JHv3BSQzefVVuEKG1QtNzX64XRiTzyyGNta5iRpYTEyorC9sWH
imported private key for: EOS6svKHUmqCbB2crGc3sWHiYp5Ly1PNB82AzerverVJryZ5kt1H8
```

``` 
$ cleos wallet keys
[
  "EOS6svKHUmqCbB2crGc3sWHiYp5Ly1PNB82AzerverVJryZ5kt1H8",
  "EOS8Z7ei9yXTcEmWKTw3tsD8KMKs9Pv4uvC7NjyuBpQfnyicbQcLu"
]
```

``` 
$ cleos wallet private_keys --password PW5HyE8aBw4xE4iVbUCX5qahGEvEXSzc9s7oqZ9CKLPJZzsVdffaL
[[
    "EOS6svKHUmqCbB2crGc3sWHiYp5Ly1PNB82AzerverVJryZ5kt1H8",
    "5JHv3BSQzefVVuEKG1QtNzX64XRiTzyyGNta5iRpYTEyorC9sWH"
  ],[
    "EOS8Z7ei9yXTcEmWKTw3tsD8KMKs9Pv4uvC7NjyuBpQfnyicbQcLu",
    "5JHkeDNJiDUQ5k464TxNpKrkGEPrjMD7KgGdMcjxSmcW5Yxcoha"
  ]
]
```

``` 
$ cleos wallet private_keys -n periwinkle --password PW5JTA9a4Qj1wNsrbpPvKyLta4Ca32PBzAxdnWcboXkX2XDNU4tnd
[]
```

``` 
eosio-wallet$ ls
config.ini        default.wallet    periwinkle.wallet
```

##  创建帐户

在区块链上执行操作需要帐户account。我们使用cleos命令向nodeos发出请求创建帐户并把帐户发布在区块链上。

-   启动单节点区块链网络

``` 
nodeos -e -p eosio --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin
```
-   也可以通过config.ini配置启动参数

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

再启动```nodeos```.


-   导入eosio的密钥```5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3```

``` 
$ cleos --wallet-url=http://127.0.0.1:8888 wallet import --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
```

-   创建帐户```myaccount```

``` 
$ cleos --wallet-url=http://127.0.0.1:8888 create account eosio myaccount EOS6svKHUmqCbB2crGc3sWHiYp5Ly1PNB82AzerverVJryZ5kt1H8 EOS8Z7ei9yXTcEmWKTw3tsD8KMKs9Pv4uvC7NjyuBpQfnyicbQcLu
executed transaction: 276a75a95d52456f62f7724491f20e779887da72274a0a4f4dd02751be172356  200 bytes  7192 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"myaccount","owner":{"threshold":1,"keys":[{"key":"EOS6svKHUmqCbB2crGc3sWH...
```


##  关于钱包(Wallet)和帐户(Account)

帐户(account)是存储在区块链上的人类可读的标识符。每个帐户都有两个默认的命名权限：所有者和执行者。开发者可以引入新的命名权限。

钱包(wallet)是一种客户端软件，用来保存密钥，这些密钥可能会、也可能不会跟帐户的权限有关联。理想状态下，钱包有两种状态：锁定状态和解锁状态。

授权(authority):

-   ```owner```授权表示对帐户的所有权，拥户帐户的最高权限。
-   ```active```授权允许执行转帐、投票以及一些高水平帐户操作。

![](/images/eos/terms/accounts.png)

![](/images/eos/terms/authority.png)


