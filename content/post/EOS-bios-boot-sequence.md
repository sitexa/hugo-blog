+++
title = "eos bios启动过程"
description = "eos bios boot sequence"
tags = [
    "BlockChain",
    "EOS",
    "bios"
]
date = "2018-06-27"
categories = [
    "Development",
    "Technology",
]
thumbnail = "images/eos/dpos_bft_opt.jpg"
+++

本文介绍从单个"创世"节点开始，直到对多个"生产"节点投票的EOS区块链的启动顺序全过程。

<!--more-->

原文：https://developers.eos.io/eosio-nodeos/docs/bios-boot-sequence

脚本程序```bios-boot-tutorial.py```实现了单服务器上启动多节点的过程，所有的nodeos节点
都运行在同一台服务器上。你可以在目录中直接运行该脚本程序。也可以通过指定```-a``` 或 ```--all```参数，
运行脚本程序。当执行完所有的步骤，有了一些生产节点之后，你可以通过执行脚本，使用```eosio.msig```
合约替换系统里的合约，然后在帐户之间随机转移token。

``` 
cd tutorials/bios-boot-tutorial
./bios-boot-tutorial.py -a    # to execute all options except replace system and transfer tokens, use -h for help
```

脚本执行时将把运行日志写在```./output.log```文件里，检查该日志文件看看程序做了什么工作。你也会看到一个```./nodeos```
文件夹，包含了很多子文件夹，每个子文件夹包含一个运行节点，以及该节点运行时的日志文件。

**注：目前，参数```-a```会导致脚本运行失败，因为帐户购买RAM已经结束。要解决这个问题，使用参数```--extra-issue 10000```
或者一个合适的数字，分派多于100亿的token。(经测试，nodeos 不认识 ```--extra-issue 10000```参数。去掉该参数能启动成功。)**

##  手工运行启动过程

下面介绍通过人工执行每一个命令的整个启动过程，完成与脚本```bios-boot-tutorial.py```一样的步骤，以便对启动过程有一个直观的了解。
但是，人工过程不会启动一个跟脚本程序一样规模的区块链，比如脚本程序默认创建了3000个帐户，以及30+个节点，人工启动就太笨拙了。通过
这个介绍，结合脚本程序，可以加深对启动过程的理解。

**在一个真实的区块链网络中，每一步都需要在独立的参与者中间进行大量的协调工作。区块链社区的一个小团体可能会密切配合，以启动并运行该
初始化区块链。该网络将会扩展到与所有参与者合作。前期步骤将会加载一批帐户、密钥、以及每个帐户的token余额。这些步骤将会持续地扩展到
网络上。这里提到了参与各方如何协调的假设。但是，社区可以选择多种方式进行协调。该过程的技术方面是客观的; 关于协调可能如何发生的假设
是推测性的。 社区已经提出了几种方法。 鼓励您审查各种方法并酌情参与讨论。**

### 配置nodeos节点的初始集合

在本教程中，我们将启动一些nodeos节点，将它们指向对方，并最终对一组生产者进行投票。 所有nodeos节点都将运行在同一台服务器上。
在接下来的部分中，我们会采取各种步骤来准备我们的候选生产者。 我们将使用命名约定accountnumXY，并从数字1-5中选择XY，例如，

    accountnum11
    accountnum12
    ..
    accountnum15
    accountnum21
    ...
    accountnum55
    
**上面说过，脚本实现了这些步骤，但是它使用了不同的（以及更多）数据值。 请参阅文件accounts.json，了解脚本使用的生产节点名称和用户帐户名称。**

### 为每个nodeos节点创建配置和数据目录

由于所有nodeos节点都将运行在同一台服务器上，因此我们需要为每个节点分别配置和设置数据目录。 以下示例为```〜/eosio_test```目录下的每个nodeos创建目录。 
使用任何最适合的方法创建多个帐户，请记住名称限制，即帐户名称必须完全是12个字符，是a-z和1-5的组合。

``` 
$ mkdir ~/eosio_test
$ for (( i = 1; i <= 5; i++ )); do for (( j = 1 ; j <=5 ; j++ )); do mkdir ~/eosio_test/accountnum$i$j; done; done
```

您将会在nodeos命令行上将这些目录与```--config-dir```和```--data-dir```参数一起使用。

### 准备对等通信的IP地址

当我们建立生产节点时，我们要配置它们指向彼此用以执行对等通信。 如果设置全网状配置，则每个节点将指向所有其他节点。 全网状网络将迅速变得笨拙。 
以下描述说明如何将节点指向其他节点以创建点对点通信。 使用它可以将您选择的节点作为对等节点，但建议不要将其用于所有节点，因为您的配置将以爆炸式增长。

确定每个nodeos节点之间的点对点通信的IP地址和端口号（您将在实际启动生产节点时稍后使用它）。 每个nodeos都可以通过设置p2p-peer-address配置属性进行配置，
可以在启动nodeos（每个对等节点一个参数）时在命令行上进行配置，也可以通过在config.ini文件中为nodeos设置属性（每个对等节点一行）。

例如，假设我们为生产节点使用端口号9011-9055（即分别为accountnum11 - accountnum55），请在accountnum12的nodeos命令行上包含以下参数：

``` 
--p2p-peer-address localhost:9011 --p2p-peer-address localhost:9013 --p2p-peer-address localhost:9014 ...
```

或者，在accountnum12的config.ini文件中，为每个对等节点添加行：

``` 
p2p-peer-address = localhost:9011
p2p-peer-address = localhost:9013
p2p-peer-address = localhost:9014
...
```

如果使用配置文件方法，您将拥有config.ini文件的多个副本，每个〜/eosio_test/accountnumXY目录中都有一个副本。 
无论使用命令行还是配置文件方法，请注意不要在对等列表中包含生产节点自己的地址。

### 启动“创世”节点

“创世”节点是我们开始的第一个nodeos节点，它将启动区块链。 所有其他节点将从创蕊节点派生出来。 在创世节点上执行以下操作。

**注意：在真实场景中，所有IP地址和端口号在bios启动过程中不过能是已知的。 例如，最早启动的节点不知道后来启动的节点的IP地址和端口。 这些信息可以逐步添加。**

### 创建钱包

创建钱包。 默认情况下，keosd会自动启动以管理钱包。

``` 
$ cleos wallet create
```

请务必保存钱包密码，以便将来可以解锁钱包。

无论正在访问哪个节点，本教程中所有帐户的所有密钥管理都将使用相同的钱包。 在分布式部署中，钱包管理应该是仅限本地的活动。

### 配置```genesis.json```文件

```genesis.json```文件定义了初始链状态。 所有节点必须从相同的初始状态开始。 这里着重介绍两个特性。 ```initial_timestamp```表示区块链的开始时间。 
```initial_key```被创始节点用来开始生成区块。 它需要在所有节点中相同。 ```genesis.json```中的```initial_key```属性必须与生成节点的公钥匹配。 
可以使用nodeos命令行上的```--private-key```参数或通过在```config.ini```文件中设置```private-key```属性来指定nodeos密钥对。

脚本使用的```genesis.json```文件可以在```tutorial/bios-boot-tutorial```目录中找到。 所有nodeos实例都将使用此文件启动。 该脚本在命令行中指定nodeos密钥对。 
该脚本允许您为生产节点（```--private-key```和```--public-key```选项）指定密钥对。 如果指定了创世节点的密钥，请确保对您的```genesis.json```文件进行必要的更改。 
该脚本将使用该密钥对开始创建节点，并使用该密钥对创建```eosio.*```帐户。

### 为eosio帐户创建密钥

nodeos预先配置了一个密钥对，但我们不想使用该密钥。 创建用于创世节点的eosio帐户的密钥对。 教程脚本预先配置了自己的自定义密钥对。

``` 
$ cleos create key
Private key: 5JGxnezvp3N4V1NxBo8LPBvCrdR85bZqZUFvBZ8ACrbRC3ZWNYv
Public key: EOS8VJybqtm41PMmXL1QUUDSfCrs9umYN4U1ZNa34JhPZ9mU5r2Cm
```

### 启动nodeos创世节点

使用生成的密钥对启动创世节点。

``` 
$ nodeos -e -p eosio --private-key '[ "EOS8VJybqtm41PMmXL1QUUDSfCrs9umYN4U1ZNa34JhPZ9mU5r2Cm","5JGxnezvp3N4V1NxBo8LPBvCrdR85bZqZUFvBZ8ACrbRC3ZWNYv" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --plugin eosio::history_api_plugin
```

### 创建重要的系统帐户

以下几个系统帐户是必须的：

``` 
  eosio.bpay
  eosio.msig
  eosio.names
  eosio.ram
  eosio.ramfee
  eosio.saving
  eosio.stake
  eosio.token
  eosio.vpay
```

重复以下步骤为每个系统帐户创建一个帐户。 在本教程中，我们将针对帐户所有者(```owner```)和主动者(```active```)使用相同的密钥对，因此我们只需在命令行上提供一次密钥值。 
对于大多数一般账户，最好使用单独的密钥给所有者(```owner```)和主动账户(```active```)。 该脚本为所有```eosio.*```帐户使用相同的密钥。 您可以为每个帐户使用不同的密钥。

``` 
$ cleos create key  # for eosio.bpay
Private key: 5KAVVPzPZnbAx8dHz6UWVPFDVFtU1P5ncUzwHGQFuTxnEbdHJL4
Public key: EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG

$ cleos wallet import 5KAVVPzPZnbAx8dHz6UWVPFDVFtU1P5ncUzwHGQFuTxnEbdHJL4
imported private key for: EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG

$ cleos create account eosio eosio.bpay EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
executed transaction: ca68bb3e931898cdd3c72d6efe373ce26e6845fc486b42bc5d185643ea7a90b1  200 bytes  280 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.bpay","owner":{"threshold":1,"keys":[{"key":"EOS84BLRbGbFahNJEpnnJH...
```

### 部署```eosio.token```合约

部署```eosio.token```合约。 本合同允许您创建，发布，传输和获取有关token的信息。 请注意，该示例假定您在```〜/Documents/eos```文件夹中构建了eos。

```
$ cleos set contract eosio.token ~/Documents/eos/build/contracts/eosio.token
Reading WAST/WASM from /Users/tutorial/Documents/eos/build/contracts/eosio.token/eosio.token.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 17fa4e06ed0b2f52cadae2cd61dee8fb3d89d3e46d5b133333816a04d23ba991  8024 bytes  974 us
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d01000000017f1560037f7e7f0060057f7e...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...
```

### 部署```eosio.msig```合约

设置```eosio.msig```合同。 msig合同支持并简化了权限级别的定义和管理以及执行多重签名操作。 请注意，这假定您在```〜/Documents/eos```文件夹中构建了EOSIO。

``` 
$ cleos set contract eosio.msig ~/Documents/eos/build/contracts/eosio.msig
Reading WAST/WASM from /Users/tutorial/Documents/eos/build/contracts/eosio.msig/eosio.msig.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 007507ad01de884377009d7dcf409bc41634e38da2feb6a117ceced8554a75bc  8840 bytes  925 us
#         eosio <= eosio::setcode               {"account":"eosio.msig","vmtype":0,"vmversion":0,"code":"0061736d010000000198011760017f0060047f7e7e7...
#         eosio <= eosio::setabi                {"account":"eosio.msig","abi":{"types":[{"new_type_name":"account_name","type":"name"}],"structs":[{...
```

### 创建并分配SYS货币

创建最大值为100亿token的SYS货币,然后发行10亿token。将SYS替换为特定的货币名称。

``` 
$ cleos push action eosio.token create '[ "eosio", "10000000000.0000 SYS" ]' -p eosio.token
executed transaction: 0440461e0d8816b4a8fd9d47c1a6a53536d3c7af54abf53eace884f008429697  120 bytes  326 us
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"10000000000.0000 SYS"}

$ cleos push action eosio.token issue '[ "eosio", "1000000000.0000 SYS", "memo" ]' -p eosio
executed transaction: a53961a566c1faa95531efb422cd952611b17d728edac833c9a55582425f98ed  128 bytes  432 us
#   eosio.token <= eosio.token::issue           {"to":"eosio","quantity":"1000000000.0000 SYS","memo":"memo"}
```

在第一步中，由```eosio.token```帐户授权的```eosio.token```合约中的创建操作在eosio帐户中创建100亿```SYS```token。 该命令创建了token的最大供应量，但不会将任何token放入流通环境中。 没有流通的token可以被视为保留token。

在第二步中，```eosio.token```合约的派发操作将10亿```SYS```token从储备中取出并进行流通。 在派发时，token在eosio账户中。 由于eosio账户拥有未流通token的储备，因此需要其权限来执行该操作。

作为一个利益点，从经济角度来看，将token从储备转移到流通（例如通过发放token）是一种通货膨胀行为。 发行token只是通胀发生的一种方式。

### 部署```eosio.system```合约

该合同为几乎所有基于token的操作提供了操作。 在安装系统合同之前，操作是独立于会计完成的。 一旦系统合约启用，任何基于token的行为都是一个经济行为。 使用资源（cpu，网络，内存）必须付费。 
同样，必须为新建帐户付费。 系统合约使得token能够被抵押和解押，资源需要购买，可能的生产节点被注册并且随后被投票，生产节点申请奖励，设置特权和限制等等。

``` 
$ cleos set contract eosio ~/Documents/eos/build/contracts/eosio.system
Reading WAST/WASM from /Users/tutorial/Documents/eos/build/contracts/eosio.system/eosio.system.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 2150ed87e4564cd3fe98ccdea841dc9ff67351f9315b6384084e8572a35887cc  39968 bytes  4395 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001be023060027f7e0060067f7e7e7f7f...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"buyrambytes","base":"","fields":[{"name":"p...

```

### 从单一生产节点转变为多个生产节点

在下一组步骤中，我们将从单个块生产节点（创始节点）过渡到多个块生产节点。 到目前为止，只有内置的eosio帐户已被授权，并且可以签名块（生产区块）。 
目标是通过一系列按照```2/3+1```规则运作的当选块生产节点来管理区块链。

块生产节点是通过选举选出的。 块生产节点名单可以改变。 管理规则不是直接向任何块生产节点授予特权，而是与名为```eosio.prods```的特殊内置帐户相关联。 
该帐户代表当选的块生产节点组。 ```eosio.prods```帐户（实际上是块生产节点组）使用由```eosio.msig```合约定义的权限进行操作。

在安装```eosio.system```合约后，我们希望尽快将```eosio.msig```设置为特权帐户，以便它可以代表```eosio```帐户进行授权。 一旦```eosio```释放特权，```eosio.prods```帐户将接管该特权。

### 使```eosio.msig```成为一个特权帐户

通过下面的命令将```eosio.msig```变成一个特权帐户。

``` 
$ cleos push action eosio setpriv '["eosio.msig", 1]' -p eosio@active
```

### 抵押token并放大网络

如果已按照上述教程步骤操作完成，则现在您已经在单个主机上安装了单个节点，配置了以下合约：

-   eosio.token
-   eosio.msig
-   eosio.system

帐户```eosio```和```eosio.msig```是特权帐户。 其他```eosio.*```帐户已创建但没有特权。

我们现在已经准备好抵押账户和扩大生产节点网络。

### 创建抵押帐户

抵押是将某人（例如，以众筹或其他方式购买东西的个人）在“现实世界”中获取的token分配给EOSIO系统内的帐户的过程。 
区块链的整个生命周期是一个持续的过程。 在bios启动过程中进行的初始抵押非常特殊。 在bios启动过程中，账户与其token放在一起。
但是，在生产节点当选之前，token实际上处于冻结状态。 因此，在BIOS启动过程中完成初始抵押的目的是获取分配给其帐户并准备好使用的token，
并让投票过程继续进行，以便生产节点可以当选并且区块链“正常”运行。

以下建议是针对初始抵押过程给出的：

-   0.1token（不是账户token的10％）被抵押以购买RAM。 默认情况下，cleos在帐户创建时支付8KB的RAM费，由帐户创建者支付。 
在最初的抵押中，eosio账户是做抵押的账户创建者。 在初始token抵押过程中抵押的token不能解押，并且在达到最低投票要求之前不能流通。
-   0.45token用于CPU，0.45token用于网络带宽。
-   最多9个token可以参与流通。
-   剩余的token按50/50的比例抵押为CPU和网络费用。
    -   例子1. accountnum11有100个SYS。 它将在RAM上抵押0.1000SYS; 
        CPU上45.4500SYS; 45.4500SYS网络; 和9.0000SYS供流通使用。
    -   例子2. accountnum33有5个SYS。 它将在RAM上抵押0.1000 SYS; 
        CPU上0.4500SYS; 网络上0.4500SYS; 和4.0000个系统用于流通。

为了使教程更真实，我们使用帕累托分布将10亿token分配给帐户。 帕累托分布模拟80-20规则，例如，80％的token被20％的人口占有。 
这里的例子并没有显示如何生成分布，而是集中在命令上进行实现。 本教程附带的脚本```bios-boot-tutorial.py```使用```Python NumPy（numpy）```库生成帕累托分布。

### 创建一个抵押帐户

使用以下步骤为每个帐户抵押token。 这些步骤必须针对每个帐户单独完成。

**密钥对在这里是为本教程创建的。 在“现实”场景中，帐户的密钥和token份额应该通过一些明确定义的外部过程来建立。**

``` 
$ cleos create key  # for accountnum11
Private key: 5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o
Public key: EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt

$ cleos wallet import 5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o
imported private key for: EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt
```

使用初始资源和公钥创建一个抵押帐户。

``` 
$ cleos system newaccount eosio --transfer accountnum11 EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt --stake-net "100000.0000 SYS" --stake-cpu "100000.0000 SYS"
775292ms thread-0   main.cpp:419                  create_action        ] result: {"binargs":"0000000000ea30551082d4334f4d113200200000"} arg: {"code":"eosio","action":"buyrambytes","args":{"payer":"eosio","receiver":"accountnum11","bytes":8192}} 
775295ms thread-0   main.cpp:419                  create_action        ] result: {"binargs":"0000000000ea30551082d4334f4d113200ca9a3b00000000045359530000000000ca9a3b00000000045359530000000001"} arg: {"code":"eosio","action":"delegatebw","args":{"from":"eosio","receiver":"accountnum11","stake_net_quantity":"100000.0000 SYS","stake_cpu_quantity":"100000.0000 SYS","transfer":true}} 
executed transaction: fb47254c316e736a26873cce1290cdafff07718f04335ea4faa4cb2e58c9982a  336 bytes  1799 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"accountnum11","owner":{"threshold":1,"keys":[{"key":"EOS8mUftJXepGzdQ2TaC...
#         eosio <= eosio::buyrambytes           {"payer":"eosio","receiver":"accountnum11","bytes":8192}
#         eosio <= eosio::delegatebw            {"from":"eosio","receiver":"accountnum11","stake_net_quantity":"100000.0000 SYS","stake_cpu_quantity...
```

### 选择生产节点

一些账户将被注册为生产节点。 选择一些抵押账户作为生产节点。

####    注册为生产节点

使用以下命令注册为生产节点。 这使得该节点成为一个生产节点，但节点实际上不会是一个生产节点，除非它被选中。

``` 
$ cleos system regproducer accountnum11 EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt https://accountnum11.com/EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt
1487984ms thread-0   main.cpp:419                  create_action        ] result: {"binargs":"1082d4334f4d11320003fedd01e019c7e91cb07c724c614bbf644a36eff83a861b36723f29ec81dc9bdb4e68747470733a2f2f6163636f756e746e756d31312e636f6d2f454f53386d5566744a586570477a64513254614364754e7553504166584a48663232756578347534316162314556763945416857740000"} arg: {"code":"eosio","action":"regproducer","args":{"producer":"accountnum11","producer_key":"EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","url":"https://accountnum11.com/EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","location":0}} 
executed transaction: 4ebe9258bdf1d9ac8ad3821f6fcdc730823810a345c18509ac41f7ef9b278e0c  216 bytes  896 us
#         eosio <= eosio::regproducer           {"producer":"accountnum11","producer_key":"EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","u...
```

####    列表显示生产节点

为了方便投票过程，列出可用的生产节点。

``` 
$ cleos system listproducers
Producer      Producer key                                           Url                                                         Scaled votes
accountnum11  EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt  https://accountnum11.com/EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22 0.0000
accountnum22  EOS5kgeCLuQo8MMLnkZfqcBA3GRFgQsPyDddHWmXceRLjRX8LJRaH  https://accountnum22.com/EOS5kgeCLuQo8MMLnkZfqcBA3GRFgQsPyD 0.0000
accountnum33  EOS63CnoyfeEQDjXXxwywN5PPKW7RYHC9tbtmvb8vFBGZooktz7kG  https://accountnum33.com/EOS63CnoyfeEQDjXXxwywN5PPKW7RYHC9t 0.0000
accountnum44  EOS6kBaCHrvz7VdUfFBLrLdhNjXYaKBmRkpDXU9PhbEUiHbspr7rz  https://accountnum44.com/EOS6kBaCHrvz7VdUfFBLrLdhNjXYaKBmRk 0.0000
```

####    启动生产节点

使用以下命令启动生产者。 回想一下，在本教程中，所有生产者都在一台服务器上运行，因此使用命令行参数来确保每个生产者都使用自己的目录。

在每个生产者的单独窗口中，运行以下nodeos命令，调整每个生产者的命令行参数。

``` 
$ nodeos --genesis-json ~/eosio_test/accountnum11/genesis.json --block-log-dir ~/eosio_test/accountnum11/blocks --config-dir ~/eosio_test/accountnum11/ --data-dir ~/eosio_test/accountnum11/ --http-server-address 127.0.0.1:8011 --p2p-listen-endpoint 127.0.0.1:9011 --enable-stale-production --producer-name accountnum11 --private-key '[ "EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --plugin eosio::history_api_plugin --p2p-peer-address localhost:9022  --p2p-peer-address localhost:9033  --p2p-peer-address localhost:9044
```

请注意，在所有生产者都启动之前，会生成以下连接错误消息。

``` 
1826099ms thread-0   net_plugin.cpp:2927           plugin_startup       ] starting listener, max clients is 25
1826099ms thread-0   net_plugin.cpp:676            connection           ] created connection to localhost:9022
1826099ms thread-0   net_plugin.cpp:1948           connect              ] host: localhost port: 9022 
1826099ms thread-0   net_plugin.cpp:676            connection           ] created connection to localhost:9033
1826099ms thread-0   net_plugin.cpp:1948           connect              ] host: localhost port: 9033 
1826099ms thread-0   net_plugin.cpp:676            connection           ] created connection to localhost:9044
1826099ms thread-0   net_plugin.cpp:1948           connect              ] host: localhost port: 9044 
1826100ms thread-0   net_plugin.cpp:1989           operator()           ] connection failed to localhost:9022: Connection refused
1826100ms thread-0   net_plugin.cpp:1989           operator()           ] connection failed to localhost:9033: Connection refused
1826100ms thread-0   net_plugin.cpp:1989           operator()           ] connection failed to localhost:9044: Connection refused
```

为了方便起见，可以复制以下命令行以在帐户accountnum22，accountnum33和accountnum44的单独shell窗口中运行nodeos。 
如果您运行这些命令，您可以看到多个节点在对等配置中运行时如何响应。 这假定帐户已经使用相关的密钥对进行了抵押（密钥对可以在下面的每个命令行中看到）。

``` 
$ nodeos --genesis-json ~/eosio_test/accountnum22/genesis.json --block-log-dir ~/eosio_test/accountnum22/blocks --config-dir ~/eosio_test/accountnum22/ --data-dir ~/eosio_test/accountnum22/ --http-server-address 127.0.0.1:8022 --p2p-listen-endpoint 127.0.0.1:9022 --enable-stale-production --producer-name accountnum22 --private-key '[ "EOS5kgeCLuQo8MMLnkZfqcBA3GRFgQsPyDddHWmXceRLjRX8LJRaH","5Jh4rseyguLx5Y7KE2oLL81PRmDcyyzbyyyJd3GvdHijKqENbRk" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --p2p-peer-address localhost:9011  --p2p-peer-address localhost:9033  --p2p-peer-address localhost:9044
$ nodeos --genesis-json ~/eosio_test/accountnum33/genesis.json --block-log-dir ~/eosio_test/accountnum33/blocks --config-dir ~/eosio_test/accountnum33/ --data-dir ~/eosio_test/accountnum33/ --http-server-address 127.0.0.1:8033 --p2p-listen-endpoint 127.0.0.1:9033 --enable-stale-production --producer-name accountnum33 --private-key '[ "EOS63CnoyfeEQDjXXxwywN5PPKW7RYHC9tbtmvb8vFBGZooktz7kG","5JWp5K24x5dbAFBNP7hwzSRS7XjD7wjHL4nrAwSLdRuJnERjgqB" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --p2p-peer-address localhost:9011  --p2p-peer-address localhost:9022  --p2p-peer-address localhost:9044
$ nodeos --genesis-json ~/eosio_test/accountnum44/genesis.json --block-log-dir ~/eosio_test/accountnum44/blocks --config-dir ~/eosio_test/accountnum44/ --data-dir ~/eosio_test/accountnum44/ --http-server-address 127.0.0.1:8044 --p2p-listen-endpoint 127.0.0.1:9044 --enable-stale-production --producer-name accountnum44 --private-key '[ "EOS6kBaCHrvz7VdUfFBLrLdhNjXYaKBmRkpDXU9PhbEUiHbspr7rz","5KeGoDkbhEdkZTpYqQg2rPZvtxqfWAtGgixuCLUt1Dmoq4NmXCj" ]' --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::http_plugin --p2p-peer-address localhost:9011  --p2p-peer-address localhost:9022  --p2p-peer-address localhost:9033
```

####    创世节点的```genesis.json```文件

假定```genesis.json```文件（见下文）已被复制到相应的帐户目录中。

``` 
{
  "initial_timestamp": "2018-03-02T12:00:00.000",
  "initial_key": "EOS8Znrtgwt8TfpmbVpTKvA2oB8Nqey625CLN8bCN3TEbgx86Dsvr",
  "initial_configuration": {
    "max_block_net_usage": 1048576,
    "target_block_net_usage_pct": 1000,
    "max_transaction_net_usage": 524288,
    "base_per_transaction_net_usage": 12,
    "net_usage_leeway": 500,
    "context_free_discount_net_usage_num": 20,
    "context_free_discount_net_usage_den": 100,
    "max_block_cpu_usage": 100000,
    "target_block_cpu_usage_pct": 500,
    "max_transaction_cpu_usage": 50000,
    "min_transaction_cpu_usage": 100,
    "max_transaction_lifetime": 3600,
    "deferred_trx_expiration_window": 600,
    "max_transaction_delay": 3888000,
    "max_inline_action_size": 4096,
    "max_inline_action_depth": 4,
    "max_authority_depth": 6,
    "max_generated_transaction_count": 16
  },
  "initial_chain_id": "0000000000000000000000000000000000000000000000000000000000000000"
}
```

**注意：每个节点的输出都会在该节点工作目录下的stderr文件中捕获，例如```./nodes/accountnum44/stderr```。
默认情况下，脚本启动的节点将持续运行。 必须小心避免来自30多个同时运行的节点的磁盘填充，每个节点分别将其输出写入文件系统中的文件。**

####    为生产节点投票

因为账户被抵押，生产者被注册，生产者的投票可以开始了。

以下命令可以投票。

``` 
$ cleos system voteproducer prods accountnum23 accountnum11 accountnum33
```

在这个例子中，accountnum23已投票给accountnum11和accountnum33。 只有达到15％的可用选票被投票后才能开始批量生产。

####    生产者可以要求奖励

在150,000,000个令牌被抵押后，生产者可以要求奖励。 选择获得奖励的最佳时间策略无疑会有很大差异。 本教程不包括该主题。

使用以下要求奖励。

``` 
$ cleos system claimrewards accountnum33
```

####    注册代理和代理投票

帐户可以注册为投票目的的代理。 其他账户可以将其投票委托给代理账户。

注册为投票代理。 在这个例子中，proxyvoter11被设置为代理投票人。

``` 
$ cleos system regproxy proxyvoter11
```

现在其他账户可以委托他们的投票由proxyvoter11代理投票：

``` 
$ cleos system voteproducer proxy accountnum24 proxyvoter11
$ cleos system voteproducer proxy accountnum34 proxyvoter11
```

在这里，accountnum24和accountnum34帐户委托他们的投票给proxyvoter11。

### eosio释权

一旦选出了生产者并满足最低数量要求，```eosio```帐户就可以辞职，并将```eosio.msig```帐户作为唯一的特权帐户。

辞职基本上涉及将```eosio.*```帐户的密钥设置为空。 使用以下命令清除```eosio.*```帐户的所有者和活动密钥：

``` 
$ cleos push action eosio updateauth '{"account": "eosio", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"}}]}}' -p eosio@owner
$ cleos push action eosio updateauth '{"account": "eosio", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"}}]}}' -p eosio@active
```

