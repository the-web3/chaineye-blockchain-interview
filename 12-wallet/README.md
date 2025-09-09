# 区块链钱包面试题目

## 1.描述一下交易钱包的充值和提现流程

### 充值

- 交易所给用户提供地址，用户把钱转进来
- 扫链：获取最新块高，从上一次解析的块开始到最新块高进行交易解析
- 交易解析完之后，如果交易里面 to 是系统用户，则是充值，解析完成之后上账，并通知业务层

### 提现

- 获取需要的签名参数
- 交易离线签名：组织交易，生成待签名消息摘要，将待签名的消息摘要递给签名机进行签名，签名机签名完成之后返回签名串
- 构建完整的交易并发送区块链网络，将签名时计算出来的交易 Hash 或者发送交易时返回交易 Hash 更新到数据库
- 扫链解析到这笔交易，说明提现成功。

## 2.HD 钱包助记词生成的流程

- 第一步：随机熵生成
- 第二步：计算校验和
- 第三步：组合熵和校验和
- 第四步：分割助记词索引
- 第五步：映射为助记词

## 3.助记词的验证过程

- 第一步：检查助记词数量
- 第二步：检查助记词是否在词汇表
- 第三步：将助记词转换成索引
- 第四步：提取种子的校验和
- 第五步：计算校验和
- 第六步：验证校验和

## 4.BIP44 路径

```
m/44'/coin_type'/account'/change/address_index
```

## 5.门限共享秘密拆分过程

- 关键点：门限共享秘密拆分是将一个秘密分成多份，只有当足够多的份（达到门限）组合起来才能恢复原秘密，常用 Shamir 的秘密共享方案。
#### 门限共享秘密拆分的定义

- 秘密-----> n 份，k 份可恢复， 任意 k 份恢复出这个秘密，分别用门限共享算法和逆门限共享秘密算法

门限共享秘密拆分是指将一个秘密 S 拆分成 n 个份额（shares），使得只有当至少 k 个份额（k 为门限）组合起来时，才能准确恢复原秘密 S，而少于 k 个份额无法获得任何关于 S 的信息。这被称为 (k, n) 门限方案。例如，在一个 (2,3) 方案中，秘密被分成 3 份，任何 2 份能恢复秘密，但 1 份无法。

最常见的方法是 Shamir 的秘密共享方案，基于多项式插值和有限域的数学性质。

#### Shamir 的秘密共享方案利用 k-1 次多项式来实现门限共享，具体步骤如下：
1. 选择一个大于秘密的质数 p。
2. 将秘密 S 设为多项式的常数项，随机生成 k-1 个系数，构成一个 k-1 次多项式 f(x)。
3. 为每个参与者分配一个唯一标识 x_i（通常从 1 到 n），计算他们的份额 y_i = f(x_i) mod p。
4. 每个参与者得到一个份额 (x_i, y_i)。

#### 恢复过程
- 任何 k 个参与者可以用他们的份额通过拉格朗日插值法恢复多项式，计算 f(0) 得到原秘密 S。
#### 以下是门限共享秘密拆分的应用场景对比：
|   场景    |    适用性     |      优势       |
|:-------:|:----------:|:-------------:|
| 分布式密钥管理 |  高，适合多方合作  | 防止单点故障，增强安全性  |
|  数据备份   | 高，适合重要数据分发 | 防止数据泄露，需多方恢复  |
|  访问控制   |  高，适合权限分配  | 灵活控制访问，满足门限要求 |

## 5.MPC 算法 Keygen 和 Sign 分别需要经过多少轮共识

GG18

- Keygen: 5 轮
- Sign: 9 轮

GG20

- Keygen: 5 轮
- Sign: 7 轮

## 6.为什么 schnorr 比特币手续费可以降低

schnorr 签名聚合成一个签名，还有短密钥，这样签名的数据会比 ECDSA 短很多，所以相对于 ECDSA 签名的交易会便宜很多

## 7. 为什么比特币早期时候不直接用 schnorr 签名算法

因为当时 schnorr 算法在专利期

## 8. 相比较之下 EDDSA 性能，安全性都会高一些，为什么比特币以太坊用了 ECDSA，没有用 EDDSA

- ECDSA 是基于更早的标准（如 FIPS 186-4 和 ANSI X9.62）发展的，因此在密码学界和工业界有较长的使用历史和广泛的标准化支持。它被大量系统和协议（如
  TLS 和 Bitcoin）采用，形成了一个庞大的生态系统。
- 虽然 EdDSA 有一些优势，如不容易受到侧信道攻击的影响（如时间攻击和缓存攻击），但 ECDSA
  的安全性也已经过广泛的研究和验证。对于很多开发者和企业来说，使用一个已被长期验证的算法是更为保守和安全的选择。
- EdDSA 通常具有更高的签名速度和较快的验证速度，尤其是在大多数软件实现中。然而，对于已经高度优化的 ECDSA
  实现，性能差异在许多应用中可能并不明显。
- EdDSA 的设计更为简单且更不易出错，特别是在处理随机数生成等方面。然而，ECDSA 由于使用历史更长，开发者更为熟悉其使用和管理。

## 9.中心化钱包开发里面的充值，提现，归集，转冷，冷转热开发业务流程描述

- 💡💡接口返回的 to 是交易所系统里面的用户地址，这笔交易为充值交易；
- 💡💡接口返回的 from 是交易所系统里面的用户地址，这笔交易为提现交易；
- 💡💡接口返回的 to 是交易所的归集地址，from 是系统的用户地址，这笔交易资金归集交易；
- 💡💡接口返回的 to 地址是冷钱包地址，from 地址时热钱包地址，这笔交易热转冷的交易。
- 💡💡接口返回的 from 地址是冷钱包地址，to 地址时热钱包地址，这笔交易冷转热的交易。

### 充值

- 获得最新块高；更新到数据库
- 从数据库中获取上次解析交易的块高做为起始块高，最新块高为截止块高，如果数据库中没有记录，说明需要从头开始扫，起始块高为 0；
- 解析区块里面的交易，to 地址是系统内部的用户地址，说明用户充值，更新交易到数据库中，将交易的状态设置为待确认。
- 所在块的交易过了确认位，将交易状态更新位充值成功并通知业务层。
- 解析到的充值交易需要在钱包的数据库里面维护 nonce, 当然也可以不维护，签名的时候去链上获取

### 提现

- 获取离线签名需要的参数，给合适的手续费
- 构建未签名的交易消息摘要，将消息摘要递给签名机签名
- 构建完整的交易并进行序列化
- 发送交易到区块链网络
- 扫链获取到交易之后更新交易状态并上报业务层

### 归集

- 将用户地址上的资金转到归集地址，签名流程类似提现
- 发送交易到区块链网络
- 扫链获取到交易之后更新交易状态

### 转冷

- 将热钱包地址上的资金转到冷钱包地址，签名流程类似提现
- 发送交易到区块链网络
- 扫链获取到交易之后更新交易状态

### 冷转热

- 手动操作转账到热钱包地址
- 扫链获取到交易之后更新交易状态

## 10.有用过 rosetta api, 请简单描述起作用，并举几个钱包常用的接口说明

Bitcoin Rosetta API 是由 Coinbase 提出的 Rosetta
标准的一部分，旨在为区块链和钱包提供一个统一的接口标准。这个标准化的接口使得与各种区块链的交互更加容易和一致，无论是对交易数据的读取还是写入。目前已经支持很多链，包含比特币，以太坊等主流链，也包含像
IoTex 和 Oasis 这样的非主流链。

### 用到的接口

- /network/list：返回比特币主网和测试网信息。
- /network/status：返回当前最新区块、已同步区块高度、区块链处理器的状态等。
- /block 和 /block/transaction：返回区块和交易的详细信息，包括交易的输入输出、金额、地址等。
- /account/balance：通过查询比特币节点，返回指定地址的余额。

### 发送交易到区块链网络

- /construction/submit：通过比特币节点提交签名后的交易。

### 以下是`Rosetta API`交易流程的表格化表示：

|   步骤    |            	端点            |        	输入         |      	输出      |    	备注     |
|:-------:|:-------------------------:|:------------------:|:-------------:|:----------:|
|  检查余额   |    	/account/balance	     |       账户地址	        |     余额数组      | 	确保有足够 ICP |
|  定义操作	  |            -	             |         -	         |     操作数组	     |    手动构造    |
|  预处理交易  | 	/construction/preprocess |     	网络标识符、操作	     | options, 所需公钥 |  	验证操作合法性  |
| 获取元数据	  |  /construction/metadata	  |   网络标识符、options    |   	metadata   |  	获取动态数据   |
| 生成未签名交易 | 	/construction/payloads	  | 网络标识符、操作、metadata	 |  未签名交易、签名负载	  |    准备签名    |
|  离线签名   |            	-             |     	签名负载、私钥	      |     签名结果	     |   可离线完成    |
| 组合签名交易	 |  /construction/combine	   |   网络标识符、未签名交易、签名   |    	签名交易	     |    合并签名    |
|  提交交易	  |   /construction/submit    |    	网络标识符、签名交易     |    	交易哈希	     |    提交网络    |
|  等待确认   |      	/transaction	       |    网络标识符、交易哈希	     |  交易详情（含区块信息）  | 	检查是否被区块包含 |

## 11.ETH2.0 的 epoch, slot 和 block 简述

### Slot（时隙）

定义：Slot 是以太坊2.0中最基本的时间单位，每个slot都有一个指定的时间长度。在每个 slot 中，可能会有一个区块被提议并添加到链中。
时间长度：一个 slot 的长度为 12 秒。这意味着每 12 秒会有一个新的 slot。
功能：在每个 slot 中，网络中的验证者将有机会提议一个新的区块。这些提议者是通过权益证明（PoS）随机选择的。

### Epoch（纪元）

定义：Epoch 是由多个连续的slot组成的更长时间段。在 Eth2.0 中，Epoch 用于管理和组织验证者的活动。
组成：一个 Epoch 由 32 个 slot 组成。
时间长度：由于一个 slot 是12秒，一个 Epoch 的总长度是 384 秒（即6.4分钟）。
功能：Epoch 是用来实现共识机制的一部分。在每个 Epoch 结束时，网络会进行状态和共识的检查和调整，包括对验证者的奖励和惩罚。

### Block（区块）

定义：Block 是包含交易和其他相关数据的记录单元。在以太坊2.0中，每个slot可能会有一个区块被提议，但不保证每个 slot 都有区块。
内容：一个区块包含区块头、交易列表、状态根哈希、签名等数据。
创建过程：在每个 slot 开始时，网络会随机选出一个验证者来提议区块。该验证者将创建一个包含新交易和其他必要信息的区块，并广播到网络中。

## 12.中心化钱包开发中为什么需要确认位，您怎么理解这个确认位的

在确认位过了之后交易回滚的难度很大，每条链不一样，根据历史和经验来定，以太坊的话可以参照区块状态来做

## 13.简单描述以太坊交易类型，并说明这个交易类型的作用

- legacy: 历史遗留交易类型，签名体为 `gasLimit` 和 `gasPrice`
- EIP1559: `BaseFee` 和 `MaxPriorityFee` 类型
- EIP4844: `blob` 交易类型
- EIP2930: `Access List` 类型

## 14.去中心化和中心化钱包开发中的异同点有哪些？

### 密钥管理方式不同

HD 钱包私钥在本地设备，私钥用户自己控制
交易所钱包中心化服务器(CloadHSM, TEE 等)，私钥项目方控制

### 资金存在方式不同

HD 资金在用户钱包地址
交易所钱包资金在交易所热钱包或者冷钱包里面

### 业务逻辑不一致

中心化钱包：实时不断扫链更新交易数据和状态
HD 钱包：根据用户的操作通过请求接口实现业务逻辑

## 15.发生硬分叉时，做为钱包的开发，您应当怎么去处理这种状况， 以 ETHPOW 和 ETH2.0 分叉这个过程

认可共识比较

## 16.TON 支持合约吗？若支持，请说出其合约开发语言

是的，TON（The Open Network）支持智能合约。TON 的设计目标之一就是提供一个高性能、可扩展的区块链平台，能够支持复杂的去中心化应用（DApps）和智能合约。

### TON 的智能合约开发可以使用以下几种语言：

- FunC
    - 这是 TON 官方支持的主要合约开发语言，类似于 C 语言，语法较为底层，上手难度较高。目前 TON
      生态中的主流应用，如钱包和去中心化交易所（DEX），大多使用 FunC 编写。它适合需要深入控制合约逻辑的开发者。
- Tact
    - 由社区支持的高级语言，语法类似于 Solidity，易于上手，适合初学者或希望快速开发简单合约的开发者（如 Token 或
      NFT）。不过，Tact 目前仍在快速发展中，功能可能不够完善，例如不支持合约升级，Gas 费用也较高。
- Tolk
    - TON 官方最新推出的语言，旨在未来逐步替代 FunC。Tolk 相较 FunC 更简洁，且官方计划将所有文档和示例逐步迁移至
      Tolk。目前它处于早期阶段，文档和生态支持还在完善中。
- Fift
    - 一种底层语言，类似于汇编语言，主要用于 TON 虚拟机（TVM）的低级操作。一般开发者很少直接使用，更多用于特定场景或调试。

### 对于大多数开发者来说：

- 如果是简单项目（如发行代币或 NFT），Tact 是快速上手的选择。
- 如果需要深入开发复杂逻辑或与现有主流合约交互，FunC 是当前最成熟和广泛使用的语言。
- Tolk 则是面向未来的选择，适合关注长期发展的开发者。

## 17.比特币的地址有哪些格式，请说明

比特币地址的格式主要基于其前缀和编码方式，可以分为以下几类：

1. Legacy地址（P2PKH）
    - 前缀：以“1”开头。
    - 示例：1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN2。
    - 描述：这是比特币最初的地址格式，基于Pay-to-Public-Key-Hash（P2PKH），要求接收者使用对应的私钥签名以证明所有权。
    - 特点：交易费用较高，二维码和手动输入较不方便，适合与不支持SegWit的老式钱包交互。
2. P2SH地址
    - 前缀：以“3”开头。
    - 示例：3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy。
    - 描述：Pay-to-Script-Hash（P2SH）地址用于更复杂的脚本支付，例如多签地址或嵌套SegWit地址。
    - 特点：支持复杂的交易条件，广泛用于交易所和第三方钱包，但兼容性较Legacy地址稍好。
3. SegWit地址（Bech32）
    - 前缀：以“bc1”开头。
    - 子类型：
        - P2WPKH地址：通常以“bc1q”开头，例如：bc1qar0srrr7xfkvy5l643lydnw9re59gtzzwf5mdq。
        - P2WSH地址：以“bc1”开头且地址较长，例如：bc1qrp33g0q5c5txsp9arysrxwj7a0r24kheu37q2vc9iam624dmybsceqc2f（52字符）。
    - 描述：SegWit（隔离见证）是比特币的一个升级，旨在提高交易效率和降低费用。Bech32编码使用32个字符（小写字母和数字），包含错误纠正码，能检测大部分输入错误。
    - 特点：P2WPKH适合标准公钥哈希支付，P2WSH适合脚本哈希支付，交易费用较低，抗输入错误能力强。
4. Taproot地址（Bech32m）
    - 前缀：通常以“bc1p”开头，例如：bc1p5d7rjq7g6rdk2yhzks9smlaqtedr4dekq08ge8ztwac72sfr9rusxg3297。
    - 描述：Taproot是2021年引入的SegWit版本1升级，使用Bech32m编码（Bech32的修改版本），旨在提高隐私和灵活性，支持Schnorr签名和更复杂的智能合约。
    - 特点：提供更好的安全性和较低的费用，多签交易在外观上与单签交易无异，增强了隐私保护。目前采用率逐渐提高，但并非所有钱包均支持。

- 地址格式的演进与对比
- 比特币地址格式的演进反映了网络的扩展需求和安全性提升：
    - Legacy地址：最早的格式，适合早期用户，但交易大小较大，费用较高。
    - P2SH地址：引入了对复杂脚本的支持，兼容性较好，常用在多签或嵌套SegWit场景。
    - SegWit地址：通过隔离见证减少了交易数据量，降低了费用，Bech32编码提高了用户友好性。
    - Taproot地址：作为SegWit的进一步优化，结合Bech32m编码，显著提升了隐私和智能合约能力。

### 以下是各格式的对比表：

|      格式类型       |  前缀  |                              示例地址                              |           特点            |
|:---------------:|:----:|:--------------------------------------------------------------:|:-----------------------:|
| Legacy (P2PKH)  |  1   |               1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN2               |     原始格式，费用高，适合老式钱包     |
|      P2SH       |  3   |               3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy               |       支持复杂脚本，兼容性好       |
| SegWit (P2WPKH) | bc1q |           bc1qar0srrr7xfkvy5l643lydnw9re59gtzzwf5mdq           |  标准SegWit支付，费用低，抗输入错误   |
| SegWit (P2WSH)  | bc1  | bc1qrp33g0q5c5txsp9arysrxwj7a0r24kheu37q2vc9iam624dmybsceqc2f  |       脚本哈希支付，地址较长       |
| Taproot (P2TR)  | bc1p | bc1p5d7rjq7g6rdk2yhzks9smlaqtedr4dekq08ge8ztwac72sfr9rusxg3297 | 隐私增强，支持Schnorr签名，采用率提升中 |

## 18.描述一些 UTXO 和账户模型的区别

- 账户模型：
    - 账户模型用于像以太坊这样的区块链。每个账户有一个余额，交易时直接从你的余额中扣除，添加到接收者的余额，就像银行转账。
- UTXO:
    - UTXO 模型用于像比特币这样的区块链。每个交易有输入和输出，输入是之前未花费的交易输出（UTXO），输出是新的
      UTXO。就像你有几张钞票，要买东西时用掉一些，可能还会有找零。

### 主要区别

- 交易处理：UTXO 需要消耗整个 UTXO 并可能创建找零；账户模型直接调整余额，无需找零。
- 状态管理：UTXO 区块链跟踪所有 UTXO；账户区块链跟踪账户余额。
- 隐私：UTXO 链隐私更好，因为交易不直接与账户关联；账户链交易与特定账户相关联。
- 复杂性：账户链更易支持复杂智能合约；UTXO 链脚本能力较简单。

### 以下是两者的关键差异，整理为表格形式：

|  方面  |                                UTXO模型                                |                          账户模型                          |
|:----:|:--------------------------------------------------------------------:|:------------------------------------------------------:|
|  定义  |  使用未花费交易输出记账方法，无账户/钱包概念，源于比特币白皮书 (https://bitcoin.org/bitcoin.pdf)。  |              类似银行账户的余额管理系统，如以太坊、EOS、Tron。              |
| 示例链  |               比特币、比特币现金、Zcash、Litecoin、Dogecoin、Dash。                |                  以太坊、EOS、Tron、以太坊经典。                   |
| 交易过程 |                消耗现有 UTXO 创建新 UTXO，UTXO 必须整体消耗（如现金钞票）。                | 可部分花费余额，无需找零。例如，从 100 ETH 发送 37.5 ETH，余额直接变为 62.5 ETH。 |
| 交易示例 | 发送 3.75 BTC，输入一个 10 BTC UTXO：接收者得 3.75 BTC，发送者得 6.25 BTC 找零（新 UTXO）。 |      发送 37.5 ETH：发送者余额变为 62.5 ETH，接收者得 37.5 ETH。       |
|  效率  |                 效率较低；需跟踪所有 UTXO 计算余额，对 dApp 开发挑战较大。                  |                  效率较高；只需检查发送者余额是否足够。                   |
| 隐私性  |                       隐私性较好，交易不直接与账户关联，通过地址管理。                       |             隐私性较差，所有交易与特定账户关联，除非使用额外隐私工具。              |
| 复杂性  |                     脚本能力有限，适合简单货币交易，难以支持复杂智能合约。                      |                 支持图灵完备智能合约，适合 dApp 开发。                 |

## 20.解释一下什么是 EVM 同源链，举例说明一下

1. EVM 同源链是指能够运行以太坊虚拟机（EVM）智能合约的区块链，开发者可以用 Solidity 编写合约并部署到这些链上，体验与以太坊类似。
2. 这些链允许开发者复用以太坊生态的工具和应用，降低开发成本，同时通常提供更低的交易费用和更高的性能。
3. 一些 EVM 同源链如 Avalanche C-Chain 和 Polygon，不仅支持以太坊的智能合约，还通过自己的扩展提升了交易速度和隐私保护。

### 以下是几个知名的 EVM 同源链及其特点，整理如下表：

|        链名         |      类型      |              特点              |           示例用例            |
|:-----------------:|:------------:|:----------------------------:|:-------------------------:|
|      Polygon      |    二层扩展方案    |    低交易费用，高吞吐量，PoS桥接以太坊主网     | NFT 市场（如 OpenSea）、DeFi 协议 |
|     BNB Chain     |     独立公链     |    高性能，低费用，支持 Binance 生态     |       去中心化交易所、稳定币发行       |
| Avalanche C-Chain |  子网（EVM 兼容）  |       快速交易，低延迟，支持多子网架构       |       DeFi 应用、跨链桥接        |
|      Fantom       |     独立公链     | 使用 Lachesis 共识，快速确认，低 gas 费用 |       DeFi 协议、链上治理        |
|     Arbitrum      | 二层扩展方案（乐观卷积） |     低费用，继承以太坊安全性，适合高频交易      |         链上游戏、支付应用         |
|     Optimism      | 二层扩展方案（乐观卷积） |     低 gas 费用，专注于以太坊生态扩展      |       DeFi 协议、链上社交        |

## 21.ERC721 和 ERC1155 区别与联系

- ERC721 和 ERC1155 的区别：ERC721 专为非同质化代币（NFT）设计，每个代币都是独特的；ERC1155 可以管理同质化代币和非同质化代币，更加灵活，支持批量操作。
- ERC721 和 ERC1155 的联系：ERC1155 可以创建非同质化代币，包含了 ERC721 的功能，是更全面的标准。

### 以下是 ERC721 和 ERC1155 的详细对比，整理为表格形式：

|  方面  |                         ERC721                          |                       ERC1155                       |
|:----:|:-------------------------------------------------------:|:---------------------------------------------------:|
| 代币类型 |                 仅支持非同质化代币（NFT），每个代币唯一。                  |          支持同质化、非同质化和半同质化代币，可在一个合约中管理多种类型。           |
| 批量操作 | 不支持批量转移，需要多次调用 transferFrom 或 safeTransferFrom，gas 成本高。 | 支持批量转移（如 safeBatchTransferFrom），一次交易可处理多个代币，节省 gas。 |
|  用途  |                适合仅需管理独特资产的项目，如数字艺术品或收藏品。                |            适合需要管理多种代币类型的项目，如游戏生态或资产管理平台。            |
|  效率  |                    效率较低，适合简单 NFT 应用。                    |                效率更高，适合复杂场景，减少合约部署成本。                |
| 批准机制 |               使用 approve 函数，针对特定代币设置批准地址。               |     使用 setApprovalForAll，允许另一个地址管理所有代币，方便市场操作。      |

## 22. Cosmos 共识算法是什么

## 23. Cosmos 钱包资金精度

## 24. Cosmos 签名结构中的 account_number 和 seqence 怎么获取

## 25. memo 是什么，在中心化钱包开发中有什么作用，请举例说明那些链带有 Memo, Memo 另一个名字是什么

## 26. 简述 Cosmos 的 Interchain Security 和 IBC Protocol

## 27. cosmos-sdk 的应用场景

## 28. solana 交易签名有有效期说法吗？若有情描述什么场景的签名会出现这种状况，怎么去解决？

### 29. 怎么去检验账号处于 active？

### 30. solana 代币精度

### 31. 什么是 EVM 同源链，EVM 同源链钱包和 Ethereum 钱包开发中有什么区别

### 32. Cosmos 新版本和老版本签名异同点点

### 33. 简要说明 EOS 账户的激活过程

### 34. 和 EOS 同源的有哪些链

### 35. SUI 链的特点

### 36. Tron 签名和 EVM 链的不同点（和 leagcy 交易类型相比较）

### 37.KDA 由多少条链组成，账户这些链之后有关联吗

### 38.KDA 跨链转账的流程描述

### 39.KDA 共识算法独立吗？矿工奖励有关联吗

### 40.ERC721 和 ERC1155 区别

### 41.NFT MINT 流程

### 42.LSD 产品的质押流程（以 lido 为例说明）

### 43. Solana 质押流程

### 44. Tezos 质押流程

## 45.如何计算一笔 BTC 交易的所需要的 gasFee，有什么方案

比特币交易的“gasFee”实际上是指交易费用（transaction fee），以 satoshis 为单位计算，取决于交易大小（以虚拟字节 vB 为单位）和当前每
vB 的费用率。

### 交易费用通过以下公式计算：

交易费用（sat） = 交易大小（vB） × 每 vB 费用率（sat/vB）

- 交易大小（vB）根据交易类型不同而变化，例如标准 P2PKH 交易约 226 vB，SegWit P2WPKH 交易约 140 vB。
- 每 vB 费用率由市场决定，可通过 Bitcoin Core、Blockchain.com 或 BitcoinFees 等工具查询。

|       交易类型       | 典型 vB 大小 |              说明               |
|:----------------:|:--------:|:-----------------------------:|
|   标准 P2PKH 交易    | 约 226 vB |    一个输入、两个输出（一个给接收者，一个找零）。    |
| SegWit P2WPKH 交易 | 约 140 vB |     使用隔离见证，输入和输出较小，费用更低。      |
|   复杂交易（多输入/输出）   |  视情况而定   | 每个额外输入约增加 41-62 vB，输出约 31 vB。 |

### 每 vB 费用率的获取

费用率由市场动态决定，基于当前 mempool（待处理交易池）的拥堵程度。以下是获取费用率的主要方案：

- Bitcoin Core 的 estimatefee 命令：
    - 如果运行 Bitcoin Core 节点，可使用 `estimatefee` 命令，输入期望的确认块数（如 1 块确认），返回建议的费用率。
    - 示例：`estimatefee 1` 返回当前网络中 1 块确认所需的费用率（sat/vB）。
- 第三方服务：
    - 使用网站如 Blockchain.com 或 BitcoinFees，提供实时费用率建议，通常分为高、中、低优先级。
    - 例如，当前高优先级可能为 20 sat/vB，中优先级 10 sat/vB，低优先级 5 sat/vB。
- 钱包工具：
    - 钱包如 Electrum 或 BlueWallet 提供动态费用建议，允许用户选择“快速”、“经济”或自定义费用率。
    - 例如，选择“快速”可能对应 20 sat/vB，确保下一块确认。

### 计算示例

- 假设一个 SegWit P2WPKH 交易，vB 大小为 140，当前费用率为 10 sat/vB:
    - 交易费用 = 140 × 10 = 1400 sat = 0.00001400 BTC
- 再假设一个标准 P2PKH 交易，vB 为 226，费用率为 10 sat/vB:
    - 交易费用 = 226 × 10 = 1400 sat = 0.0.00002260 BTC

可见，SegWit 交易费用更低。

### 以下是计算交易费用的具体方案：

- 手动计算：
    - 确定交易类型，估算 vB 大小（基于输入/输出数量）。
    - 从上述来源获取当前费用率，代入公式计算。
    - 适合技术用户，但需注意 vB 计算的复杂性。
- 钱包自动计算：
    - 大多数比特币钱包（如 Electrum、BlueWallet）自动计算费用，基于用户选择的优先级。
    - 用户可自定义费用率，适合灵活控制。
- 在线计算器：
    - 使用在线工具如 Bitcoin Transaction Fee Calculator，输入交易详情（输入/输出数量、类型），获取费用估算。
    - 适合非技术用户，快速获取结果。
- 动态估算算法：
    - Bitcoin Core 使用基于 mempool 的算法，预测不同确认时间所需的费用率。
    - 第三方服务可能使用中位数或百分位数（如 90% 交易支付的费用率）估算。

### 46.如何计算一笔evm交易的所需要的gasfee，有什么方案

## 47.如何处理BTC的交易执行缓慢，有什么方案，分别有什么区别？

1. 提高交易费用（Replace By Fee, RBF）
    - 如果你是发送方，且钱包支持“替换按费用”（RBF），可以替换原交易，设置更高的费用以加速确认。需要确保原交易未确认且支持RBF。
    - 定义：RBF允许发送方替换未确认交易，新的交易需支付更高费用以提高优先级。
    - 工作原理：原交易需设置RBF标志（opt-in RBF），新交易必须支付更高的总费用，且输出保持相同或更高。矿工会优先处理费用更高的交易。
    - 适用场景：适合发送方，且钱包支持RBF（如Bitcoin Core、Electrum）。
    - 实施步骤：
        1. 确认原交易未确认且支持RBF。
        2. 使用钱包的“增加费用”功能（如Blockstream Green）或手动创建新交易。
        3. 设置更高费用率（如从5 sat/vB提高到20 sat/vB）。
    - 限制：需钱包支持，部分钱包默认关闭RBF；若原交易已确认，RBF无效。
    - 示例：假设原交易费用为1400 sat（140 vB × 10 sat/vB），可替换为费用2260 sat（140 vB × 16 sat/vB）以加速。
2. 使用子支付父（Child Pays for Parent, CPFP）
    - 如果你是接收方，且控制了被卡交易的输出，可以创建一个新交易，花费该输出并支付高费用，激励矿工同时确认两者。适合接收方操作。
    - 定义：接收方创建一个新交易，花费被卡交易的输出，并支付高费用，激励矿工同时确认两者。
    - 工作原理：新交易（子交易）的高费用率使矿工愿意同时确认其依赖的父交易。比特币共识规则要求父交易先于子交易被确认。
    - 适用场景：适合接收方，且控制了被卡交易的输出地址。
    - 实施步骤：
        1. 确认被卡交易输出地址由你控制（如Electrum中查看详情）。
        2. 创建新交易，输入为被卡交易的输出，输出为你的新地址，设置高费用（如20 sat/vB）。
        3. 广播新交易，等待矿工确认。
    - 限制：需有足够余额支付子交易费用；若父交易输出已被花费，CPFP不可用。
    - 示例：若父交易费用低（5 sat/vB），子交易可设20 sat/vB，总费用率提高，矿工更倾向确认。
3. 交易加速服务
    - 通过第三方服务（如ViaBTC或BitAccelerate）提交交易ID，支付费用让矿工优先处理。服务有免费和付费选项，但需注意潜在风险。
    - 定义：第三方服务通过与矿池合作，优先处理用户提交的交易，通常需支付费用。
    - 工作原理：用户提交交易ID（TXID），服务通过多节点重播或直接请求矿池优先处理。部分服务免费，部分收费。
    - 适用场景：当RBF和CPFP不可用时，或用户不熟悉技术操作。
    - 常见服务：
        - ViaBTC：提供免费（限时）和付费服务，付费可加速1000次/小时，费用0.0001 BTC/KB (ViaBTC Transaction Accelerator)。
        - BitAccelerate：免费重播交易至10节点，无需注册 (BitAccelerate)。
        - BTC.com：批处理加速，费用低，但需支付加速费。
    - 限制：效果不确定，可能存在诈骗风险；收费服务需额外成本。
    - 示例：提交TXID至ViaBTC，支付0.0001 BTC/KB，交易可能1小时内确认。
4. 等待网络恢复
    - 如果不急，可以等待网络拥堵缓解，交易自然确认，但可能需要较长时间。
    - 定义：不采取行动，等待网络拥堵缓解，交易自然被矿工确认。
    - 工作原理：比特币每10分钟生成一个块，块大小1MB，拥堵时低费用交易被延迟。网络恢复后，交易可能被处理。
    - 适用场景：不急需资金，且能接受较长等待时间（如周末网络较空闲）。
    - 限制：可能需数小时或数天，交易可能被内存池丢弃。
    - 示例：若当前费用率10 sat/vB，等待至周末费用率降至5 sat/vB，交易可能被确认。

#### 令人惊讶的细节：SegWit的优势

令人惊讶的是，使用SegWit交易因体积小（约140 vB vs 226 vB），费用低，更易被矿工优先确认，适合未来交易预防延迟。

### 以下是各方案的对比表：

|   方案   |  适用角色  |    费用    |    技术要求    | 效果不确定性 |      适用场景      |
|:------:|:------:|:--------:|:----------:|:------:|:--------------:|
|  RBF   |  发送方   | 无额外（原交易） | 中等（需RBF支持） |   低    |  发送方，钱包支持RBF   |
|  CPFP  |  接收方   | 无额外（原交易） | 中等（需控制输出）  |   低    |   接收方，控制输出地址   |
| 交易加速服务 | 发送/接收方 |  可能需付费   | 低（提交TXID）  |  中等至高  | 其他方案不可用，需快速确认  | 
| 等待网络恢复 | 发送/接收方 |    无     |     无      |   高    | 不急需资金，能接受长等待时间 |

- 选择建议：
    - 优先尝试RBF（发送方）或CPFP（接收方），无额外成本，效果较确定。
    - 若无法操作，可选择信誉良好的交易加速服务，但注意费用和风险。
    - 若不急，可等待网络恢复，但需耐心。

### 48.如何处理EVM的交易执行缓慢，有什么方案，分别有什么区别？

### 49.请设计一个合约的多签钱包

### 50.你怎么对接的tee？怎么请求的tee？tee之间是如何相互通讯的？

### 51.mpc密钥的安全性保证

### 52.Solana 代币的特殊性

### 53.aptos代币的特殊性

### 54.椭圆曲线算法底层实现，以及rsv是什么？分别介绍一下

### 55.ecdsa 和 eddsa 区别

### 56.签名机有支持HD钱包方式吗？

### 57.签名的安全传输方案

### 58.BTC和ETH签名的全流程

### 59.交易里面如何处理合约充值

### 60.什么是 Bitcoin 交易延展性，隔离见证是如何消除了交易延展性

### 61.solana地址大小写敏感怎么处理，比如你发给A的地址却发到了a的地址，有遇到过吗，怎么处理

### 62. solana没有abi 你怎么处理合约交易

### 63 要统计很多代币合约的热度，交易量，用户排行榜等，这个系统等架构数据库啊，技术选型你怎么设计？

### 64 正常派生和硬化派生的区别与联系

## 65 以太坊RLP序列化时ChainID的处理

- RLP（Recursive Length Prefix）：RLP是以太坊中用于序列化数据结构的编码方式，主要用于编码交易、区块等数据结构。

- ChainID的作用：ChainID用于区分不同的以太坊网络（如主网、测试网等），防止重放攻击（即在一个网络上的交易被复制到另一个网络）。

- ChainID在RLP中的处理：

    - 在以太坊交易中，ChainID是EIP-155引入的，用于签名交易。

    -
  在RLP编码中，ChainID会被包含在交易的签名数据中，具体格式为：[nonce, gasPrice, gasLimit, to, value, data, chainId, 0, 0]。

    - 签名时，ChainID会被编码到签名的v值中，公式为：v = recovery_id + 35 + chainId * 2。

- 总结：ChainID通过RLP编码和签名机制，确保交易只能在特定网络中生效。

## 66 Protobuf序列化之后的二进制结构

- Protobuf（Protocol Buffers）：是一种高效的二进制序列化格式，由Google开发，用于结构化数据的序列化和反序列化。

- 二进制结构：

    - Protobuf使用Tag-Length-Value（TLV）格式编码数据。

    - Tag：由字段编号和数据类型组成，占用1个或多个字节。

    - 字段编号：用于标识字段。

    - 数据类型：标识字段的类型（如Varint、64-bit、Length-delimited等）。

    - Length：对于长度可变的数据类型（如字符串、字节数组），会有一个长度字段。

    - Value：字段的实际值。

- 示例：

    - 对于字段int32 id = 1;，如果id = 42，编码结果为：08 2A。

        - 08：Tag（字段编号1，数据类型Varint）。

        - 2A：Value（42的Varint编码）。

- 总结：Protobuf的二进制结构紧凑高效，适合网络传输和存储。

## 67.Shamir共享秘密的本质和使用流程

Shamir共享秘密的本质和使用流程

- Shamir共享秘密：由Adi Shamir提出，是一种秘密共享方案，将秘密分成多个部分（称为“份额”），只有收集到足够数量的份额才能恢复秘密。

- 本质：

    - 基于多项式插值，将秘密编码为一个多项式的常数项。

    - 生成多个点（份额），只有收集到足够数量的点才能重建多项式并恢复秘密。

使用流程：

- 初始化：

    - 选择一个素数p作为有限域。

    - 选择一个秘密s（常数项）。

    - 生成一个k-1次多项式：f(x) = s + a₁x + a₂x² + ... + aₖ₋₁x^(k-1)。

- 生成份额：

    - 为每个参与者生成一个点(x, f(x))。

- 恢复秘密：

    - 收集至少k个点，使用拉格朗日插值法重建多项式，恢复秘密s。

- 总结：Shamir共享秘密是一种安全的分布式存储方案，适用于密钥管理和多方协作场景。

## 68.secp256k1和r1的区别，以及比特币和以太坊为何选择secp256k1

- secp256k1：

    - 椭圆曲线方程：y² = x³ + 7。

    - 特点：高效、计算速度快，适合区块链场景。

    - 使用范围：比特币、以太坊等区块链系统。

- secp256r1（也称为NIST P-256）：

    - 椭圆曲线方程：y² = x³ - 3x + b。

    - 特点：参数由NIST标准化，安全性较高，但计算效率略低于secp256k1。

    - 使用范围：TLS、SSL等传统加密场景。

- 区别：

    - 曲线参数不同：secp256k1的曲线参数简单，secp256r1的参数由NIST定义。

    - 性能：secp256k1在签名和验证速度上更快。

    - 安全性：两者均被认为安全，但secp256k1的简单性使其更受区块链青睐。

- 比特币和以太坊选择secp256k1的原因：

    - 效率：secp256k1的计算速度更快，适合区块链的高频交易场景。

    - 透明性：secp256k1的曲线参数简单，避免了NIST可能存在的后门争议。

    - 社区共识：比特币率先采用secp256k1，以太坊为兼容性和一致性也选择该曲线。


## 69.回滚区块的时候，是怎么确认回滚哪个块

## 70.mpc 签名后的交易 signature 是明文，不需要加密嘛？不会被黑客盗走

## 71.如果 solana 那边区块节点挂了，出现了区块堆积，他们那边恢复以后，我们从最新区块接着扫，然后断掉的区块用另一个定时任务去扫，但是流水表(交易日报)怎么接上去

## 72.怎么解决重放攻击，有些链是没有链 ID，比如 xlm 链没有链ID

## 73.你们项目的归集是怎么做的? 那么大批量的归集（20-30万笔）, 是一笔一笔的签名吗?

## 74.你们项目的充值提现有没有做过什么安全措施

## 75.你们签名机的热钱包和冷钱包是怎么进行数据隔离和权限隔离的?

## 76.合约内部交易是怎么解析的

## 77.用第三节点和自建节点的区别与联系

## 78.如何获取 Solana 智能合约的 IDL

## 79.uniswap上发一笔交易，过程发生了什么越详细越好

## 80.请详细说一下 Solana 交易的解析问题

## 81.交易所储备金证明系统怎么做的

## 82.大规模的批量提现

## 83.假充值怎么解决

假充值例子：
- 伪造代币合约充值：部署假的代币合约，模仿ETH、USDT代币的名称和符号，但合约地址不一样。通过调用合约方法，直接修改目标钱包的余额显示。
  - 解决方式：
    - 钱包系统内有白名单地址，只有配置了白名单的合约，才能往交易所内充值；
    - 代币充值，钱包也是有个token表来配置，只有配置了的代币才能往里面冲。
- 虚假转账记录：通过伪造交易数据，让钱包显示收到一笔转账，但这笔交易并未发生在链上。
  - 技术原理：
    - 利用钱包软件的漏洞，直接修改本地显示数据（较少见，需用户使用被篡改的钱包）
    - 或通过虚假节点返回伪造的区块链数据（针对未验证的 RPC 端点）
  - 解决方式：使用一个RPC节点，来执行一笔转账交易，但不上链，看看能否成功，用来表明该账户是不是有充值这么多金额

一般解决方式：
- 通过区块链浏览器检查交易记录
- 验证合约地址
- 测试转账
- 请求风控校验

## 84.交易所资产证明，证明自己有钱

- https://github.com/the-web3/chaineye-binance-por 币安的资产证明的整个项目概述

## 85.私钥怎么和助记词对应，路径推导的协议怎么推导出来的

如果是使用12个助记词的话，首先生成128位随机熵，然后进行SHA-256哈希计算得到哈希值，取出哈希值的前四位，拼接到随机熵后面。之后取每十一位来转换成10进制索引，从2048个单词里得到相应的助记词。
然后使用PBKDF2基于密码的密钥派生函数，以助记词为入参，对助记词进行2028次哈希计算（HMAC-SHA512），得到512位（64字节）的Seed。

种子生成后，通过BIP32标准推导主私钥和主链码：

对种子使用HMAC-SHA512计算，得到主私钥（前32字节）和主链码（后32字节）

## 86.怎么算solana多少cu，一个账户占多少字节

## 87.Solana签名，怎么签 feepayer，两个签名怎么拼

签名规则：
- feePayer必须是第一个签名者
- 其他签名者按accountKeys中需要签名的顺序排列
- 签名者对消息部分（Message）的哈希进行签名
问题场景：
- 目标：feePayer使用私钥A签名，指令的发起者使用私钥B签名
- 挑战：
  - 默认情况下，Transaction.sign会用单一私钥签名所有需要签名的账户
  - 需要分离签名过程，确保每个私钥只签名其对应的账户
手动拼接步骤：
- 构造交易信息：
  - 使用@solana/web3.js创建交易并提取消息部分
  - 设置feePayer和其他签名账户
- 序列化消息：获取交易的Message部分并序列化，用于签名
- 分别签名：
  - 使用每个私钥对消息进行独立签名
  - 确保签名顺序与accountKeys中需要签名的账户顺序一致
- 拼接交易：将签名数组和消息部分手动合成完整的交易字节数组
- 验证和发送：将拼接后的交易提交到网络

```
import { Connection, Keypair, Transaction, SystemProgram, PublicKey } from "@solana/web3.js";
import { sign } from "@solana/web3.js/lib/ed25519"; // 手动签名工具

const connection = new Connection("https://api.devnet.solana.com", "confirmed");

// 生成两个密钥对
const feePayer = Keypair.generate(); // 支付费用的账户
const sender = Keypair.generate();   // 转账发起者
const recipient = Keypair.generate(); // 接收者

async function createAndSignTransaction() {
  // 构造交易
  const tx = new Transaction();
  const { blockhash } = await connection.getLatestBlockhash();
  tx.recentBlockhash = blockhash;
  tx.feePayer = feePayer.publicKey;

  // 添加转账指令
  tx.add(
    SystemProgram.transfer({
      fromPubkey: sender.publicKey,
      toPubkey: recipient.publicKey,
      lamports: 1000000, // 1 SOL
    })
  );

  // 编译消息（不签名）
  const message = tx.compileMessage();
  const serializedMessage = message.serialize();

  // 分别签名
  const feePayerSignature = sign(serializedMessage, feePayer.secretKey); // feePayer 签名
  const senderSignature = sign(serializedMessage, sender.secretKey);     // sender 签名

  // 手动拼接交易
  const numSignatures = 2; // feePayer + sender
  const signatures = [
    feePayerSignature, // 第一个签名是 feePayer
    senderSignature,   // 第二个签名是 sender
  ];

  // 创建完整的交易字节数组
  const txBytes = Buffer.concat([
    Buffer.from([numSignatures]), // 签名数量
    ...signatures.map(sig => Buffer.from(sig)), // 签名数组
    Buffer.from(serializedMessage), // 消息部分
  ]);

  // 转换为 base58 编码（可选，用于 RPC）
  const base58 = require("bs58");
  const encodedTx = base58.encode(txBytes);

  // 模拟交易（验证）
  const simulation = await connection.simulateTransaction(
    Transaction.from(txBytes),
    { commitment: "confirmed" }
  );
  console.log("Simulation Result:", simulation.value);

  // 发送交易（需确保账户有足够 SOL）
  // const signature = await connection.sendRawTransaction(txBytes);
  // console.log("Transaction Signature:", signature);
}

createAndSignTransaction().catch(console.error);
```
构造交易消息：创建交易并设置 feePayer 和指令。
序列化消息：提取交易的 Message 并序列化为字节数组。
分别签名：使用每个私钥对消息签名。
拼接交易：将签名和消息组合成完整交易。
验证和发送：通过 RPC 模拟或提交交易。

Golang实现方式：
```
package main

import (
        "context"
        "fmt"
        "log"

        "github.com/gagliardetto/solana-go"
        "github.com/gagliardetto/solana-go/programs/system"
        "github.com/gagliardetto/solana-go/rpc"
        "github.com/gagliardetto/solana-go/text"
)

func main() {
        // 初始化 RPC 客户端
        client := rpc.New("https://api.devnet.solana.com")

        // 生成密钥对
        feePayer, err := solana.NewRandomPrivateKey()
        if err != nil {
                log.Fatalf("Failed to generate feePayer key: %v", err)
        }
        sender, err := solana.NewRandomPrivateKey()
        if err != nil {
                log.Fatalf("Failed to generate sender key: %v", err)
        }
        recipient := solana.NewWallet().PublicKey()

        fmt.Printf("FeePayer: %s\n", feePayer.PublicKey())
        fmt.Printf("Sender: %s\n", sender.PublicKey())
        fmt.Printf("Recipient: %s\n", recipient)

        // 获取最近区块哈希
        ctx := context.Background()
        recent, err := client.GetLatestBlockhash(ctx, rpc.CommitmentFinalized)
        if err != nil {
                log.Fatalf("Failed to get recent blockhash: %v", err)
        }

        // 构造交易
        tx := solana.NewTransactionBuilder().
                SetFeePayer(feePayer.PublicKey()).
                SetRecentBlockHash(recent.Value.Blockhash).
                AddInstruction(
                        system.NewTransferInstruction(
                                1000000, // 1 SOL
                                sender.PublicKey(),
                                recipient,
                        ).Build(),
                )

        // 编译消息（不签名）
        message, err := tx.Build()
        if err != nil {
                log.Fatalf("Failed to build message: %v", err)
        }

        // 序列化消息
        serializedMessage, err := message.MarshalBinary()
        if err != nil {
                log.Fatalf("Failed to serialize message: %v", err)
        }

        // 分别签名
        feePayerSig, err := feePayer.Sign(serializedMessage)
        if err != nil {
                log.Fatalf("Failed to sign with feePayer: %v", err)
        }
        senderSig, err := sender.Sign(serializedMessage)
        if err != nil {
                log.Fatalf("Failed to sign with sender: %v", err)
        }

        // 拼接交易
        signatures := []solana.Signature{feePayerSig, senderSig}
        completeTx := &solana.Transaction{
                Signatures: signatures,
                Message:    *message,
        }

        // 序列化为字节数组（可选，用于手动检查）
        txBytes, err := completeTx.MarshalBinary()
        if err != nil {
                log.Fatalf("Failed to marshal transaction: %v", err)
        }

        // 模拟交易
        simResult, err := client.SimulateTransactionWithOpts(
                ctx,
                completeTx,
                &rpc.SimulateTransactionOpts{
                        SigVerify:              false, // 不验证签名（测试用）
                        Commitment:             rpc.CommitmentFinalized,
                        ReplaceRecentBlockhash: false,
                },
        )
        if err != nil {
                log.Fatalf("Failed to simulate transaction: %v", err)
        }

        // 输出结果
        if simResult.Value.Err != nil {
                fmt.Printf("Simulation failed: %v\n", simResult.Value.Err)
        } else {
                fmt.Printf("Simulation succeeded:\n")
                fmt.Printf("Units Consumed: %d\n", simResult.Value.UnitsConsumed)
                for _, log := range simResult.Value.Logs {
                        fmt.Println(log)
                }
        }

        // 发送交易（需确保账户有资金）
        // sig, err := client.SendTransaction(ctx, completeTx)
        // if err != nil {
        //         log.Fatalf("Failed to send transaction: %v", err)
        // }
        // fmt.Printf("Transaction Signature: %s\n", sig)
}
```

## 88.哈希环是不是一致性hash算法？


## 89.布隆过滤器，怎么避免误判？


## 90.工作中有没有遇到资金损失，或者系统故障，怎么解决的


## 91.提现成功率多少


## 92.每条链，每天多少笔


## 93.钱包对账怎么做的


## 93.BTC粉尘资金多少
- P2PKH 546聪
- P2SH 828聪
- P2WPKH 270聪
- P2WSH 417聪
- P2TR 276聪

## 94.Solana匹配交易的时候，lookup account什么时候用


## 95.比特币usdt怎么实现的？
比特币原生不支持发行代币，USDT 是通过Omni Layer 协议在比特币之上“搭建了一层”，来实现“代币发行和转账”的。
- blockchain-wallet/Omni/README.md at master · the-web3/blockchain-wallet
- https://github.com/OmniLayer/spec

## 96.BTC你们怎么使用账本的，提现时使用哪种账本，怎么做
- 提现时，通过最小化找零算法，来找一个金额相似的账本用来提现

## 97.不同layer2有些用的debug_traceTransaction，有些不支持，你们用的什么或者你知道接口名字吗

- 支持debug_traceTransaction的链：Optimism、Arbitrum、Polygon PoS
- 不支持的链：
  - zkSync：使用zks_getTransactionDetails接口

## 98.solana使用getblock接口，里面encoding参数json、jsonParsed有什么区别？


## 99.钱包导入助记词，有些钱包就把我不同地址资产识别出来了，怎么做的

## 100.一个私钥，在一个spl-token里面可以有几个 ata 账户

## 100.BTC getBlock接口

## 101.如何验证一笔交易是合法的。在RPC节点里面是否可以验证？如果可以的话，RPC节点是如何验证这个交易是合法的（具体验证哪些字段）

## 102.如何节省gas费？从数据结构设计的角度展开说说？

## 103.调用ec2的服务器被黑了咋办

## 104.温钱包，多签（你们温钱包怎么实现的）

## 105.你们这个签名机方案如果是业务层被端了，你们咋办

## 106.讲一下cosmos的特点

## 107.说明下solana的ata账户

## 108.归集的时候问了下手续费的计算

## 109.token 归集的时候需要注意哪些

## 110.BTC 提现的时候，用哪个账本算法？引出问题：BTC提现的时候，不都是已经归集了吗？为啥还要考虑这个算法

## 111.有没有遇到过，提现的时候没有，还有一些用户资金还没有归集，那么你们是如何处理的呢？

## 112.你们钱包和风控的交互是咋么样的？请具体描述一下


/*
* 自己维护 nonce, 有多笔充值需要归集，如果是同一个地址，可以签名之后丢到 queue 里面发送出去，另一个任务处理 receipt
* 系统里面的热钱包地址是很多，这段时间内有 n 个，将这些交易均分给热钱包，然后再进签名交易进入 queue 发送处理，另一个任务处理 receipt
*/

