＃序列化格式

本文档中的序列化格式描述了二进制格式
用于以所使用的二进制格式编码不同的对象
对于：

*哈希（例如，块哈希）
*插入Merkle Patricia树（例如，州树，交易树）。
*签署交易（即，序列化表格已签署）。

其他格式可用于节点之间的通信或用于
用户API。

##块和标题

有两种类型的块：

*微块 - 包含标题和事务列表。
*密钥块 - 仅包含标题。

块以分阶段方式序列化以实现一致性。关键块
不包含任何比标题更多的信息，所以
块的表示和标题是相同的。微块
具有静态大小标题和动态大小“正文”。

标题以版本号（32位）开头，后跟a
保留标志字段（32位）。第一个标志位将标题标记为
键或微标题。其他标志字段分别定义
标题类型。请注意，未使用的标志必须设置为零。

|标题类型标志|标题类型|
| --- | --- |
| 0 |微|
| 1 |关键|

###密钥块/标题

所有字段大小都是静态知道的，可以直接构建
作为字节数组。标头以版本号（32位）开头，
后跟保留标志字段（32位）。

对于标志位：
*在Roma版本中，只使用一个标志位，将标题标记为键标题。
*在Minerva版本中，使用另一位来标记标题中是否存在可选信息字段。
*其他标志必须设置为零。

|字段名称|大小（字节）|备注|
| --- | --- | --- |
|版本| 32位| |
| key_tag | 1位| |
| info_flag | 1位|来自Minerva协议|
| unused_flag | 1位（设为0）|罗马协议|
| unused_flags | 30位（全部设为0）| |
|身高| 8 | |
| prev_hash | 32 | |
| prev_key_hash | 32 | |
| state_hash | 32 | |
|矿工| 32 | |
|受益人| 32 | |
|目标| 4 | |
| pow | 168 | |
| nonce | 8 | |
|时间| 8 | |
|信息| 0或4 |来自Minerva协议|

注意：

*信息字段存在，大小为4字节或不存在
  目前（0字节）。必须发信号通知信息字段的存在
  通过将info_flag设置为1.信息字段的内容为no
  目前的解释，但计划是用它来发出信号
  有关矿工的信息（例如，如果矿工知道的话
  来硬叉）。

###微块

|字段名称|尺寸|
| --- | --- |
| Micro Header | 216或238字节（见下文）|
| [微体]（＃微体）|变量|

###微块头

|字段名称|大小（字节）|
| --- | --- |
|版本| 32位|
| micro_tag | 1位|
| has_fraud | 1位|
| unused_flags | 30位（全部设为0）|
|身高| 8 |
| prev_hash | 32 |
| prev_key_hash | 32 |
| state_hash | 32 |
| txs_hash | 32 |
|时间| 8 |
| fraud_hash | 0或32 |
|签名| 64 |

注意：

*必须填写* signature *（64字节）的字段
  在构造或验证签名时仅使用零。

*微块头有两个有效的大小取决于是否
  块包含欺诈证据。这是使用
  * has_fraud *标志。

| has_fraud | fraud_hash size |
| --- | --- |
| 1 | 32字节|
| 0 | 0字节|



##动态大小对象序列化

在本节中，我们使用`[]`来表示列表，`<name>`来表示字段，
`，`分隔列表中的字段。我们使用`::`来分隔字段
他们的类型。我们使用`int（）`来表示整数类型和`binary（）`
表示字节数组类型。可变长度列表用表示
类型域中的`[]`（例如，`[int（）]`是整数列表）。
固定长度列表用'{}'表示（例如'{int（），int（）}'）
表示正好两个整数的列表。我们使用`++`作为列表
连接运算符。我们还使用`bool（）`类型作为特殊类型
用作布尔类型的整数“0”和“1”的情况。

### id（）类型

特殊类型`id（）`是表示标记二进制文件的`binary（）`
（例如，散列或公钥）引用链中的对象（例如，
帐号，oracle）。第一个字节是表示哪种类型的标记
标识符，剩下的32个字节是标识符本身。这个
用于区分可能存在的标识符
歧义。

| Id标签|标识符类型|
| --- | --- |
| 1 |帐户|
| 2 |名字|
| 3 |承诺|
| 4 | oracle |
| 5 |合同|
| 6 |渠道|

在Erlang表示法中，`id（）`类型模式是：
```
<< Tag：1 / unsigned-integer-unit：8，Hash：32 / binary-unit：8 >>
```


### RLP编码

我们使用递归长度前缀编码
（https://github.com/ethereum/wiki/wiki/RLP）用于序列化对象
具有动态大小的字段。

RLP确保每个对应的序列化只有一个
最低级别的对象，但它只能编码两个原语
对象，列表和字节数组。

RLP确保序列化是非空字节数组。

Æternity中的对象被编码为字段列表，其中两个字段
第一个字段描述对象类型和对象版本。

```
[<tag>，<version>，<field1>，<field2> ...]
```

由于所有值都是RLP中的字节数组，因此`int（）`需要是一个字节
数组也是如此。我们将所有整数编码为无符号的大端字节
阵列。为了避免整数编码的模糊性，我们调整了
与RLP相同的方案，并要求用整数编码整数
最小字节数（即，不允许编码的前导零）
字节数组）。序列化格式中不使用负整数
因为他们不需要。如果需要，该计划应该是
延长。


###二进制序列化

动态大小对象的序列化“S”定义为“O”

```
S（O）= rlp（[tag（O），vsn（O）] ++字段（O））
```

标签在下表中给出，字段在
后续部分按对象划分。


####对象标签表
|输入|标签|
| --- | --- |
|帐户| 10 |
|签署交易| 11 |
|花费交易| 12 |
| Oracle | 20 |
| Oracle查询| 21 |
| Oracle注册事务| 22 |
| Oracle查询事务| 23 |
| Oracle响应事务| 24 |
| Oracle扩展事务| 25 |
|名称服务名称| 30 |
|姓名服务承诺| 31 |
|名称服务索赔事务| 32 |
|名称服务预告交易| 33 |
|名称服务更新事务| 34 |
|名称服务撤销事务| 35 |
|名称服务转移事务| 36 |
|合同| 40 |
|合同电话| 41 |
|合同创建交易| 42 |
|合同电话交易| 43 |
|渠道创建交易| 50 |
|渠道存款交易| 51 |
|渠道撤销交易| 52 |
|渠道强制进度交易| 521 |
|渠道关闭相互交易| 53 |
|频道关闭独奏交易| 54 |
|频道斜杠交易| 55 |
|渠道结算交易| 56 |
|渠道脱链交易| 57 |
|渠道离线更新转移| 570 |
|渠道离线更新存款| 571 |
|渠道离线更新撤销| 572 |
|渠道离线更新创建合同| 573 |
|渠道离线更新呼叫合同| 574 |
|频道| 58 |
|频道快照事务| 59 |
| POI | 60 |
|树木与DB | 61 |
|州树| 62 |
| Merkle Patricia树| 63 |
| Merkle Patricia树的价值| 64 |
|合约的Merkle Patricia树的价值| 621 |
|合同调用'Merkle Patricia树的价值| 622 |
|频道的Merkle Patricia树的价值| 623 |
| Nameservice的Merkle Patricia树的价值| 624 |
| Oracles的Merkle Patricia树的价值| 625 |
|账户的Merkle Patricia树的价值| 626 |
|索菲亚字节码| 70 |
|密钥块| 100 |
|微块| 101 |
|轻微块| 102 |

####账户
```
[<nonce> :: int（）
，<balance> :: int（）
]
```
###签名交易
```
[<signatures> :: [binary（）]
，<transaction> :: binary（）
]
```

签名已排序。

###花费交易
```
[<sender> :: id（）
，<recipient> :: id（）
，<amount> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
，<nonce> :: int（）
，<payload> :: binary（）
]
```

收件人必须是以下之一：
*帐户标识符。
* oracle标识符。
*合同标识符。
*名称标识符，其相关名称条目具有标识符作为指针的值，具有键“account_pubkey”。
  如果对于这样的密钥存在多个指针条目，则使用第一个这样的条目。

#### Oracles
```
[<owner> :: id（）
，<query_format> :: binary（）
，<response_format> :: binary（）
，<query_fee> :: int（）
，<expires> :: int（）
，<abi_version >> :: int（）
]
```

#### Oracle查询
```
[<sender_address> :: id（）
，<sender_nonce> :: int（）
，<oracle_address> :: id（）
，<query> :: binary（）
，<has_response> :: bool（）
，<response> :: binary（）
，<expires> :: int（）
，<response_ttl> :: int（）
，<fee> :: int（）
]
```

#### Oracle注册事务
```
[<account> :: id（）
，<nonce> :: int（）
，<query_spec> :: binary（）
，<response_spec> :: binary（）
，<query_fee> :: int（）
，<ttl_type> :: int（）
，<ttl_value> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
，<abi_version> :: int（）
]
```

#### Oracle查询事务
```
[<sender> :: id（）
，<nonce> :: int（）
，<oracle> :: id（）
，<query> :: binary（）
，<query_fee> :: int（）
，<query_ttl_type> :: int（）
，<query_ttl_value> :: int（）
，<response_ttl_type> :: int（）
，<response_ttl_value> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
```

#### Oracle响应事务
```
[<oracle> :: id（）
，<nonce> :: int（）
，<query_id> :: binary（）
，<response> :: binary（）
，<response_ttl_type> :: int（）
，<response_ttl_value> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

#### Oracle扩展事务
```
[<oracle> :: id（）
，<nonce> :: int（）
，<ttl_type> :: int（）
，<ttl_value> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

####合同

对于地址为`<contractpubkey>`的合同，合同对象的字段（需要预先标记标签和版本）是：

```
[<owner> :: id（）
，<ct_version> :: int（）
，<code> :: binary（）
，<log> :: binary（），
，<active> :: bool（），
，<referers> :: [id（）]，
，<deposit> :: int（）
]
```

帐户的余额存储在帐户状态树中。

合同存储（或状态），它是从（key :: binary（）到value :: binary（））的键值映射
存储在自己的子树中。合同存储价值的关键是：
```
<contractpubkey> <16> <key> :: binary（）
```
`<key>`非空。

每个值只是按原样存储为二进制文件 - 没有标记或版本。
如果值为空二进制，则从树中删除密钥。

合同存储的内容取决于[ABI和VM版本]（/ contracts / contract_vms.md）。

####合同电话
```
[<caller_id> :: id（）
，<caller_nonce> :: int（）
，<height> :: int（）
，<contract_id> :: id（）
，<gas_price> :: int（）
，<gas_used> :: int（）
，<return_value> :: binary（）
，<return_type> :: int（）
，<log> :: [{<address> :: id，[<topics> :: binary（）]，<data> :: binary（）}]
]
```

####合同创建交易
```
[<owner> :: id（）
，<nonce> :: int（）
，<code> :: binary（）
，<ct_version> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
，<deposit> :: int（）
，<amount> :: int（）
，<gas> :: int（）
，<gas_price> :: int（）
，<call_data> :: binary（）
]
```

####合同电话交易
```
[<caller> :: id（）
，<nonce> :: int（）
，<contract> :: id（）
，<abi_version> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
，<amount> :: int（）
，<gas> :: int（）
，<gas_price> :: int（）
，<call_data> :: binary（）
]
```

####名称服务名称
```
[<owner> :: id（）
，<expires_by> :: int（）
，<status> :: binary（）
，<client_ttl> :: int（）
，<pointers> :: [{binary（），id（）}]
```

####名称服务承诺
```
[<owner> :: id（）
，<created> :: int（）
，<expires> :: int（）
]
```

####名称服务声明事务
```
[<account> :: id（）
，<nonce> :: int（）
，<name> :: binary（）%%实际名称，而不是哈希
，<name_salt> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

####名称服务预告交易
```
[<account> :: id（）
，<nonce> :: int（）
，<commitment> :: id（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

####名称服务更新事务
```
[<account> :: id（）
，<nonce> :: int（）
，<hash> :: id（）
，<name_ttl> :: int（）
，<pointers> :: [{binary（），id（）}]
，<client_ttl> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

####名称服务撤销事务
```
[<account> :: id（）
，<nonce> :: int（）
，<hash> :: id（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

####名称服务转移事务
```
[<account> :: id（）
，<nonce> :: int（）
，<hash> :: id（）
，<recipient> :: id（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

收件人必须是以下之一：
*帐户标识符。
*名称标识符，其相关名称条目具有标识符作为指针的值，具有键“account_pubkey”。
  如果对于这样的密钥存在多个指针条目，则使用第一个这样的条目。

###频道

####频道创建交易
```
[<initiator> :: id（）
，<initiator_amount> :: int（）
，<responder> :: id（）
，<responder_amount> :: int（）
，<channel_reserve> :: int（）
，<lock_period> :: int（）
，<ttl> :: int（）
，<fee> :: int（）
，<delegate_ids> :: [id（）]
，<state_hash> :: binary（）
，<nonce> :: int（）
]
```

####频道存款交易
```
[<channel_id> :: id（）
，<from_id> :: id（）
，<amount> :: int（）
，<ttl> :: int（）
，<fee> :: int（）
，<state_hash> :: binary（）
，<round> :: int（）
，<nonce> :: int（）
]
```

####渠道撤销交易
```
[<channel_id> :: id（）
，<to_id> :: id（）
，<amount> :: int（）
，<ttl> :: int（）
，<fee> :: int（）
，<state_hash> :: binary（）
，<round> :: int（）
，<nonce> :: int（）
]
```

####频道关闭相互交易
```
[<channel_id> :: id（）
，<from_id> :: id（）
，<initiator_amount_final> :: int（）
，<responder_amount_final> :: int（）
，<ttl> :: int（）
，<fee> :: int（）
，<nonce> :: int（）
]
```

####频道关闭独奏交易
```
[<channel_id> :: id（）
，<from_id> :: id（）
，<payload> :: binary（）
，<poi> :: poi（）
，<ttl> :: int（）
，<fee> :: int（）
，<nonce> :: int（）
]
```

有效负载是序列化的签名信道离线事务，或者它是空的。

####频道斜杠交易
```
[<channel_id> :: id（）
，<from_id> :: id（）
，<payload> :: binary（）
，<poi> :: poi（）
，<ttl> :: int（）
，<fee> :: int（）
，<nonce> :: int（）
]
```

有效负载是序列化的签名信道离线事务，或者它是空的。

####渠道结算交易
```
[<channel_id> :: id（）
，<from_id> :: id（）
，<initiator_amount_final> :: int（）
，<responder_amount_final> :: int（）
，<ttl> :: int（）
，<fee> :: int（）
，<nonce> :: int（）
]
```

####频道快照独奏交易
```
[<channel_id> :: id（）
，<from_id> :: id（）
，<payload> :: binary（）
，<ttl> :: int（）
，<fee> :: int（）
，<nonce> :: int（）
]
```

有效负载是序列化的签名通道离线事务，不能为空。


####频道独奏进度交易
```
[<channel_id> :: id（）
，<from_id> :: id（）
，<payload> :: binary（）
，<round> :: int（）
，<update> :: binary（）
，<state_hash> :: binary（）
，<offchain_trees> :: trees（）
，<ttl> :: int（）
，<fee> :: int（）
，<nonce> :: int（）
]
```

有效负载是序列化的共同签名通道离线事务，或者它是空的。
该回合是新通道的回合，将通过强制产生
进展。
更新是要在旧频道上应用的合同调用更新
州。
状态哈希是生成的通道的状态树的预期根哈希
更新后应用于证明中提供的通道状态
包容性
包含的证明具有与共同签名的状态哈希相同的根哈希
有效载荷。

####渠道离线更新

通道更新轮次由各种更新描述，这些更新在中定义
这一小节。在本文档的其余部分中每次提及类型update（）都是
意思是理解为指代这些更新中的任何一个。如果没有指定
不同的东西，涉及的任何地址都可能属于参与者，
合同甚至是该渠道的脱链国家树的其他地址部分。

######频道离线更新转移

这是从一个地址到另一个地址的内部脱链转移。

```
[<from> :: id（）
，<to> :: id（）
，<amount> :: int（）
]

```

######频道离线更新存款

这是内部的脱链平衡增量。它被使用了
`channel_deposit_tx`内部表示。

```
[<from> :: id（）
，<amount> :: int（）
]

```

######渠道离线更新提款

这是内部的脱链平衡减量。它被使用了
`channel_withdraw_tx`内部表示。

```
[<to> :: id（）
，<amount> :: int（）
]

```

######渠道离线更新创建合同

这是在渠道的脱链状态内创建新合约的更新
树。

```
[<owner> :: id（）
，<ct_version> :: int（）
，<code> :: binary（）
，<deposit> :: int（）
，<call_data> :: binary（）
]

```

######频道离线更新通话合约

这是在渠道的脱链状态内调用合同的更新
树。

```
[<caller> :: id（）
，<contract> :: id（），
，<abi_version> :: int（）
，<amount> :: int（）
，<call_data> :: binary（）
，<call_stack> :: [int（）]
，<gas_price> :: int（）
，<gas_limit> :: int（）
]

```

####渠道离线交易

通道脱链事务不直接包含在事务树中，而是间接作为以下内容的有效负载：
*频道关闭独奏交易。
*频道斜杠交易。
*频道快照独奏交易。
*渠道强制进度交易。

```
[<channel_id> :: id（）
，<round> :: int（）
，<updates> :: [update（）]
，<state_hash> :: binary（）
]
```

####频道
```
[<initiator> :: id（）
，<responder> :: id（）
，<channel_amount> :: int（）
，<initiator_amount> :: int（）
，<responder_amount> :: int（）
，<channel_reserve> :: int（）
，<delegate_ids> :: [id（）]
，<state_hash> :: binary（）
，<round> :: int（）
，<solo_round> :: int（）
，<lock_period> :: int（）
，<locked_until> :: int（）
]
```

####状态树上的包含证明（POI）:: poi（）
```
[{<accounts> :: [<proof_of_inclusion> :: {<root_hash> :: binary（），[{<mpt_hash> :: binary（），<mpt_value> :: [binary（）]}]}]}
，{<calls> :: [<proof_of_inclusion> :: {<root_hash> :: binary（），[{<mpt_hash> :: binary（），<mpt_value> :: [binary（）]}]}]}
，{<channels> :: [<proof_of_inclusion> :: {<root_hash> :: binary（），[{<mpt_hash> :: binary（），<mpt_value> :: [binary（）]}]}]}
，{<contracts> :: [<proof_of_inclusion> :: {<root_hash> :: binary（），[{<mpt_hash> :: binary（），<mpt_value> :: [binary（）]}]}]}
，{<ns> :: [<proof_of_inclusion> :: {<root_hash> :: binary（），[{<mpt_hash> :: binary（），<mpt_value> :: [binary（）]}]}]}
，{<oracles> :: [<proof_of_inclusion> :: {<root_hash> :: binary（），[{<mpt_hash> :: binary（），<mpt_value> :: [binary（）]}]}]}
]
```

如果一个子树（例如`<accounts>`）为空，那么它的序列化只是`[]`（例如`<accounts>`是`[]`）否则它至少包含它的根哈希（例如`<accounts> `是`[{<root_hash>，[]}]`）。

注意：`[{<mpt_hash>，<mpt_value>}]`是证明中Merkle Patricia Tree节点的排序列表。

注意：由于POI包含Merkle Patricia Tree节点（例如，不仅仅是它们的哈希）：
*每个状态子树不一定包含相同密钥长度的元素。
*对象本身不包含自己的id，因为它可以从树中的位置派生。
*用于在每个状态子树中存储每个对象的密钥不一定是从对象本身派生的。
*包含POI证明的价值包含在POI本身中。

#### Merkle Patricia Value
```
[<key> :: binary（）
，<val> :: binary（）
]
```
#### Merkle Patricia Tree :: mtree（）
```
[<values> :: [binary（）]
]
```

所有Merkle Patricia值都是序列化的。

####状态树::树（）
```
[<contracts> :: binary（）
，<calls> :: binary（）
，<channels> :: binary（）
，<ns> :: binary（）
，<oracles> :: binary（）
，<accounts> :: binary（）
]
```

所有二进制文件都按如下方式序列化：

#####合同的状态树
```
[<contracts> :: binary（）
]
```

二进制文件是序列化的Merkle Patricia树。

##### Contract Calls'状态树
```
[<calls> :: binary（）
]
```

二进制文件是序列化的Merkle Patricia树。

#####频道的州树
```
[<channels> :: binary（）
]
```

二进制文件是序列化的Merkle Patricia树。

##### Nameservice的状态树
```
[<mtree> :: binary（）
]
```

二进制文件是序列化的Merkle Patricia树。

##### Oracles的州树
```
[<otree> :: binary（）
]
```

二进制文件是序列化的Merkle Patricia树。

#####帐户'状态树
```
[<accounts> :: binary（）
]
```

二进制文件是序列化的Merkle Patricia树。

####索菲亚字节码（版本1，罗马版）
```
[<源代码哈希> :: binary（）
，<type info> :: [{<fun_hash> :: binary（），<fun_name> :: binary（），<arg_type> :: binary（），<out_type> :: binary（）}]
，<byte code> :: binary（）
]
```

#### Sophia字节码（版本2，Minerva发布）
```
[<源代码哈希> :: binary（）
，<type info> :: [{<fun_hash> :: binary（），<fun_name> :: binary（），<arg_type> :: binary（），<out_type> :: binary（）}]
，<byte code> :: binary（）
，<compiler version> :: binary（）
]
```

#### Micro Body
```
[<transactions> :: [binary（）]
，<proof_of_fraud >> :: [binary（）]
]
```
注意：
* * transactions *是签名交易。
* * proof_of_fraud *列表为空（即，没有欺诈）或具有一个元素（即，包含一个欺诈证据）。


####欺诈证明
```
[<header1> :: binary（）
，<header2> :: binary（）
，<pubkey> :: binary（）
]
```
