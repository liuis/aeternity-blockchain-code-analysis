[返回]（./ oracles.md）
## Oracle事务

Oracle事务有四种类型：
 - 注册
 - 延长
 - 查询
 - 回应

### Oracle注册事务

oracle运营商可以将现有帐户注册为oracle。

该交易包含：
 - 应注册为oracle（oracle_owner）+ nonce的地址
 - 查询格式定义
 - 响应格式定义
 - 查询费用（应该为向oracle发布查询而支付）。
 -  TTL（块数或绝对块高度的相对值）
 -  Vm版本（参见[VM描述]（../ contracts / contract_vms.md））。
- 手续费。

请参见[序列化规范]（/ serializations.md #oracle-register-transaction）。

＃＃＃＃ 去做

 - 将来我们可以想象一个oracle注册事务
  通过使用源对请求进行双重签名来创建新帐户
  帐户和新帐户。

### Oracle扩展事务

oracle运算符可以扩展现有oracle的TTL。

该交易包含：
 - 应该扩展的地址/ oracle（以及一个nonce）
 -  TTL的扩展（相对于当前块数的到期）
- 手续费。

请参阅[序列化规范]（/ serializations.md #oracle-extend-transaction）。

### Oracle查询事务

 - 包含：
   - 发件人（地址）+ nonce
   - 神谕（地址）
   - 二进制格式的查询
   - 查询费用 - 锁定，直到：
     -  oracle回答并收取费用
     -  TTL到期，发件人获得退款
   - 查询TTL
   - 响应TTL
   - 交易费。

该事务在oracle中创建一个oracle交互对象
州树。此对象的id由查询构造
transaction作为{sender_address，nonce，oracle_address}的哈希值

查询TTL决定查询打开多长时间来响应
神谕。

查询TTL可以是绝对的（块高）或相对的
（也在块高）到包含查询的块。

响应TTL决定给定时响应的可用时间
来自神谕。响应TTL始终是相对的。这不是
自从神谕以来，给oracle提供了一些激励，以便及时发布答案
正在支付回复费用。

请参阅[序列化规范]（/ serializations.md #oracle-query-transaction）。

####问题/稍后

 - 我们可以包括oracle可以回答的最早时间
防止恶意的oracles及早回答并收取费用。

### Oracle响应

oracle运算符通过发布oracle响应来响应查询
事务，使用oracle帐户的私钥对其进行签名。

如果查询中的TTL具有，则响应事务无效
过期。

oracle支付回复交易的费用。最低费用
由查询的响应TTL和大小决定
响应。

请注意，有动力保持响应的准确性（和
小）因为oracle支付响应交易。

该交易包含
 -  oracle（地址）+ nonce
 -  oracle交互ID（从查询派生）
 - 二进制格式的响应
 - 响应TTL（冗余，但使事务自包含）
 - 交易费。

请参阅[序列化规范]（/ serializations.md #oracle-response-transaction）。

####问题/稍后

 - 我们应该在查询中定义自动回调吗？
   - 任何回调都由oracle支付。
 - 如果有一些神谕，我们是否应该能够返还部分费用
  理由无法提供答案。
