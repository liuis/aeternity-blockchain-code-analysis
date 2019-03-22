## Oracle生命周期示例

### 足球比赛结果的 oracle

爱丽丝是足球界的忠实粉丝，她想分享比赛的结果。

Alice通过向链提交oracle注册事务来注册oracle。

Alice为提交交易支付了费用。

Alice需要在事务中包含一些信息
 - 支付费用的查询费。
 - 查询格式的声明。
 - 响应格式的声明。

当oracle事务包含在链中时，oracle将在oracle状态树中创建。

Alice正在网络中运行一个节点。 Alice在她的本地节点中注册订阅，因此当有人向她的oracle发布查询时，她会收到通知。

Bob希望在块链上使用特定游戏的结果。

Bob在链上发现了Alice的oracle，并向链发布了一个查询事务。

Bob为提交交易支付了费用。 费用由矿工收取。

Bob向oracle发送查询支付费用。 费用转移到oracle的账户。

查询事务包含：
 - 地址或爱丽丝的oracle。
 - 以Alice的oracle声明的格式的查询。
 - 矿工的交易费。
 - 将coin转移到爱丽丝oracle。
 - 查询的TTL（如果Alice的oracle在给定时间内没有回答查询，Bob退还coin）

 - 查询事务在oracle状态树中创建oracle交互对象。 oracle交互对象的id是从查询事务派生的。


   Bob在节点上注册订阅（使用oracle交互ID），以便在对其查询进行响应时收到通知。

   当Bob的查询事务被接受到链中时，Alice的事件订阅触发，并且她被告知链上有新的查询。

   该活动包含：

   - 发件人帐户的地址，

   - oracle交互的ID。

   当比赛已经播放并且有结果时，Alice在链上发布了oracle响应事务。

   响应事务包含：

- oracle交互对象的Id。

- oracle定义中定义的结果。

   Alice为发布交易支付交易费。费用由矿工收取。作为交易的一部分，Alice的oracle收到查询费用。

   当响应事务已被链接受时，Bob的事件订阅会通知他存在响应。

   该事件包含：

- oracle交互对象的Id。

-  回复的内容。

-  响应的TTL。

 Bob现在可以通过以下方式使用响应：

- 执行引用oracle交互对象的智能合约。
- 使用节点的公共API来获得答案。
