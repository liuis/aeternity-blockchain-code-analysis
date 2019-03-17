＃Æternity共识协议

本文件定义了Æternity共识协议。

＃＃ 介绍

关键词“必须”，“绝不”，“必须”，“应该”，“不要”，“应该”，
本文档中的“不应该”，“推荐”，“可能”和“可选”是
按照[RFC 2119]（https://tools.ietf.org/html/rfc2119）中的描述进行解释。

##表示法

 - `x || y`，x与y连接
 - `sk < -  rand（1 ^ n）`，采样一个随机的n位数

##概述

###区块链

区块链是由分布式分类帐组成的特定实例
块，排列成树，树的根是
`genesis block`。这棵树中的每个节点都只有一个“父”，但可以有
多个孩子。块通过中的`prev_hash`字段指定其父级
块头，创世块具有全“0”的“prev_hash”。

在任何时候，“块高度”是与当前有效“顶部”的距离
阻止创世块。起源块的高度为零。

此处的有效块表示根据所描述的共识规则有效
下面。

我们使用临时随机领导来构建块树。

为了找到给定一个块树，每个节点的最佳有效链
网络中的操作员总结了完成的工作量（参见工作证明）
每个链。最有效的链是那个工作最多的链
完成。这里的任何关系将在收到一个区块时解决，
即它会更喜欢它首先收到的块。


###块

本文档依赖于[比特币-NG协议] [ng]，它描述了构建区块链的规则。

比特币NG引入了两种类型的块：

1.关键块 - 关键块带有领导者的公钥和工作证明
   证据和相关元数据
2.微块 - 微块承载由领导者确认的交易
   相关元数据

两个键块之间可能存在“0-N”微块。微块很小并且它们经常产生，即每三秒钟产生一次。

关键区块由矿工生成。 [每个关键区块] [ng-rewards]引入了10个新硬币。
微块由矿工生成，他们通过挖掘关键块成为领导者。

#### Genesis块

起源块是特殊的，因为它没有父母和
包含Æternity区块链的初始状态。初始状态的一部分
是由ÆternityERC20令牌的分布产生的
以太坊区块链。


###交易

Æternity遵循“胖协议”方法，这意味着协议
本身具有许多内置的功能，而不是例如以太坊，映射
基本交易之外的任何功能都会改变状态
帐户到智能合约。

###交易签名

我们签署前缀为网络ID的序列化交易。
有关详细信息，请参见[serialization]（../ serializations.md #binary-serialization）定义。

```
NetworkId :: binary（）
SerializedObject :: binary（）
签名::二进制（）

签名=符号（NetworkId + SerializedObject）
```

前缀默认为``ae_mainnet``（``binary（）``），它可以通过节点配置进行配置。
前缀不是序列化事务的一部分。我们只为签名添加它。
考虑在分配区块链网络的情况下更改id，
为了降低重播攻击的危险。

###工作证明

[Cuckoo Cycle]（https://github.com/tromp/cuckoo）是用于工作证明的算法。
它旨在防止比特币和
类似的系统，采矿业务由特殊目的主导
硬件。要解决的问题是在二分图中找到循环。

工作证明用于领导者选举，这通过关键区块进行
比特币-NG。微块不需要任何工作证明。

###帐户

### P2P网络

对等网络不是共识的一部分，将在单独的文档中描述。

###硬币和代币

#### Aeons

永旺是Æternity系统中使用的主要硬币。我们使用以下面额：

```
1
1_000
1_000_000
1_000_000_000
1_000_000_000_000
1_000_000_000_000_000
1_000_000_000_000_000_000 |永世
```

##规格

###加密

Blake2b（256位摘要）和ed25519

####键

密钥生成根据[RFC8032]（https://tools.ietf.org/html/rfc8032#page-13）完成，
请注意，我们有64字节的密钥，由连接的种子组成
用公钥来[节省CPU周期时
签名]（https://download.libsodium.org/doc/public-key_cryptography/public-key_signatures.html）
公钥是32字节。

####签名

ECDSA，DER编码（大小约为72字节）

##帐户

```
地址：公钥
平衡：
现时：
```

###块

#### Key Blocks

```
PROTOCOL_VERSION：1（Roma发布）
GENESIS_VERSION：1（罗马发布）
```

 - 密钥块的时间戳必须小于`now（）+ 9m`
 - 密钥块的时间戳必须大于：
   - 最后11个块的中间时间戳（如果高度> = 12）
   - 创世块的时间戳（如果高度<= 11）
 - 版本必须匹配`PROTOCOL_VERSION`
 - `pow_evidence`必须有效
 - 必须有'0 <= nonce <= MAX_NONCE`
 -  txs_hash必须是所有事务的merkle树的正确根哈希。
 -  *** TODO ***：填写缺失的规则

```
PROTOCOL_VERSION：2（Minerva发布）
GENESIS_VERSION：1（罗马发布）
```
 - 可选信息字段添加到密钥头
 - 必须通过将info字段标志设置为1来标记信息字段的存在
 - 必须通过将info字段标志设置为0来标记信息字段的缺失
 - 信息字段（如果存在）未被解释，但包含在块哈希中（即，它正在达成共识）。

标题[序列化格式]（../ serializations.md＃key-blockheader）

阻止[序列化格式]（../ serializations.md＃key-block）

####微块

```
PROTOCOL_VERSION：1
GENESIS_VERSION：1
```

 - 微块的时间戳必须小于`now（）+ 9m`
 - 如果前一个块是微块：
   - 时间戳必须大于或等于前一个块+ 3s的时间戳
 - 如果前一个块是一个关键块：
   - 时间戳必须大于前一个块的时间戳
 - 版本必须匹配`PROTOCOL_VERSION`
 - 签名必须是发行人签发的有效签名
  上一个关键块
 - 微块中交易的气体总和必须低于或等于
  每个微块的气体限制
 -  *** TODO ***：填写缺失的规则

标题[序列化格式]（../ serializations.md #micro-block-header）

阻止[序列化格式]（../ serializations.md #micro-block）


###交易

####签名交易

```
 字段名大小（字节）
 -------------- -----
|数据| var |
 -------------- -----
|签名| var |
 -------------- -----
```

必须有来自属于“sender”的私钥的有效签名。

####交易的常用字段

每个（链上）交易都包含以下字段：
*费用。它必须至少是交易的气体乘以
  最低的汽油价格，（在MINERVA硬叉高度之后）
  '1000000`（* 10 ^ -18）永久。 （在MINERVA硬叉高度之前
  '1`（* 10 ^ -18）永远。）
*生存时间（TTL）。

花费。####花

```
 字段名大小（字节）
 -------------- -----
|发件人| 32 |
 -------------- -----
|收件人| 32 |
 -------------- -----
|金额| 8 |
 -------------- -----
|费用| 8 |
 -------------- -----
| nonce | 8 |
 -------------- -----
```

＃＃＃＃ 加油站

为了控制微块中的事务的大小和数量，
每笔交易都有天然气。所有交易的天然气总和不能
超过每个微块的气体限制，即600万。

交易的气体是以下的总和：
*基础气体;
*其他气体成分，如气体与字节大小成正比
  交易或相对TTL，合同执行所需的天然气。

|交易|基础气体|其他气体成分|
| ---------------------- | -------------- | ---------------------- |
|花费| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（SpendTx）* Gas​​PerByte` |
| Oracle注册| `BaseGas` |与oracle TTL参数“TTL”（解释为相对）成比例，具体为：`ceiling（32000 * RelativeTTL / floor（60 * 24 * 365 / key_block_interval））`和事务的字节大小，具体为：`byte_size（OracleRegisterTx） * Gas​​PerByte` |
| Oracle查询| `BaseGas` |与oracle查询的比例TTL参数`QTTL`（解释为相对），具体为：`ceiling（32000 * RelativeTTL / floor（60 * 24 * 365 / key_block_interval））`和事务的字节大小，具体为：`byte_size（OracleQueryTx ）* Gas​​PerByte` |
| Oracle回复| `BaseGas` |与oracle响应成比例TTL参数`RTTL`在oracle查询中（在状态中的oracle查询中找到，并解释为相对），具体为：`ceiling（32000 * RelativeTTL / floor（60 * 24 * 365 / key_block_interval））`和事务的字节大小，特别是：`byte_size（OracleRespondTx）* Gas​​PerByte` |
| Oracle扩展| `BaseGas` |与oracle TTL参数“TTL”（解释为相对）成比例，具体为：`ceiling（32000 * RelativeTTL / floor（60 * 24 * 365 / key_block_interval））`和事务的字节大小，具体为：`byte_size（OracleExtendTx） * Gas​​PerByte` |
|名称预备| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（NamePreclaimTx）* Gas​​PerByte` |
|姓名索赔| `BaseGas` |与事务的字节大小成比例，具体为：`byte_size（NameClaimTx）* Gas​​PerByte` |
|名称更新| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（NameUpdateTx）* Gas​​PerByte` |
|名称转移| `BaseGas` |与事务的字节大小成比例，具体为：`byte_size（NameTransferTx）* Gas​​PerByte` |
|名称撤销| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（NameRevokeTx）* Gas​​PerByte` |
|频道创建| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（ChannelCreateTx）* Gas​​PerByte` |
|渠道存款| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（ChannelDepositTx）* Gas​​PerByte` |
|渠道退出| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（ChannelWithdrawTx）* Gas​​PerByte` |
|渠道结算| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（ChannelSettleTx）* Gas​​PerByte` |
|频道斜杠| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（ChannelSlashTx）* Gas​​PerByte` |
|频道关闭独奏| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（ChannelCloseSoloTx）* Gas​​PerByte` |
|渠道紧密相互| `BaseGas` |与事务的字节大小成比例，具体为：`byte_size（ChannelCloseMutualTx）* Gas​​PerByte` |
|频道快照独奏| `BaseGas` |与事务的字节大小成比例，特别是：`byte_size（ChannelSnapshotSoloTx）* Gas​​PerByte` |
|渠道力量进展| `BaseGas` |与事务的字节大小成比例，具体为：`byte_size（ChannelForceProgressTx）* Gas​​PerByte`。它还可能包括合同执行的气体。 |
|频道offchain | 0 | 0 |
|合同创建| `5 * BaseGas` |与事务的字节大小成比例，具体为：`byte_size（ContractCreateTx）* Gas​​PerByte`。它还包括合同执行的天然气。 |
|合同电话| `30 * BaseGas` |与事务的字节大小成比例，具体为：`byte_size（ContractCallTx）* Gas​​PerByte`。它还包括合同执行的天然气。 |

`BaseGas`是15 000。

`GasPerByte`是20。

###工作证明

```
HIGHEST_TARGET_SCI：0x2100ffff
HIGHEST_TARGET_INT：0xffff0000000000000000000000000000000000000000000000000000000000000000
NONCE_BITS：64
MAX_NONCE：0xffffffffffffffff
EXPECTED_BLOCK_MINE_RATE：每3分钟60 * 3＃
RECALCULATE_DIFFICULTY_FREQUENCY：10
```

[Cuckoo cycle]（https://github.com/tromp/cuckoo）用作工作证明
算法。解决方案采用数组长度为“l”的形式，其中`l`是长度
（'PROOFSIZE`）二分图中的循环。表示图的大小
通过[`图形大小的2-log，即节点标识符的位大小]（https://github.com/tromp/cuckoo/blob/488c03f5dbbfdac6d2d3a7e1d0746c9a7dafc48f/src/cuckoo.h#L11） EDGEBITS`。困难
具有“M”节点和“N”边缘的图形基于与标准的比率“M / N”
实现固定在“M / N = 1/2”。有关详细信息，请参阅[白皮书]（https://github.com/tromp/cuckoo/blob/master/doc/cuckoo.pdf?raw=true）。这些指数被称为“微米”，而不是
与标题本身中包含的随机数混淆。
这个图是通过用键控散列对每个边缘`i in（0..N）`进行两次散列来生成的
函数`h`（[SipHash]（https://131002.net/siphash/））。用于这些操作的`key`
是块头的哈希加上一个随机数。属于每个边的第一个节点是
由散列`h（key，index * 2）`给出，第二个由'h（key，index * 2 + 1）`给出。

当前默认值：

```
PROOFSIZE：42
EDGEBITS：29
```


```
header =版本||旗帜||身高|| prev_hash || prev_key_hash || state_hash ||矿工||受益人||目标||证据|| nonce ||时间
           32 32 64 256 256 256 256 256 32 42 * 32 64 64
```

为了计算工作证明拼图解决方案，证据和
nonce分别设置为`[0] * 42`和`0`，因为两者都不可用
在谜题解决之前。

当节点收到新的密钥块时，它必须重复该过程，设置
所有0的证据和随机数，以便能够验证工作证明。

```
h_header = Blake2b（标题）
```

杜鹃的默认`key`长度为80个字节。这里的'nonce`表达了
在little-endian表示法中，要与杜鹃循环保持一致，这样做
相同。请注意，散列头和随机数都是base64编码的。
这意味着散列头长为44个字节，nonce为12个字节。

```
key = h_header || nonce || 0..0
        352 92 196
```


一个有效的解决方案必须：

 - 根据解决方案编码规则进行编码
 - 形成一个循环（*** TODO：使这更精确***）
 - 仅包含`micrononces <2 ^ EDGEBITS  -  1`
 - 按升序排列微米级
 - 通过难度检查


####解决方案编码规则

由于`EDGEBITS <32`，我们可以将边缘编码为32位整数，否则
它必须是64位。因此，目前的解决方案是`[u32; 42]`。


####难度目标

参见[杜鹃循环文章第9章]（https://github.com/tromp/cuckoo/blob/master/doc/cuckoo.pdf?raw=true）：

>为了进一步控制，引入了难度目标0 <T <2 ^ 256，并且
>我们强加了Blake2b 256位消化的额外约束
>循环nonce按升序排列小于T，从而降低成功率
>概率乘以2 ^ 256。

因此，我们采用比特币的目标概念，也使用相同的方式
表达出来。

目标阈值与另一个值相关：难度。这是
与PoW任务的硬度成正比：

```
难度= <难度目标1> /目标
```

浮点值。

比特币使用`0x00000000FFFF00000000000000000000000000000000000000000000000000000000`
作为难度1目标（科学记数法中的“0x1d00ffff”，见下文）。对于
Cuckoo Cycle我们需要比SHA-256更轻的过滤解决方案
基本算法比简单的哈希生成要慢得多，所以我们使用了
最大可能值：`0xFFFF0000000000000000000000000000000000000000000000000000000000000000`
（科学记数法中的“0x2100ffff”）为难度1。

我们将当前目标阈值存储在科学的块头中
符号。
难度用于选择新块的获胜分支：块链的难度是每个块的难度的总和。

以科学记数法表示的整数：
`2 ^ 24 * <base-2 exponent + 3> +前3个最重要的字节`（即
[有效数字]（https://en.wikipedia.org/wiki/Significand））。
+ 3对应于有效数字的长度（即，int值为
`0. <significand> * 8 ^ <exponent>`）。
https://en.bitcoin.it/wiki/Difficulty#How_is_difficulty_stored_in_blocks.3F）


####目标调整

一些概念：

```
难度= HIGHEST_TARGET /目标
速率=容量/难度（块/ ms）
容量=矿工产生的每毫秒潜在解决方案的数量

DesiredTimeBetweenBlocks = aec_governance：expected_block_mine_rate（）
DesiredRate = 1 / DesiredTimeBetweenBlocks
```

该算法的基本思想是估计当前的网络容量
基于`N`（= 17）之前的块并使用它来设置新的
目标：

```
NewDifficulty = EstimatedCapacity / DesiredRate
NewTarget = HIGHEST_TARGET / NewDifficulty
              = HIGHEST_TARGET * DesiredRate / EstimatedCapacity
```

我们可以估计用于挖掘给定块“i”的网络容量

```
EstimatedCapacity [i] =难度[i] / MiningTime [i]
MiningTime [i] =时间[i]  - 时间[i  -  1]
```

然后，对所有“N”块的估计容量进行加权（按时间）
每个区块的估计容量的平均值。注意，因为`MiningTime [i]`
需要`Time [i  -  1]`并且我们不在目标中使用创世块时间戳
调整我们不能开始使用目标估计，直到块'N + 2（= 19）`。

```
EstimatedCapacity = Sum（EstimatedCapacity [i] * MiningTime [i]）/ TotalTime
                  =总和（难度[i]）/总时间
                  = Sum（HIGHEST_TARGET / Target [i]）/ TotalTime
```

为了在响应时间和稳定性之间取得良好的平衡，我们使用了一个变体
[的digiSHIELD]（https://github.com/zawy12/difficulty-algorithms/issues/9）。
这意味着我们计算一个调和的TotalTime（算法描述中的总解决时间）：

```
TotalTime'= Sum（SolveTime [i]）
SolveTime [i] = max（-FTL，min（6 * DesiredTimeBetweenBlocks，Time [i]  -  Time [i-1]））
TemperedTotalTime = 0.75 * N * DesiredTimeBetweenBlocks + 0.2523 * TotalTime
```

其中FTL =未来时间限制 - 即允许一个块的时间
“来自未来”。我们使用9分钟（540秒），即“DesiredTimeBetweenBlocks”的3倍。

现在，问题是我们不能做任何浮点运算（to
确保计算可以被其他节点验证），所以我们选择一个
合理的大整数K（= HIGHEST_TARGET * 2 ^ 32）并计算

```
EstimatedCapacity≈Sum（K * HIGHEST_TARGET div Target [i]）/ TotalTime / K.
TemperedTotalTime≈（3 * N * DesiredTimeBetweenBlocks）div 4 +（2523 * TotalTime'）div 10000
```

然后

```
NewTarget = HIGHEST_TARGET * DesiredRate / EstimatedCapacity
          ≈HIGHEST_TARGET* DesiredRate * TotalTime * K / Sum（K * HIGHEST_TARGET div Target [i]）
          ≈DesiredRate* TotalTime * K / Sum（K div Target [i]）
          ≈TotalTime* K div（DesiredTimeBetweenBlocks * Sum（K div Target [i]））
```

[ng]：./ bitcoin-ng.md
[ng-rewards]：./ bitcoin-ng.md#rewards
