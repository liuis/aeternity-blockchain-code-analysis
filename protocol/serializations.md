# 序列化格式

这个文档的序列化格式描述不同对象的二进制格式用于编码的二进制格式,用于:

* 哈希（例如，块哈希）

* 插入Merkle Patricia树（例如，状态树，交易树）。

* 签署交易（即，序列化表格已签署）。

其他格式可用于节点之间的通信或用于用户API。

## 块和区块头

有两种类型的块：

* 微块（micro block）- 包含区块链头和事务列表（transactions）。

* key块（key block） - 仅包含区块头

块以分阶段方式序列化以实现一致性。key块不包含任何比区块头更多的信息，所以
块的表示和区块头是相同的。微块具有静态大小区块头和动态大小“正文”。

区块头以版本号（32位）开头，后跟保留标志字段（32位）。第一个标志位将区块头标记为
key或micro区块头。其他标志字段分别定义区块头类型。请注意，未使用的标志必须设置为零。

|区块头类型标志|区块头类型|
| --- | --- |
| 0 |micro|
| 1 |key|

### key块/区块头

所有字段大小都是静态知道的，可以直接构建作为字节数组。标头以版本号（32位）开头，
后跟保留标志字段（32位）。

对于标志位：

* 在Roma版本中，只使用一个标志位，将区块头标记为键区块头。
* 在Minerva版本中，使用另一位来标记区块头中是否存在可选信息字段。
* 其他标志必须设置为零。

|字段名称|大小（字节）|备注|
| --- | --- | --- |
|version| 32位| |
| key_tag | 1位| |
| info_flag | 1位|来自Minerva协议|
| unused_flag | 1位（设为0）|用于Roma 协议|
| unused_flags | 30位（全部设为0）| |
|height| 8 | |
| prev_hash | 32 | |
| prev_key_hash | 32 | |
| state_hash | 32 | |
|miner矿工| 32 | |
|beneficiary受益人| 32 | |
|target目标| 4 | |
| pow | 168 | |
| nonce | 8 | |
|time时间| 8 | |
|info信息| 0或4 |来自Minerva协议|

注意：

*信息字段存在，大小为4字节或目前不存在（0字节）。必须通过将info_flag设置为1，发信号通知信息字段的存在.

信息字段的内容没有当前的解释，但计划是使用它来发出有关矿工的信息（例如，矿工是否知道即将到来的硬叉）。

### 微块Micro block

|字段名称|尺寸|
| --- | --- |
| Micro Header | 216或238字节（见下文）|
| Micro Body |变量variable|

### 微块头Micro block header

|字段名称|大小（字节）|
| --- | --- |
|version| 32位|
| micro_tag | 1位|
| has_fraud | 1位|
| unused_flags | 30位（全部设为0）|
|height| 8 |
| prev_hash | 32 |
| prev_key_hash | 32 |
| state_hash | 32 |
| txs_hash | 32 |
|time时间| 8 |
| fraud_hash | 0或32 |
|signature签名| 64 |

注意：

* 在构造或验证签名时，签名字段（64字节）必须仅填充零。

* 微块头具有两个有效大小，具体取决于块是否包含欺诈证明。 这是使用has_fraud标志发出的信号。

| has_fraud | fraud_hash size |
| --- | --- |
| 1 | 32字节|
| 0 | 0字节|



## 动态大小对象序列化

在本节中，我们使用[]表示列表，<name>表示字段，以分隔列表中的字段。 我们使用::将字段与其类型分开。 我们使用int（）来表示整数类型，使用binary（）来表示字节数组类型。 可变长度列表在类型域中用[]表示（例如，[int（）]是整数列表）。 固定长度列表用'{}'表示（例如'{int（），int（）}'）表示恰好两个整数的列表。 我们使用++作为列表连接运算符。 我们还使用bool（）类型作为整数0和1的特殊情况，用作布尔类型。

### id（）类型

特殊类型id（）是binary（），表示引用链中对象的标记二进制（例如，hash或公钥）（例如，帐户account，oracle）。 第一个字节是表示标识符类型的标记，剩余的32个字节是标识符本身。 这用于区分可能存在歧义的标识符。

| Id标签|标识符类型|
| --- | --- |
| 1 |帐户 account|
| 2 |名字 name|
| 3 |承诺  commitment|
| 4 | oracle |
| 5 |合约  contract|
| 6 |channel|

在Erlang表示法中，`id（）`类型模式是：
```
<< Tag：1 / unsigned-integer-unit：8，Hash：32 / binary-unit：8 >>
```


### RLP编码

我们使用递归长度前缀编码（https://github.com/ethereum/wiki/wiki/RLP）用于序列化对象
具有动态大小的字段。

RLP确保每个对应的序列化只有一个最低级别的对象，但它只能编码两个原语对象，列表和字节数组。

RLP确保序列化是非空字节数组。

Æternity中的对象被编码为字段列表，其中两个字段 第一个字段描述对象类型和对象版本。

```
[<tag>，<version>，<field1>，<field2> ...]
```

由于所有值都是RLP中的字节数组，因此int（）也需要是一个字节数组。 我们将所有整数编码为无符号的大端字节数组。 为了避免整数编码的模糊性，我们采用与RLP相同的方案，并要求用最小字节数编码整数（即，不允许编码字节数组中的前导零）。 由于不需要，因此序列化格式中不使用负整数。 如果需要，应该延长该计划。


### 二进制序列化

动态大小对象的序列化S，O，定义为：

```
S（O）= rlp（[tag（O），vsn（O）] ++字段（O））
```

标签在下表中给出，字段在后续部分按对象划分。


#### 对象标签表




|Type|Tag|
| --- | --- |
|account 帐户| 10 |
|签署交易 signed transaction| 11 |
|花费交易  Spend transaction| 12 |
| Oracle | 20 |
| Oracle query查询 | 21 |
| Oracle register transaction注册事务 | 22 |
| Oracle query transaction查询事务 | 23 |
| Oracle response transaction响应事务 | 24 |
| Oracle extend transaction扩展事务 | 25 |
|名称服务  Name service name| 30 |
|姓名服务承诺 Name service commitment| 31 |
|名称服务索赔事务  Name service claim transaction| 32 |
|名称服务预告交易 Name service preclaim transaction| 33 |
|名称服务更新事务 Name service update transaction| 34 |
|名称服务撤销事务 Name service revoke transaction| 35 |
|名称服务转移事务 Name service transfer transaction| 36 |
|合约 Contract| 40 |
|合约回调 Contract call| 41 |
|合约创建交易 Contract create transaction| 42 |
|合约回调交易 Contract call transaction| 43 |
|通道创建交易  Channel create transaction| 50 |
|通道存款交易  Channel deposit transaction| 51 |
|通道撤销交易 Channel withdraw transaction| 52 |
|通道强制进度交易 Channel force progress transaction| 521 |
|通道关闭相互交易 Channel close mutual transaction| 53 |
|通道关闭solo交易  Channel close solo transaction| 54 |
|通道削减交易  Channel slash transaction| 55 |
|通道结算交易 Channel settle transaction| 56 |
|通道脱链交易 Channel off-chain transaction| 57 |
|通道离线更新转移  Channel off-chain update transfer| 570 |
|通道离线更新存款  Channel off-chain update deposit| 571 |
|通道离线更新撤销  Channel off-chain update withdrawal| 572 |
|通道离线更新创建合约 Channel off-chain update create contract| 573 |
|通道离线更新呼叫合约  Channel off-chain update call contract| 574 |
|通道 Channel| 58 |
|通道快照事务  Channel snapshot transaction| 59 |
| POI | 60 |
|DB树 Trees with DB | 61 |
|State树| 62 |
| Merkle Patricia树| 63 |
| Merkle Patricia树的value | 64 |
|合约的Merkle Patricia树的value| 621 |
|合约调用'Merkle Patricia树的value| 622 |
|通道的Merkle Patricia树的value| 623 |
| Name service的Merkle Patricia树的value | 624 |
| Oracles的Merkle Patricia树的value | 625 |
|账户的Merkle Patricia树的value| 626 |
|Sophia字节码  Sophia byte code| 70 |
|密钥块  Key block| 100 |
|微块   Micro block| 101 |
|轻微块  Light micro block    此处翻译可能不准确，以代码为准| 102 |

#### 账户 accounts
```
[<nonce> :: int（）
，<balance> :: int（）
]
```
### 签名交易 Signed transaction
```
[<signatures> :: [binary（）]
，<transaction> :: binary（）
]
```

签名已排序。

### 花费交易  Spend transaciton
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

* 帐户标识符。

* oracle标识符。
* 合约标识符。
* 名称标识符，其相关名称条目具有标识符作为指针的值，具有键“account_pubkey”。
  如果对于这样的密钥存在多个指针条目，则使用第一个这样的条目。

#### Oracles 预言机
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

#### 合约

对于地址为`<contractpubkey>`的合约，合约对象的字段（需要预先标记标签和版本）是：

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

合约存储（或状态），它是从（key :: binary（）到value :: binary（））的键值映射
存储在自己的子树中。合约存储价值的关键是：

```
<contractpubkey> <16> <key> :: binary（）
```
`<key>`非空。

每个值只是按原样存储为二进制文件 - 没有标记或版本。
如果值为空二进制，则从树中删除密钥。

合约存储的内容取决于[ABI和VM版本]（/ contracts / contract_vms.md）。

#### 合约回调
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

#### 合约创建交易
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

#### 合约回调交易
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

#### 名称服务名称
```
[<owner> :: id（）
，<expires_by> :: int（）
，<status> :: binary（）
，<client_ttl> :: int（）
，<pointers> :: [{binary（），id（）}]
```

#### 名称服务承诺
```
[<owner> :: id（）
，<created> :: int（）
，<expires> :: int（）
]
```

#### 名称服务声明事务
```
[<account> :: id（）
，<nonce> :: int（）
，<name> :: binary（）%%实际名称，而不是哈希
，<name_salt> :: int（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

#### 名称服务预告交易
```
[<account> :: id（）
，<nonce> :: int（）
，<commitment> :: id（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

#### 名称服务更新事务
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

#### 名称服务撤销事务
```
[<account> :: id（）
，<nonce> :: int（）
，<hash> :: id（）
，<fee> :: int（）
，<ttl> :: int（）
]
```

#### 名称服务转移事务
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

* 帐户标识符。
* 名称标识符，其相关名称条目具有标识符作为指针的值，具有键“account_pubkey”。
    如果对于这样的密钥存在多个指针条目，则使用第一个这样的条目。

### 通道

#### 通道创建交易
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

#### 通道存款交易
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

#### 通道撤销交易
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

#### 通道关闭相互交易
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

#### 通道关闭solo交易
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

#### 通道slash交易
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

#### 通道结算交易
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

#### 通道快照solo交易
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


#### 通道solo进度交易
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

有效负载是序列化的共同签名通道离线事务，或者它是空的。 该回合是新通道的轮次，将通过强制进步产生。 更新是要在旧通道的状态上应用的合同调用更新。 状态散列是在将更新应用于包含证明中提供的通道状态之后生成的通道的状态树的预期根散列。 包含的证明具有与共同签名的有效载荷的状态哈希相同的根哈希。

#### 通道离线更新

通道更新轮次由本小节中定义的各种更新描述。 本文档其余部分中每次提及类型update（）都应理解为指代这些更新中的任何一个。 如果未指定不同的内容，则涉及的任何地址可以属于该通道的脱链状态树的参与者，合同或甚至一些其他地址部分。

###### 通道离线更新转移

这是从一个地址到另一个地址的内部脱链转移。

```
[<from> :: id（）
，<to> :: id（）
，<amount> :: int（）
]

```

###### 通道离线更新存款

这是内部的脱链平衡增量。它被使用了
`channel_deposit_tx`内部表示。

```
[<from> :: id（）
，<amount> :: int（）
]

```

###### 通道离线更新提款

这是内部的脱链平衡减量。它被使用了
`channel_withdraw_tx`内部表示。

```
[<to> :: id（）
，<amount> :: int（）
]

```

###### 通道离线更新创建合约

这是在通道的脱链状态内创建新合约的更新
树。

```
[<owner> :: id（）
，<ct_version> :: int（）
，<code> :: binary（）
，<deposit> :: int（）
，<call_data> :: binary（）
]

```

###### 通道离线更新通话合约

这是在通道的脱链状态内调用合约的更新
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

#### 通道离线交易

通道脱链事务不直接包含在事务树中，而是间接作为以下内容的有效负载：

* 通道关闭交易。
* 通道slash交易。
* 通道快照solo交易。
* 通道强制进度交易。

```
[<channel_id> :: id（）
，<round> :: int（）
，<updates> :: [update（）]
，<state_hash> :: binary（）
]
```

#### 通道
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

#### 状态树上的包含证明（POI）:: poi（）
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

* 每个状态子树不一定包含相同密钥长度的元素。
* 对象本身不包含自己的id，因为它可以从树中的位置派生。
* 用于在每个状态子树中存储每个对象的密钥不一定是从对象本身派生的。
* 包含POI证明的价值包含在POI本身中。

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

#### 状态树::树（）
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

##### 合约的状态树
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

##### 通道的状态树
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

##### Oracles的state树
```
[<otree> :: binary（）
]
```

二进制文件是序列化的Merkle Patricia树。

##### 帐户'状态树
```
[<accounts> :: binary（）
]
```

二进制文件是序列化的Merkle Patricia树。

#### Sophia 字节码（版本1，Roma发布）
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
* transactions *是签名交易。
* proof_of_fraud *列表为空（即，没有欺诈）或具有一个元素（即，包含一个欺诈证据）。


#### 欺诈证明 proof of fraud
```
[<header1> :: binary（）
，<header2> :: binary（）
，<pubkey> :: binary（）
]
```
