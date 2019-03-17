#On-chain

操作状态通道至少需要通过a进行初始设置
`channel_create`事务，锁定可配置数量的硬币
每个参与该频道的一方。这需要以形式的同意
来自所有参与者的签名。

在理想情况下，所有参与者都签署了所有链上交易，
意味着没有分歧。可以提交这些操作
立即并且通常会优先于单方面启动的所有操作。

除了`channel_snapshot_solo`之外，没有人签名的交易可以
通过`channel_slash`和`channel_force_progress_tx`交易进行争议。
在正常操作期间，这些独立交易可以无限期地进行争议。
如果频道处于关闭状态，则必须在该频道内发生争议
预先协商的`lock_period`，因为闭包需要有限的终结。

更新链上状态的每个事务都提供了两个必不可少的字段
未来的冲突解决方案：`round`和`state_hash`。状态哈希是根
在链上的应用之后，通道的状态树的哈希值
当地的州树。 `round`是_next_状态通道内部回合。从而
链上更新事务表示链上下一个脱链状态
通道。这样我们可以根据最后一个单独关闭一个频道
在链状态。我们所要做的就是提供包含证明
同样的`state_hash`。


 -  [建立频道]（＃established-a-channel）
+ [`channel_create`]（＃channel_create）
 -  [更新频道]（＃更新频道）
+ [`channel_deposit`]（＃channel_deposit）
+ [`channel_withdraw`]（＃channel_withdraw）
+ [`channel_snapshot_solo`]（＃channel_snapshot_solo）
 -  [关闭频道]（＃closing-a-channel）
+ [`channel_close_mutual`]（＃channel_close_mutual）
+ [`channel_close_solo`]（＃channel_close_solo）
+ [`channel_settle`]（＃channel_settlement）
 -  [争议更新]（#sudputing-updates）
+ [`channel_slash`]（＃channel_slash）
 -  [强制进度]（＃强制进度）
+ [`channel_force_progress_tx`]（＃channel_force_progress_tx）
 -  [通道状态树]（＃channel-state-tree）

##建立一个频道

所有的链上操作都可以由任何同行提交，但我们会假设
发起对等体支付频道的设置。因此`发起者`
必须支付标准交易费用。


###`channel_create`

`channel_create`事务用于注册链上的通道及其
包含在链上导致指定的金额被锁定。

例如

```
帐户（发起人）.balance：=帐户（发起人）.balance  -  initiator_amount

帐户（响应者）.balance：=帐户（响应者）.balance  -  responder_amount

频道（cid）。amount：= initiator_amount + responder_amount
```

序列化定义[这里]（../ serializations.md＃channel-create-transaction）

 - `initiator_id`：发起对等体的帐户ID
 - `responder_id`：响应对等方的帐户ID
 - `initiator_amount`：发起者提交给的无符号金额
  渠道
 - `responder_amount`：响应者提交的无符号硬币数量
  渠道
 - `lock_period`：独奏操作后的争议期
 - `delegate_ids`：代表帐户ID列表。代表们发挥了作用
  独奏结束序列：除了频道的参与者，他们
  是唯一可以提供斜杠交易的人。
 - `state_hash`：信道状态树的根哈希;这没有经过验证，
  只是保留在频道的对象中
 - `ttl`：blockheight target，可以包含此事务
 - “费用”：交易费
 - `nonce`：`initiator`的帐户nonce


`lock_period`的长度是响应性之间的权衡，例如，怎么样
可以提交快速独奏操作和安全性。选择`lock_period`
这为参与者提供了足够的时间来应对潜在的恶意独奏
运营至关重要。

`ttl`是绝对链高。有关各方将要设置
`ttl`到一个比当前链高大很多的值，以避免
不确定。如果包含的费用很低而且交易压力很高，那么
交易最终可能会被困在mempool中一段时间​​。
`ttl`是可选的，没有`ttl`表示“永远”有效的事务。

`fee`和`nonce`指的是`initiator`帐户，即`fee`必须
取自他们的余额，他们的账户的“nonce”必须递增。


＃＃＃＃ 要求

`帐户（发起人）.balance> = initiator_amount +费用`

`帐户（响应者）.balance> = responder_amount`

`initiator_amount> = channel_reserve`

`responder_amount> = channel_reserve`


##更新频道

对开放频道的更新需要所有参与者的签名和a
频道识别（`channel_id`）。

`channel_deposit`和`channel_withdraw`都必须由所有参与者签名
各方，因为改变频道余额可能会改变代码的动态
在频道中运行。

`channel_id`是根据发起者的公钥来计算的
使用Blake2b创建事务和响应者的公钥（256
比特摘要）。

```
channel_id = Blake2b（发起人|| channel_create_tx_nonce || responder）
                        32 32 32
```


###`channel_deposit`

在创建之后将资金存入频道应该允许频道更多
由于更容易平衡它们而长期存在。硬币的数量
发送此事务将像初始一样被锁定
存款。

虽然可能希望允许任何人存入频道，但我们是
将限制存款限制在渠道的同行。这意味着，`from_id`
字段必须是目标频道的参与者之一的地址
标准交易费必须由`from_id`账户支付。

对于正常的通道操作，此操作不是必需的。

序列化定义[此处]（../ serializations.md＃channel-deposit-transaction）

 - `channel_id`：在链上记录的频道ID
 - `from_id`：存款的发件人
 - “金额”：存入的硬币数量
 - `state_hash`：存款后的通道状态树的根哈希
  已被应用;这未经过验证，只保留在频道的对象中
 - “round”：应用存款的渠道内部回合
 - `ttl`：blockheight target，可以包含此事务
 - “费用”：交易费
 - `nonce`：`from_id`的帐户nonce

注意，'round`应该在每次离线更新时递增。这个
意味着，为了进行链上交易，指的是链外状态
被认为是有效的，它必须包括比当前更大的'round`
记录了`频道（channel_id）.round`。

如果此事务有效，则设置：

 - `频道（channel_id）.round：= round`
 - `频道（channel_id）.solo_round：= round`
 - `频道（channel_id）.state_hash：= state_hash`
 - `频道（channel_id）.total_amount：=频道（channel_id）.total_amount +金额


###`channel_withdraw`

通道通常不应用于容纳大量硬币，但
能够取出锁定的硬币可能仍然有用。

序列化定义[这里]（../ serializations.md＃channel-withdraw-transaction）

 - `channel_id`：在链上记录的频道ID
 - `to`：撤回的接收者
 - “金额”：撤回的硬币数量
 - `state_hash`：撤销后的通道状态树的根哈希值
  已被应用;这未经过验证，只保留在频道的对象中
 - “round”：应用撤回的频道内部回合
 - `ttl`：blockheight target，可以包含此事务
 - “费用”：交易费
 - `nonce`：`to`的帐户nonce

“to”帐户必须是目标渠道的参与者。 “金额”必须
小于或等于所有参与者余额的总和，即渠道不能
凭空创造硬币。费用由“to”帐户支付
帐户应该持有足够的硬币来支付费用，即减去费用
在撤回的硬币到来之前。

注意，'round`应该在每次离线更新时递增。这个
意味着，为了进行链上交易，指的是链外状态
被认为是有效的，它必须包括一个大于或等于的圆形
目前录制的`频道（channel_id）.round`。

如果此事务有效，则设置：

 - `频道（channel_id）.round：= round`
 - `频道（channel_id）.solo_round：= round`
 - `频道（channel_id）.state_hash：= state_hash`
 - `频道（channel_id）.total_amount：=频道（channel_id）.total_amount  - 金额


###`channel_snapshot_solo`

为了使渠道既安全又无信任，即使一方走了
离线时，我们提供快照功能。
快照提供了最近在链上记录的脱链状态。这个状态
用'round`和`state_hash`表示。包含频道后
不能使用较旧的状态关闭 - 如'round`所示 - 而不是一个
在快照中提供。

序列化定义[这里]（../ serializations.md＃channel-snapshot-solo-transaction）

 - `channel_id`：在链上记录的频道ID
 - `from_id`：发布交易的帐户
 - `payload`：同一频道的共同签名的脱链状态
 - `ttl`：blockheight target，可以包含此事务
 - “费用”：交易费
 - `nonce`：`from_id`帐户nonce

`from_id`帐户必须是目标渠道的参与者。 `payload`
必须是一个共同签署的脱链国家。它必须是同一频道的一部分
（包含相同的通道ID）并且它必须具有大于的'round`
一个目前记录在链上。

此事务不得触发`lock_period`，并且不得在何时使用
通道处于锁定状态。它可以用来覆盖生成的状态
当通道处于打开状态时，通过`channel_force_progress_tx`。

如果此事务有效，则设置：

 - `Channel（channel_id）.round：= payload.round`
 - `Channel（channel_id）.solo_round：= payload.round`
 - `Channel（channel_id）.state_hash：= payload.state_hash`


##关闭频道

我们希望渠道只在非合作或恶意的情况下关闭
行为。如果所有各方决定关闭该频道，那么结束只是一个问题
发布一个由所涉及的每个人签名的在线交易。

在单独关闭的情况下，操作受`lock_period`的约束，
在此期间，他们可以通过`channel_slash`进行争议。


###`channel_close_mutual`

序列化定义[这里]（../ serializations.md＃channel-close-mutual-transaction）

 - `channel_id`：在链上记录的频道ID
 - `from_id`：发布交易的帐户
 - `initiator_amount_final`：发起者的最终余额
 - `responder_amount_final`：响应者的最终余额
 - `ttl`：blockheight target，可以包含此事务
 - “费用”：交易费
 - `nonce`：`from_id`帐户nonce

`initiator_amount_final`和`responder_amount_final`是商定的硬币分配。
发起人和响应者的帐户余额递增
分别是`initiator_amount_final`和`responder_amount_final`。
该频道必须有足够的总硬币来支付费用以及费用
商定的金额。通道的总关闭量由下式计算
添加发起者和响应者的数量（在结束前）和
费用。如果此总收盘金额低于硬币总金额
专用于频道，多余的硬币被[锁定]（../ consensus / locking.md）。


＃＃＃＃ 要求

该交易必须具有所有相关方的有效签名。

该交易不得有争议，必须考虑任何持续的争议
通过此交易解决。

在此事务已包含在块中之后，通道必须是
被视为已结束，不允许进一步修改。

`频道总数==
  transcation initiator_amount_final + responder_amount_final + fee`


###`channel_close_solo`

为了单方面关闭频道，参与者必须发送一个
`channel_close_solo`交易。只有在一个对等体停止时才需要这样做
响应但也可以被试图关闭频道的恶意对等者使用
与所有参与者未达成一致意见的州。

在任何时候，频道参与者都可以启动独奏结束序列。
在发布`channel_close_solo`并将其包含在链a中之后
`lock_period`块高度计时器启动。
需要此锁定期才能让对方有机会提出异议
关闭序列所基于的状态。这可以通过
`channel_slash`和`channel_force_progress_tx`交易。

通过在链上包含此事务，通道进入“锁定”
状态，在此期间可以对`channel_close_solo`提出异议。

序列化定义[这里]（../ serializations.md＃channel-close-solo-transaction）

 - `channel_id`：在链上记录的频道ID
 - `from_id`：发布结账交易的渠道参与者
 - `payload`：空或证明包含证明是其中一部分的事务
  通道
 - `poi`：包含的证明
 - `ttl`：blockheight target，可以包含此事务
 - “费用”：交易费
 - `nonce`：取自`from_id`的帐号

包含证明代表了渠道的内部状态。至少
它必须包括所有帐户及其余额。它必须提供足够的
关闭频道的信息。矿工要检查其中的余额并使用
此数据用于更新频道的链上表示。这是怎么回事
海报启动独奏关闭序列。
如果渠道中有任何合同且有自己的余额，
它们没有在包含证明中提供，但它们更倾向于强制 - 
推进后续交易。参与者可以自行决定
想要发布它们。因此，账户中的累积余额
单独关闭交易可以低于持续链上的渠道余额。

`payload`可以是空的或签名的事务。


####清空有效负载

如果有效负载为空，则最后一个on-chain持久状态和`solo_round`为
用过的。在这种情况下，包含根哈希的证明必须等于一
坚持通道链。如果该国是单方面生产的，
即通过`channel_force_progress_tx`，制作
`频道（channel_id）.round！=频道（channel_id）.solo_round`，独奏结束
基于`solo_round`，因此仍然可以有争议。

 - `Channel（channel_id）.locked_until：= Block.height + Channel（channel_id）.lock_period`


####交易有效负载

如果有效载荷是一个事务，它必须是`channel_offchain_tx`。肯定是
共同签署。

Payload是一个有效的交易，具有：

*`state_hash`等于包含根哈希的证明。这是一个证明
  PoI是正确的
*`channel_id`与事务`channel_id`相同
*`round`大于`Channel（channel_id）.round`。如果
  `频道（channel_id）.solo_round>频道（channel_id）.round`，然后
  这种关闭将使链上产生的进展无效

如果为true，将进行以下更改：

 - `Channel（channel_id）.round：= payload.round`
 - `Channel（channel_id）.solo_round：= payload.round`
 - `Channel（channel_id）.state_hash：= payload.state_hash`
 - `Channel（channel_id）.locked_until：= Block.height + Channel（channel_id）.lock_period`


###`channel_settle`

结算交易是渠道生命周期中的最后一个，但是
只有当有关各方在尝试时无法合作时才需要
关闭频道。必须在所有可能的争议发布后发布
决定再重新分配锁定的硬币。

如果出现以下情况，`channel_settle`必须只包含在一个块中：

 - 发布了`channel_close_solo`交易并已过期，即
`blockheight（top） -  blockheight（channel_close_solo_tx）> = lock_period`
 - 没有公开争议，这意味着该频道目前不存在
  从先前的独奏行动中锁定

序列化定义[这里]（../ serializations.md＃channel-sett-transaction）

 - `channel_id`：在链上记录的频道ID
 - `from_id`：发布结算交易的渠道的参与者
 - `initiator_amount_final`：发起者从中获取的无符号金额
  渠道
 - `responder_amount_final`：响应者从中获得的无符号金额
  渠道
 - `ttl`：blockheight target，可以包含此事务
 - “费用”：交易费
 - `nonce`：取自`from_id`的帐号


＃＃＃＃ 要求

在此事务已包含在块中之后，通道必须是
被视为已结束，不允许进一步修改。

必须使用对应于公钥“from_id”的私钥来签署事务。

金额必须与最后提供的链上的金额相对应
`channel_close_solo`或`channel_slash`。形成最终金额的总和
频道的总收盘金额。如果这个总结算金额较低
比已经专用于频道的硬币总量，超出了
硬币是[锁定]（../ consensus / locking.md）。

##强制进步

强制进步是在发生争议时使用的机制
各方和其中一方想要使用区块链作为仲裁者。海报
提供脱链状态，以便可以执行脱链合同
在链上产生通道的下一个脱链状态。

这可能在通道关闭或仍处于活动状态时发生。如果
频道没有关闭，参与者可以继续从链上使用它
产生状态或开始关闭。如果频道已经关闭，那么
force-progress更新每个的当前预期收盘金额
参与者（根据合同的执行情况）。

力量进展基于被认为是最新的脱链
州。我们无法证明这个州实际上是最新的
通过提供一个共同签署的州，可以始终对链上的进展提出异议
轮次数高于有争议的轮数
使用`channel_force_progress_tx`事务。

如果通道处于关闭状态，则发出“channel_force_progress_tx”
将其锁定为`lock_period`块，在此期间可以对更新进行争议。这个
锁定是必要的，以防止通道立即关闭
在链上生产了新一轮。

值得一提的是，有争议的是脱链国家
部队的进展是基于而不是强迫进步本身。如果
强迫方提供了较旧的国家，另一方可以发布
一个较新的共同签署的脱链国家（例如通过快照）。共同签署
具有相同或更大轮次的状态将取代链上生产的状态。


###`channel_force_progress_tx`

序列化定义[此处]（../ serializations.md＃channel-solo-force-progress-transaction）

 - `channel_id`：在链上记录的频道ID
 - `from_id`：发布强制进度事务的频道的参与者
 - `payload`：空或证明状态树是其中一部分的事务
  渠道
 - `round`：频道的下一轮
 - `update`：通道离线更新，包含与天然气的合同调用
  链上执行所消耗的限价和汽油价格
 - `state_hash`：通道预期的脱链状态树的新根哈希值
 - `offchain_trees`：完整状态通道的状态树
 - `ttl`：blockheight target，可以包含此事务
 - “费用”：交易费
 - `nonce`：取自`from_id`的帐号

`offchain_trees`是完整的脱链状态树集：所有帐户和
所有合同。它必须包括参与者的脱链账户。
基于`offchain_trees`，将计算下一个状态及其状态
root hash将成为新状态的`state_hash`。新生产的州
树木将与提供的树木不同，至少与新添加的树木不同
`call`对象。因此，即使合同调用失败，新的`state_hash`也是如此
产生的。

如果在通道处于“关闭”状态时发送此事务，则会发生
将通道转换为下一个`lock_period`块的`locked`状态。

有效负载可以是空的或签名的事务。


####清空有效负载

如果有效负载为空，则最后一个on-chain持久状态和`solo_round`为
用过的。在这种情况下，状态树根哈希必须等于持久的哈希
对于通道链。

如果合同执行成功，通道将更新为：

 - `频道（channel_id）.solo_round：=频道（channel_id）.solo_round + 1`
 - `频道（channel_id）.state_hash：= state_hash`

此外，如果频道处于“关闭”状态：

 - `Channel（channel_id）.locked_until：= Block.height + Channel（channel_id）.lock_period`


####交易有效负载

如果有效载荷是一个事务，它必须是`channel_offchain_tx`。肯定是
共同签署。

如果离线事务有效负载具有以下条件，则它是有效的事务：

*`state_hash`等于状态树的根哈希。这是一个证明
  “offchain_trees”是参与者同意的事情
*`channel_id`与事务`channel_id`相同
*`round`大于`Channel（channel_id）.round`

更新必须是对合同的调用。此更新的调用者必须是
强制进度交易的海报。 `amount`，`gas`和`gas_price`都是
在更新中也指定了。燃气费将由海报支付
交易。
state_hash将是更新的通道的状态树的根哈希。后
将合同调用应用于提供的`offchain_trees`并更新帐户
因此，产生了新通道的状态树。必须有相同的
root哈希的值作为状态哈希。如果那些与力量进展不匹配
失败，但因为这只能在执行调用后确定，a
呼叫对象被添加到链上并且消耗了气体。

如果合同调用成功，则将更新通道状态：

 - `Channel（channel_id）.round：= payload.round`
 - `Channel（channel_id）.solo_round：= payload.round + 1`
 - `频道（channel_id）.state_hash：= state_hash`

此外，如果频道处于“关闭”状态：

 - `Channel（channel_id）.locked_until：= Block.height + Channel（channel_id）.lock_period`


####强制进步副作用

#####更新频道对象

根据`offchain_trees`来重建通道状态树
提供。此更新是一项离线合同调用。它适用于
通道的状态树并修改它们。修改后的树有根
哈希值。有可能：

 - 等于强制进度事务中提供的`state_hash`。
  这个哈希确实是合同调用的预期结果
  区块链已经证实了这一点。链上通道对象已更新
  因此：
   - 通道的状态哈希被更新为新计算的哈希
   - 频道轮次是强制进度交易中的一个
   - 如果该频道处于关闭状态，则结束余额
    参与者根据修改的频道状态中的参与者进行更新
    树木
 - 不等于强制进度交易中提供的`state_hash`。该
  提供的哈希未被确认为预期的哈希。在这种情况下的力量
  进度失败，并且没有创建新的状态通道状态。链上频道
  对象未被修改，因此 - 存储的`round`和`state_hash`
  链上保持不变。气体仍然消耗，并且创建了一个调用对象
  上链。

一个特殊情况是提供无效更新调用的forcer
被迫的。无效更新调用的示例如下：

*远程调用丢失的合同
*在通话中花费太多硬币，以便参与者的链外平衡
  低于`channel_reserve`阈值
*由于'out_of_gas`异常而终止合同通知

在这种情况下，合同无法执行和强制进展
未能产生新的状态。最终结果与在那里完全相同
与生成的`state_hash`和预期的`state_hash`不匹配。


#####调用对象

如果`channel_force_progress_tx`有效，则在`update`中调用契约
在由offchain_trees制作的MPT上执行。
输出是一个新的MPT，代表新的脱链通道状态。
参与者要么继续使用频道，要么关闭频道。如果有
没有后来的脱链更新，他们预计会在两者中使用这个产生的MPT
案例。

合同执行消耗天然气。 `update`本身定义了两种气体
限价和汽油价格。合同调用已经执行并且真实
燃气消耗量已计算，账户余额已过账
交易更新以支付燃气费。由于这不是共同签署
交易而非单方面交易的发起者
执法支付费用。

契约调用在链上状态下生成链上一个新的调用对象
合同电话的树木。通常，调用具有由该组成的键
合同的地址，来电者的地址和来电者的随机数。自从关闭
连锁合约不是持久链，它没有可以的地址
以这种方式使用。强制进步产生的呼叫使用
相反，`channel_force_progress_tx`的哈希。

由于矿工正在为合同的执行消耗资源，天然气
无论如何，支付费用并为每个部队进度创建呼叫对象
如果成功更新了链上通道对象。


##争议更新

争议应被视为异常，只有在一方当事人才会发生
试图单方面发布一个过时的国家，同时：

 - 单方面关闭渠道
 - 强迫进步
 - 或削减

并且可以通过`channel_slash`，`channel_force_progress_tx`进行争议，
`channel_snapshot_solo`交易。

由于争议本身可能受到挑战，我们最终可能陷入困境，
恶意方通过`channel_force_progress_tx`迅速进展的地方
交易，剥夺了另一方的争议能力。阻止
这种情况我们可以：

1.强制执行每个争议，以便始终等待`lock_period`到期
   并且每个时期只有一个争议
2.或不限制连续争议的数量但总是可以选择
   挑战任何争议链的第一个要素，使全部无效
   链。

我们选择使用第二种策略，因为它允许更快的进展
在保证安全的同时消失的同伴的情况。这使得
跟踪正确的状态更复杂，但我们假设一个同伴
消失的可能性高于他们主动恶意的可能性。
因此，我们尝试针对该案例进行优化。


###强制进度争议与结束渠道

如果通道处于关闭状态，只能通过a
`channel_close_solo`，然后每个争议触发频道的扩展
由`lock_period`块锁定。如果频道被锁定，则遵循与上述相同的规则
应用。也就是说，尽可能多的`channel_force_progress_tx`或`channel_slash`
可以提交所需的交易 - 每个交易都延长了锁定 - 但同样长
当通道被锁定时，如果状态为a，则整个链可以无效
可以发布更高的回合数。

这里需要锁定，因为否则恶意方可能会发布
包含过时状态的`channel_force_progress_tx`或`channel_slash`
然后立即尝试根据结果确定渠道。


###用开放频道示例强制进度争议

建立：

 -  Bob和Alice用`lock_period：= 100`打开彼此之间的通道
 - 在`chain_height：= 1000`Bob用`发布`channel_force_progress_tx`
  有效载荷包含`round：= 23`并在链上产生`round：= 24`
 - 通道状态现在将包含23作为最新的'round`和24作为
  `solo_round`

使用所选策略，Bob不必等待`lock_period`
到期并且可以根据需要发布尽可能多的`channel_force_progress_tx`，例如在
`chain_height：= 1001`他生成`solo_round：= 25`并且通过`chain_height：= 1011`
到达`solo_round：= 31`。 `round`仍然是23。

现在，如果Alice在`chain_height：= 1110`返回，她仍然可以提出异议
*初始*更新由Bob在`chain_height：= 1000`发布，提供了一个
通过`channel_force_progress_tx`与`round：= 24`或更高版本共同签署有效载荷
或者是`channel_snapshot_solo`。


###强制进度争议与结束渠道示例

建立：

 -  Bob和Alice用`lock_period：= 100`打开彼此之间的通道
 - 在`chain_height：= 1000`Bob用`发布`channel_close_solo`
  包含`round：= 23`的有效负载。该频道现已锁定，直到
  `chain_height == 1100`

根据所选策略，Bob必须等待`lock_period`到期，如果
他想要解决这个频道。但是当频道被锁定时，他仍然可以
根据需要发布尽可能多的`channel_force_progress_tx`，例如在
`chain_height：= 1001`他生成`solo_round：= 24`并且通过`chain_height：= 1011`
到达`solo_round：= 31`。每次后续的操作都会发生颠簸
`lock_period`提前。也就是说，通过`chain_height == 1011`设置`lock_period`
跑到`chain_height == 1111`。

现在重要的是要注意，即使在`chain_height：= 1110`，爱丽丝也可以
仍然对Bob在`chain_height：= 1000`发布的* initial *更新提出异议
提供带有'round：= 24'或更高的共同签名的有效载荷，但她的争议会
也可以通过`lock_period`来锁定锁。


###`channel_slash`

如果恶意方发送了`channel_close_solo`或`channel_force_progress_tx`
在一个过时的国家，诚实的党有机会发出一个
`channel_slash`交易。该交易必须包括一个更高的州
'round`号码比争议者多，由所有同行签名成功
挑战。

序列化定义[这里]（../ serializations.md＃channel-slash-transaction）


 - `channel_id`：在链上记录的频道ID
 - `from_id`：发布斜杠事务的频道参与者或代表
 - `payload`：证明包含证明是其中一部分的事务
  渠道
 - `poi`：包含的证明
 - `ttl`：blockheight target，可以包含此事务
 - “费用”：交易费
 - `nonce`：取自`from_id`的帐号

包含证明代表了渠道的内部状态。它必须
包括参与者的帐户和余额。
如果渠道中有任何合同且有自己的余额，
它们没有提供包含证明，但它们更像是武力
推进后续交易。参与者可以自行决定
想要发布它们。因此，账户中的累积余额
斜杠交易可以低于持续链上的渠道余额。

有效负载可以是空的或签名的事务。


####清空有效负载

如果有效负载为空，则最后一个on-chain持久状态和`solo_round`为
用过的。在这种情况下，包含根哈希的证明必须等于一
坚持通道链。如果该国是单方面生产的，
即通过`channel_force_progress_tx`，制作
`频道（channel_id）.round！=频道（channel_id）.solo_round`，斜线是
基于`solo_round`，因此仍然可以有争议。

 - `Channel（channel_id）.locked_until：= Block.height + Channel（channel_id）.lock_period`


#### Transaction payload

If the payload is a transaction it MUST be a `channel_offchain_tx`. It MUST be
co-signed.

Payload is a valid transaction that has:

* `state_hash` equal to the proof of inclusion's root hash. This is a proof
  that the PoI is correct
* `channel_id` being the same as the transaction `channel_id`
* `round` greater than `Channel(channel_id).round`.如果
  `Channel(channel_id).solo_round > Channel(channel_id).round`, then
  this slash will invalidate progress produced on-chain

If true, the following changes will be made:

- `Channel(channel_id).round := payload.round`
- `Channel(channel_id).solo_round := payload.round`
- `Channel(channel_id).state_hash := payload.state_hash`
- `Channel(channel_id).locked_until := Block.height + Channel(channel_id).lock_period`


#### Requirements

MUST be signed using private key corresponding to the public key `from_id`.


## Channel state tree

Each block MUST commit to a Merkle Patricia tree of open channels, where the
`channel_id` specifies the path.
At a leaf, nodes store information pertaining to the current state of the given
channel.

- `channel_id`
- `initiator_id`
- `responder_id`
- `delegator_ids`
- `total_amount`
- `initiator_amount`
- `responder_amount`
- `channel_reserve`
- `state_hash`: last published state_hash
- `round`: last known co-signed round
- `solo_round`: last round produced via a `channel_force_progress_tx`
- `lock_period`: agreed upon locking period by peers
- `locked_until`: on-chain channel height after which the channel can be settled

Keeping track of the `state_hash`, `round`, `locked_until`, and `lock_period` is
necessary for nodes to be able to assess the validity of `channel_slash` and
`channel_settle` transactions.

The `locked_until` is initialised with `0` and will stay `0` until the channel
enters the `closing` state.

Serialization defined [here](../serializations.md#channel)


