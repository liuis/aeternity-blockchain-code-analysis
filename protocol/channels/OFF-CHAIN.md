#Off-chain

每一方都保留一个特定于该频道的状态树。它由所有人组成
渠道数据：帐户，合同等，并具有相同的结构
链状态树。离线交易更新此渠道辅助树。
各方有责任在当地保留此信息。独奏结束
交易提供了包含它的证据，而不是
张贴整棵树。
每个离线更新都包含在通道状态之上应用的更新
树和一个整数值`round`表示它何时发生。自'圆`
必须经常碰撞，提供两个我们可以解释的离线交易
比其他人早进行。


##消息

 -  [控制消息]（#control-messages）
  + [`error`]（＃error）
  + [`ping` /`pong`]（#pingpong）
 -  [建立渠道脱链]（＃established-channel-off-chain）
  + [`channel_open`]（＃channel_open）
  + [`channel_accept`]（＃channel_accept）
  + [funding_created`]（＃funding_created）
  + [funding_signed`]（＃funding_signed）
  + [funding_locked`]（＃funding_locked）
  + [`channel_reestablish`]（＃channel_reestablish）
  + [`channel_reestablish_ack`]（＃channel_reestablish_ack）
 -  [更新]（#state-update）
  + [`update`]（#news）
  + [`update_ack`]（＃update_ack）
  + [`update_error`]（＃update_error）
  + [`deposit_created`]（＃deposit_created）
  + [`deposit_signed`]（＃deposit_signed）
  + [`deposit_locked`]（＃deposit_locked）
  + [`deposit_error`]（＃deposit_error）
  + [`withdraw_created`]（＃withdraw_created）
  + [`withdraw_signed`]（＃withdraw_signed）
  + [`withdraw_locked`]（＃withdraw_locked）
  + [`withdraw_error`]（＃withdraw_error）
 -  [其他互动]（＃other-interaction）
  + [`inband_msg`]（＃inband_msg）
 -  [结束]（＃频道关闭）
  + [`leave`]（＃leave）
  + [`leave_ack`]（＃leave_ack）
  + [`shutdown`]（#shutdown）
  + [`shutdown_ack`]（＃shutdown_ack）
 -  [合同]（＃合同）
  + [在线执法]（＃on-chain-enforcement）
  + [生命周期]（＃contracts-lifecycle）
  + [引用链上对象]（＃contracts-refer-to-chain-data）
  + [气体消耗量]（＃耗气量）


##概述

协议方用于运行智能合约的是两阶段提交协议，
如果一方提议进行更改，则由其他方签署，然后进行
在本地提交更新。这些检查是避免各方获得的必要条件
如果同时提出更新，则会感到困惑。
在更高层次上，为了保持脱链和链上状态同步，各方应该
如果没有获得签名，则拒绝签署更新
其他，例如不要也不要签署链“channel_deposit”交易
也可以更新频道中的状态。

由于各方之间的状态更新是一致的，因此确实如此
因此，各方必须确保不会失去当地的国家
它可能导致他们无法削减过时的州
发表在链上。


##控制消息

###框架

每条消息由1字节的消息代码标识。以下的大小
消息由类型定义 - 请参阅每个人的描述
消息类型。

```
  名称大小（字节）
 ---------------------- ----
| msg_type | 1 |
 ---------------------- ----
|消息| .. |
 ---------------------- ----
```

定义了以下消息代码：
```
  类型代码
-------------------------
| channel_open | 1 |
| channel_accept | 2 |
| channel_reestabl | 3 |
| channel_reest_ack | 4
| funding_created | 5 |
| funding_signed | 6 |
| funding_locked | 7 |
|更新| 8 |
| update_ack | 9 |
| update_error | 10 |
| deposit_created | 11 |
| deposit_signed | 12 |
| deposit_locked | 13 |
| deposit_error | 14 |
| withdraw_created | 15 |
| withdraw_signed | 16 |
| withdraw_locked | 17 |
| withdraw_error | 18 |
|离开| 94 |
| leave_ack | 95 |
| inband_message | 96 |
|错误| 97 |
|关机| 98 |
| shutdown_ack | 99 |
-------------------------
```

###`error`

消息代码：97

明确地传达错误应该使调试更容易。为了避免
复杂的错误处理，一旦成为一个渠道，往往容易出错
返回错误，必须被视为无法使用。因此，错误应该是
仅为不可恢复的故障保留。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| |
 ---------------------- ----
```

 - `channel_id`：临时或最终频道ID
 - `length`：数据字段的长度
 - （可选）`data`：相关信息


###`ping` /`pong`


##建立渠道脱链

```
A（发起人）B
| |
| --- channel_open  - > |
| < -  channel_accept --- |
| |
|  -  funding_created  - > |
| < -  funding_signed --- |
| |
| --- funding_locked  - > |
| < -  funding_locked --- |
| |
```

为了建立一个渠道，双方需要就初始协议达成一致
条件，例如资金数额，最低储备金和锁定时间。

默认情况下，频道的发起人将支付开放费用
交易。


###`channel_open`

消息代码：1

此消息启动通道的打开并传达启动器'
意图潜在的未来派对。

`channel_open`消息应该为接受方提供所有信息
它需要评估是否应该接受该频道。


```
  名称大小（字节）
 ---------------------- ----
| chain_hash | 32 |
 ---------------------- ----
| temporary_channel_id | 32 |
 ---------------------- ----
| lock_period | 2 |
 ---------------------- ----
| push_amount | 8 |
 ---------------------- ----
| initiator_amount | 8 |
 ---------------------- ----
| responder_amount | 8 |
 ---------------------- ----
| channel_reserve | 8 |
 ---------------------- ----
| initiator_pubkey | 32 |
 ---------------------- ----
```

 - `chain_hash`：您要使用的链的事务哈希，例如哈希的
  创世纪
 - `temporary_channel_id`：在所涉及的各方之间随机选择的唯一ID
 - `lock_period`：块中的时间，如果没有，则接受通道关闭
  相互或一般的各方等待新消息。
 - `push_amount`：由发起人初始存款以支持响应者
 - `initiator_amount`：发起者愿意提交的金额
 - `responder_amount`：发起者希望响应者提交的数量
 - `channel_reserve`：双方需要维持的最低金额，金额
  在恶意行为的情况下，一方要输掉的硬币
 - `initiator_pubkey`：发起者想要用来打开的帐户
  渠道


（*** TODO ***：金额的大小是多少。我们想要
阻止频道持有大量资金？）

（*** TODO ***：如果发起人也发送费用和随机数以便那么
响应者可以自己组装`channel_create`吗？）

在未来，国家渠道可能存在于不同的链中，此时
指定`chain_hash`将变得有意义。

`lock_period`可以自由选择。设置得太高可能会锁定资金
在不合作的情况下太长时间并且设置得太低可能会离开
没有足够时间对试图单方面的恶意方做出反应的一方
关闭一个频道。

能够包含一个`push_amount`，它将资金归功于另一个
派对，简化了想要打开频道并付钱给某人的常见情况
立即。交易所可能希望通过这种机制向您发送资金，因为
它只需要一个链上交易，也有副作用
打开一个频道。
如果`push_amount> 0`则发起者应该发送签名更新，
在发送`funding_created`之前，将该金额分配给响应者
信息。

`channel_reserve`确保各方在这种情况下会丢失一些东西
他们开始恶意行事。必须通过以下方式执行此规则
客户，实际上意味着他们不应该签署任何最终的更新
违反这个不变量。


＃＃＃＃ 要求

* *发起方：

 - `chain_hash`必须识别要使用的链
 - `temporary_channel_id`必须在相关各方之间是唯一的
 - `lock_period`应该有足够的时间安全地将事务发布到
  区块链阻止骗子
 - `push_amount`必须小于或等于`initiator_amount`
 - `initiator_pubkey`必须是一个有效的ed25519 pubkey

*响应者*必须在以下情况下中止：

 - `chain_hash`无法识别
 - `initiator_pubkey`不是有效的ed25519 pubkey
 - `temporary_channel_id`在各方之间并不是唯一的

*响应者*如果出现以下情况，应该中止：

 - `initiator_pubkey`没有足够的余额来覆盖
  `initiator_amount`

*响应者*可以在以下情况下中止：

 - `lock_period`太小了
 - `push_amount`太小了
 - `channel_reserve`太大或太小
 - `responder_amount`太大了
 - `initiator_amount`太小了


###`channel_accept`

消息代码：2

此消息由“响应”方发送。它用来传达
他们愿意接受条款提出的条件的条件
发起方。

```
  名称大小（字节）
 ---------------------- ----
| chain_hash | 32 |
 ---------------------- ----
| temporary_channel_id | 32 |
 ---------------------- ----
| minimum_depth | 4 |
 ---------------------- ----
| initiator_amount | 8 |
 ---------------------- ----
| responder_amount | 8 |
 ---------------------- ----
| channel_reserve | 8 |
 ---------------------- ----
| responder_pubkey | 32 |
 ---------------------- ----
```

 - `chain_hash`：您要使用的链的事务哈希
 - `temporary_channel_id`：随机选择的ID在所涉及的各方之间是唯一的，
 - `minimum_depth`：开放事务之前的块数
  被认为是最终`minimum_depth`由响应方设置，因为
  他们通常是提供服务的人。注意'minimum_depth`为0
  （零）将导致微块确认（见下文）。
 - `initiator_amount`：发起者愿意提交的金额
 - `responder_amount`：发起者希望响应者提交的数量
 - `channel_reserve`：双方需要维持的最低金额。这使得
  确保两者都必须丢失一些东西以防他们恶意行事
 - `responder_pubkey`：发起者想要用来打开的帐户
  渠道

（*** TODO ***：这可以是互动的，即如果响应方发送
不同的金额，然后可能会传达它想要这些。）

＃＃＃＃ 要求

*响应*：

 - `chain_hash`必须识别要使用的链
 - `temporary_channel_id`必须在相关各方之间是唯一的
 - `lock_period`应该有足够的时间安全地将事务发布到
  区块链阻止骗子
 - `responder_pubkey`必须是一个有效的ed25519 pubkey

#### Microblock确认

正如[比特币 -  NG博客文章]（http://hackingdistributed.com/2015/11/09/bitcoin-ng-followup/）所述，_Nan microblock提供的严格保证比比特币中的0确认更严格。在Aeternity中，当最小深度确认设置为零时，系统
仍然会等到微块内的事务被发现。这意味着
这笔交易已被闲聊，从mempool接收并被矿工接受。

请注意，高价值事务应该等待许多密钥块
防止分叉重组链，但微块确认至少提供
最初接受交易的收据。

###`channel_reestablish`

消息代码：3

可以恢复已经终止的信道
客户失败或参与者双方同意离开。

```
  名称大小（字节）
 ---------------------- ----
| chain_hash | 32 |
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

有效载荷（`data`）必须是最新的相互签名的offchain状态，并且
客户端必须验证它们是否具有匹配的相应状态树
状态（否则，以后将无法使用该频道。）

在Aeternity节点实现中，状态树被缓存在节点内。
请注意，如果节点重新启动，则缓存的数据不太可能存活（确实如此）
由于性能原因，不会在每次更新时保留。）Aeternity节点频道
fsm自动恢复状态树并验证提供的
国家实际上是最后一个相互签署的国家。

＃＃＃＃ 要求

*响应*：

 - 如果`chain_id`与当前链不匹配，必须中止。
 - 如果有效载荷与最后一次已知的有效载荷不对应，则必须中止
  签署国。
 - 如果它没有相应的状态树（能够验证，则必须中止）
  提供状态的包含证明。

###`channel_reestablish_ack`

消息代码：4

```
  名称大小（字节）
 ---------------------- ----
| chain_hash | 32 |
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

这是对“channel_reestablish”的响应消息，预计包含
相同的信息。

＃＃＃＃ 要求

* *发起方：

 - 如果内容与前面的内容不完全匹配，必须中止
  `channel_reestablish`
 - 如果`chain_id`与当前链不匹配，必须中止。
 - 如果有效载荷与最后一次已知的有效载荷不对应，则必须中止
  签署国。
 - 如果它没有相应的状态树（能够验证，则必须中止）
  提供状态的包含证明。


###`funding_created`

消息代码：5

为了在链上开辟渠道，各方需要合作并共同签署
`channel_create`交易。 `funding_created`消息由。使用
发起者发送初始状态 - 一个`channel_create_tx`对象，已签名
由发起人。

```
  名称大小（字节）
 ---------------------- ----
| temporary_channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```


＃＃＃＃ 要求

*响应*：

 - 如果有效载荷的大小与“长度”不匹配，必须中止
 - 如果`data`无法反序列化为有效，则必须中止
  `channel_create_tx`对象
 - 如果签名对提供的交易数据无效，则必须中止
 - 如果状态的“round”不等于“1”，应该中止


###`funding_signed`

消息代码：6

如果响应者能够验证发送者的签名
`funding_created`消息，然后它应该为状态添加自己的签名
宾语。共同签名的对象将成为最初的脱链状态
渠道。

```
  名称大小（字节）
 ---------------------- ----
| temporary_channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

＃＃＃＃ 要求

* *发起方：

 - 如果有效载荷的大小与“长度”不匹配，必须中止
 - 如果`data`无法反序列化为有效，则必须中止
  `channel_create_tx`对象
 - 如果`data`对象不是提供的初始状态，则必须中止
 - 如果'data`对象不是双方共同签署的话，必须中止。


###`funding_locked`

消息代码：7

打开频道需要进行链上交易。这个交易需要
包含在一个区块中，因为我们只有概率最终性，所以
充分确认，以便链重组的可能性
可以忽略不计。

双方交换该消息以向彼此发出上述信号
条件已经从他们的观点来看，并且只有在他们两个人之后才能满足
同意这个，可以认为该频道是开放的。

所有后续消息必须使用包含的`channel_id`而不是
`temporary_channel_id`。


```
  名称大小（字节）
 ---------------------- ----
| temporary_channel_id | 32 |
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
```


＃＃＃＃ 要求

 - 节点绝不能发送`funding_locked`消息，除非`channel_create`
交易有'minimum_depth`确认。

##状态更新

州更新需要双方同意。

每个更新必须有一个严格增加的轮次，应该开始
在信道初始化时为“0”。

参数：

 - `channel_id`：
 - '余额'：
 - “州”


###`update`

消息代码：8

一旦交换了“funding_locked”消息，频道就会进入
`open`状态。可以通过发送一个来实现对链外状态的改变
`update`消息。此消息的有效负载必须是单独签名的
离链状态，其中`updates`元素包含一个操作列表
要在先前的状态下执行，并且`state_hash`是聚合的
结果状态树的根哈希。接收方必须验证
state是一个有效的结果，然后在`update_ack`中返回，共同签名
信息。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

###`update_ack`

消息代码：9

响应“更新”消息，接收方验证该消息
在`updates`列表中列出的操作，适用于最近的共同签名
state，导致对应于`state_hash`的状态树。如果是这样的话
state对象是共同签名的，然后在`update_ack`中作为有效负载返回
信息。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

###`update_error`

消息代码：10

由于双方都可以发起“更新”请求，因此两者都有可能
可能会在大致相同的时间这样做。特别是在Aeternity节点系统中，
如果在fsm等待其客户端时，`update`请求到达
签署一个新的状态更新，它将恢复自己的更新尝试和
其他参与者的更新请求通过发送`update_error`消息。
`round`元素表示回退状态的回合，必须
是最后一个相互签署的州。接收者不回复。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|圆形| 4 |
 ---------------------- ----
```

###`deposit_created`

消息代码：11

为了将更多资金存入渠道，一方可以发起
一个`deposit_created`请求。有效载荷是单独签名的
`channel_deposit_tx`对象，包括状态哈希和循环
下一个脱链国家，在应用预期的存款操作后
量。发起方只能从自己的链上存入硬币
帐户（相同的公钥）到渠道状态下自己的离线帐户。

请注意，可以存入零金额，基本上可以存入
操作链上快照。

接收方验证操作并共同签署状态，并将其返回
在“deposit_signed”消息中。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```


###`deposit_signed`

消息代码：12

这是对前面的“deposit_created”消息的确认（参见
以上）。收到共同签署的状态后，接收方会验证它是否存在
确实是由建议的存款操作产生的状态。该
然后将`channel_deposit_tx`推送到链，并获取数量
等待确认（`minimum_depth`）。一旦确认
收到，发送一个`deposit_locked`消息，带有散列
`channel_deposit_tx`交易。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

###`deposit_locked`

消息代码：13

收到'minimum_depth`确认后发送此消息
`channel_deposit_tx`交易。有效载荷是散列的
`channel_deposit_tx`事务对象。邮件发送后，
通道返回到'open`状态，新的off-chain状态是
可用。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

###`deposit_error`

消息代码：14

由于双方都可以发起“deposit_created”请求，因此可以
两者都可能在大致相同的时间这样做。特别是在Aeternity节点中
系统，如果在fsm等待时出现`deposit_created`请求
它的客户端要签署一个新的状态更新，它会恢复自己的更新尝试
和另一个参与者的更新请求通过发送`deposit_error`消息。
`round`元素表示回退状态的回合，必须
是最后一个相互签署的州。接收者不回复。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|圆形| 4 |
 ---------------------- ----
```


###`withdraw_created`

消息代码：15

为了从频道中提取硬币，一方可以发起
一个`withdraw_created`请求。有效载荷是单独签名的
`channel_withdraw_tx`对象，包括状态哈希和循环
下一个脱链状态，在应用预期的退出操作后
量。发起方只能从自己的外链中提取硬币
帐户处于渠道状态（相同的公钥）到其自己的链上帐户。
请注意，可以提取零金额，基本上可以
操作链上快照。

接收方验证操作并共同签署状态，并将其返回
在`withdraw_signed`消息中。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

###`withdraw_signed`

消息代码：16

这是对前面的“withdraw_created”消息的确认（参见
以上）。收到共同签署的状态后，接收方会验证它是否存在
实际上是由提议的退出操作产生的状态。该
然后将`channel_withdraw_tx`推送到链，并获取数量
等待确认（`minimum_depth`）。一旦确认
收到后，会发送一个`withdraw_locked`消息，其中包含哈希值
`channel_withdraw_tx`交易。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

###`withdraw_locked`

消息代码：17

收到'minimum_depth`确认后发送此消息
`channel_withdraw_tx`交易。有效载荷是散列的
`channel_withdraw_tx`事务对象。邮件发送后，
通道返回到'open`状态，新的off-chain状态是
可用。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

###`withdraw_error`

消息代码：18

由于双方都可以发起`withdraw_created`请求，因此可以
两者都可能在大致相同的时间这样做。特别是在Aeternity节点中
系统，如果在fsm等待时发出`withdraw_created`请求
它的客户端要签署一个新的状态更新，它会恢复自己的更新尝试
和另一个参与者通过发送`withdraw_error`撤消请求
信息。 `round`元素表示回退状态的回合
必须是最后一个相互签署的州。接收者不回复。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|圆形| 4 |
 ---------------------- ----
```

##其他互动

###`inband_message`

消息代码：96

带内消息是在通道参与者之间发送的任意消息。
有效负载必须限制为65,535字节。 fsm必须提供带内
消息立即发送，接收方必须处理它（例如将其传送给
客户端）立即保留消息排序。

带内消息的一种可能用途是同步脱链状态
更新，但允许任何应用程序。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

##频道关闭


###`离开`

消息代码：94

虽然可以简单地退出频道然后重新建立它，
重建运作的成功取决于对方的保持
最新的国家副本。确保这一点的礼貌方式是发送
一个“请假”请求。预计接收方将以“leave_ack”响应。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
```


###`leave_ack`

消息代码：95

响应“leave”请求发送此消息。发件人可以终止
消息传递后，它的一面。接收器应该等待
在它之前的`leave_ack`终止。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
```

###`shutdown`

消息代码：98

为了以有序的方式关闭通道，“关闭”消息
发送，传递一个单独签名的`channel_close_mutual_tx`事务对象
有效载荷。发件人从最新创建`channel_close_mutual_tx`
共同签名状态，包括相应状态树的根哈希。
接收方必须验证有效载荷是否与其最新的共同签名相对应
state，然后回复一个`shutdown_ack`消息。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

###`shutdown_ack`

消息代码：99

发送此消息是为了响应已验证的“shutdown”消息。寄件人
消息传递后可能会关闭。接收器必须在之后
验证`shutdown_ack`消息的有效负载（必须是相同的
`channel_close_mutual_tx`对象，共同签名），推送`channel_close_mutual_tx`
交易到链上，然后终止。

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
|长度| 2 |
 ---------------------- ----
|数据| N |
 ---------------------- ----
```

##频道关闭

一个频道可以在三种情况下关闭：

双方同意结束并一并签署结账交易，
   然后广播并包含在区块链中。
2.一方希望关闭频道：另一方可能已经失踪
   一段时间或曾经试图作弊。在这种情况下，任一方都可以发布
   双方签署的最新国家，并在此之后申请余额
   协商超时。
恶意方试图发布一个过时的状态，它更喜欢a
   以后的状态。在这种情况下，诚实的一方可以发布由两者签署的州
   更高的一轮，从而证明另一个人试图作弊。
   具有较高轮次的交易将覆盖较低轮次的交易。

在双方决定关闭频道的情况下，可以获得资金
在他们的协议交易被包括在一个区块后，
否则他们必须至少等待'lock_period`块才能发生争议。

```
一个B.
| |
| ----关闭---> |
| <---关闭---- |
| 。 |
| 。 |
|解决待处理的op |
| 。 |
| 。 |
| 。 |
| < -  closing_created --- |
| --- closing_signed  - > |
| |
```

如果双方都希望在协议中关闭频道
在渠道中留下足够的硬币来支付费用，然后是
返回通道中留下的硬币的建议分布是
如下：

```
如果initiator_amount + responder_amount <费用
  返回错误
否则如果initiator_amount> = ceil（费用/ 2）&& responder_amount> = floor（费用/ 2）
  initiator_final：= initiator_amount  -  ceil（费用/ 2）
  responder_final：= responder_amount  -  floor（费用/ 2）
否则，如果responder_amount> = ceil（费用/ 2）&& initiator_amount> =楼层（费用/ 2）
  responder_final：= responder_amount  -  ceil（费用/ 2）
  initiator_final：= initiator_amount  -  floor（费用/ 2）
否则如果initiator_amount> responder_amount
  initiator_final：= initiator_amount  -  fee + responder_amount
  responder_final：= 0
其他
  responder_final：= responder_amount  -  fee + initiator_amount
  initiator_final：= 0
  ```

这是费用的示例分配。如果这被接受为
规范 - 这意味着其中一方将提出这些最终金额
关闭交易和另一个，也遵循这个建议，将
高兴地签字。
最终在链上的是费用和收盘金额
那些政党，那些派对。他们达成协议的过程不属于
协议本身。


###`shutdown`

```
  名称大小（字节）
 ---------------------- ----
| channel_id | 32 |
 ---------------------- ----
```


`shutdown`消息启动通道关闭，可以通过发送
任何一方。一方发送'shutdown`消息后，它不能提出任何消息
更多更新消息。


＃＃＃＃ 要求

在签署链上通道打开之前，无法启动关闭。

发起人不得在`funding_created`之前发送`shutdown`，并且响应者在发送'funding_signed`之前不得发送`shutdown`。
在相应的点之前，各方仍然可以安全地中止该程序
没有承诺任何事情。

###`closing_created`

如果双方同意关闭，那么他们都需要签署`channel_close_mutual`
交易

###`closing_signed`


##当地的州

缔约方需要存储当地国家，以便能够跟踪国家
渠道运营。

 - `chain_hash`
 - `initiator_pubkey`
 - `responder_pubkey`
 - `initiator_amount`
 - `responder_amount`
 - `channel_active`
 - `round`
 - `updated_at`
 - “关闭”
 -  [{}]

##合同

在州渠道内执行合同要求各方能够
初始化虚拟机以运行其智能合约。

合同以轮次执行，由“round”属性表示。

每个方在本地执行每个智能合约并检查签名状态
他们与他们相匹配。在状态和签名的情况下
有效，他们应用更新，然后发出他们的签名。如果有的话
合同执行中的错误或签名不匹配
国家，他们发送一个新的更新与先前的状态，但增加一轮
数字 - 以避免混淆 - 以及它们的签名，以表示某事
出错。

在共同签名模式下运行时，可能需要以某种方式编写合同
避免免费选项问题。

###链上执法

 - 提交代码，状态，输入
 - 代码应该输出新的余额分配？
 - 提交预计将成为新州根的哈希值
 - 政党可以选择从那里继续

With on chain enforcement of contracts it becomes possible to unilaterally force
progress by publishing contract, state and input on chain. A miner would then
execute the contract given the state and inputs to produce a new state. This new
state could then either be used for both parties to continue operation from there
or leave it at that.

### Contracts lifecycle

Contracts are part of the [update](#state-update) mechanism: it is different
updates that are being used. Serialization of those can be found [here](../serializations.md#channel-off-chain-update).

First one participant initiates an update round containing a [channel create
contract update](../serializations.md#channel-off-chain-update-create-contract). It contains all the
information needed for a contract creation. The other participant co-signs
the changes and the contract is considered to be created.

After a contract is created it can be called. For this one of the participants
initiates an update round containing a [channel call
contract update](../serializations.md#channel-off-chain-update-call-contract).
It contains all the information needed for a contract call, including the
contract address. The other participant co-signs the changes and the contract
call is considered to be executed. Its results can be extracted from the calls
tree in the state tree.
Part of the call is the `amount` a participants commits to the contract.这个
is not to be confused with [gas consumption](#gas-consumption) - `amount` are
the tokens moved from the caller's off-chain balance to the off-chain balance
of the contract been called.

### Contracts referring to on-chain data

Contracts being used in channels off-chain calls have the exact same semantics
as [those being used
on-chain](/contracts/contract_transactions.md#contract-call-transaction). Because of the different environment however,
off-chain callsoff-chain calls  might have different results as the on-chain ones.

Since there is no single source of truth, each participant considers their
current view of the chain to be the correct one. This is essential for the
trustless environment. Every off-chain contract call is based on the top of
the chain, as it is seen by each participant. This could cause some differences
in local contract executions. If those can not be resolved, participants can
always rely on the blockchain as an arbiter by using forcing of progress.然后
the top is used as it is seen by the Bitcoin-NG leader.

It is worth mentioning that both local contracts' executions and the forced
progress ones must be fully deterministic and this implies some restrictions
on using on-chain objects in the off-chain contracts.
This is especially true for the chain-related primitives (e.g. coinbase,
timestamp, block height, difficulty). Please refer to the documentation of the
applicable VM version. Registration and updates of names or asking of oracles
is impossible in the off-chain contract calls.

Using on-chain contracts in off-chain ones is a tricky task. On-chain
contracts reference-count contracts that refer to them. They can be deleted
from the blockchain only once they are no longer referenced by any other
contract.
This can not be enforced for off-chain contracts because there is no knowledge
of them on-chain. Also if we were to use an on-chain contract referencing it
by a registered name, the name on-chain could be changed to point to another
different contract. This opens a security hole especially if one of the
participants is in control of the name.
For these reasons off-chain contracts are not allowed to use on-chain ones,
not even stateless on-chain contracts. Participants can still use well-known
contracts that are present on-chain but they have to copy them into their
off-chain state. That way participants take care of their own data in a
trustless manner - they don't have to rely on other entities keeping contracts
on-chain for them.

### Gas consumption

While making off-chain updates that both parties co-sign, no gas is being
consumed. It is worth mentioning that although contract call updates do include
values for the gas limit and the gas price, those are ignored. Assumption is
that since both participants are executing the contracts locally, they are
equal in their energy consumption.

When a dispute arises and a contract is to be called on-chain, the miner that
includes the transaction should be compensated for the energy used.这就是为什么
when forcing progress of off-chain contract calls gas is consumed. Gas limit
and gas prices are specified in the contract call update itself. Now the
values not being used off-chain come into play. Since the force progress is an
unilateral act, it is the forcer that specifies them and it is the forcer's
on-chain balance that is paying for the consumed gas.
