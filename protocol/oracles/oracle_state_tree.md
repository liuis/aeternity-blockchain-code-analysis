## Oracle状态树

oracle状态树包含oracle对象和oracle查询对象。 当双方相互契约时，必须证明存在oracle和查询/响应的存在。 oracle和查询存储在同一棵树中; 其中查询的id是oracle id和查询id的串联。 因此，查询实际上形成了oracle的子树，可以迭代，等等。

### Oracle状态树对象

 -  oracle状态树包含
   -  Oracle对象
   - 查询对象

#### oracle对象

 - 由oracle注册事务创建。
 - 当TTL过期时删除。
 - 更新了oracle扩展事务的新（更长）TTL。

```
{owner :: pubkey（）
，query_format :: type_spec（）
，response_format :: type_spec（）
，query_fee :: amount（）
，expires :: block_height（）
}
```

#### oracle查询对象

 - 由oracle查询事务创建。
 - 由oracle响应事务关闭。
 - 一旦关闭就不可变。
 - 过期时删除（当查询到期时打开查询，以及何时过期）
关闭查询的响应到期）

在创建时通过查询TTL和响应确定到期
oracle查询事务中的TTL。如果/当oracle响应事务是
在链上接受，到期根据响应TTL更新。
*注意：*如果查询的最大TTL（查询TTL +响应TTL）更长
比Oracle的TTL，然后查询被拒绝*。

```
{query_id :: id（）
，oracle_address :: pubkey（）
，query :: query（）
，response :: oracle_response（）
，expires :: block_height（）
，response_ttl :: relative_ttl（）
，fee::integer（）
}
```

### Oracle状态树更新

当到达对象的TTL时，将修剪oracle状态树链的高度。我们将操作顺序定义为：

1.删​​除过期的对象。应按ID的升序删除对象。
2.在块的事务顺序中插入新对象。

请注意，ID的排序顺序与树的顺序遍历相同。

### 处理对象的TTL

我们将保留按TTL和ID排序的对象缓存。这样的缓存有下列好处：
 - 要删除的下一个对象是缓存中的第一个对象。
   -  TTL低于块高度 - >我们完成了。
   -  TTL等于块高度 - >删除对象。
 - 可以通过执行有序遍历来重建缓存oracle状态树。

### 修剪oracle查询对象


如果oracle查询没有给出响应，则查询的poster应该退还oracle查询费用。 如果oracle已经做出回应，那么oracle已经在响应时获得了资金。

