[back](../SYNC.md)

# Mempool/TX-pool synchronization

On startup the Aeternity node needs to update (or get from scratch if it is a
completely new node) its list of unconfirmed transactions. New transactions it
will get through gossip, but by construction older transactions that (for some
reason) have not made it onto the chain is not gossiped.

Potentially there is a big overlap between the list of transactions held by the
starting node (transactions are persisted) and the peer it is synchronizing
with; thus to save network bandwidth it is preferrable to only send the missing
transactions. The protocol is straightforward:
  1. the node picks one peer to synchronize with
  2. they each create a MP tree holding their local transactions - In the MP
     tree, each element has as key the hash of the signed transaction and as
     (placeholder) value the empty list.
  3. the initiator asks for a partial unfolding of the peer's tree
  4. the initiator computes what parts of the tree it doesn't know about and
     either goes back to 3. and asks for further information, or
  5. the initiator asks for the missing transactions.

The P2P messages used are listed in [p2p_messages](./p2p_messages.md).

在启动时，Aeternity节点需要更新（或者从头开始，如果它是
全新的节点）其未经证实的交易清单。它的新交易
将通过gossip，但通过建设旧的交易（对于一些人
原因）没有把它放到链条上没有gossiped。

可能存在的交易清单之间存在很大的重叠
起始节点（事务是持久的）和正在同步的对等体
用;因此，为了节省网络带宽，优选仅发送丢失的网络带宽
交易。协议很简单：

1.节点选择一个peer进行同步

2.他们每人创建一个MP树，持有他们的本地交易 - 在MP
树，每个元素都具有签名事务的哈希值和密钥
（占位符）值空列表。

3.发起者要求peer's tree的部分展开

4.启动器计算它不知道的树的哪些部分和
要么回到3.并要求进一步的信息，或

5.发起人要求丢失交易。

使用的P2P消息列在p2p_messages中。

## Unfold serialization

The instructions for unfolding an MP tree are sent over the P2P protocol, thus
they need to be serialized. There are four different messages sent, they are
listed below. Except for the types mentioned in
[p2p_messages](./p2p_messages.md) we also use the type `path` which is path to
a node in an MP-tree, thus it could be an even or odd number of _nibbles_. RLP
require an even number of nibbles, i.e. whole bytes, so we pad with a parity:

  - even number of nibbles add byte with 0
  - odd number of nibbles add a nibble with value 1

The result is an even number of nibbles and the first nibble indicate how it
should be deserialized.

用于展开MP树的指令是通过P2P协议发送的，因此需要对它们进行序列化。 发送了四种不同的消息，它们列在下面。 除了p2p_messages中提到的类型之外，我们还使用类型 path，它是MP-tree中节点的path，因此它可以是偶数或奇数个半字节。 RLP需要偶数个半字节，即整个字节，所以我们填充奇偶校验：

- 偶数个半字节用0添加字节
- 奇数个半字节添加一个值为1的半字节


结果是偶数个半字节，第一个半字节表示它应该如何反序列化。

### UNFOLD_NODE
Message is RLP encoded, fields:
  - `type :: int` = 0
  - `path :: path`
  - `node :: byte_array`

### LEAF_NODE
Message is RLP encoded, fields:
  - `type :: int` = 1
  - `leaf :: path`

### SUBTREE_NODE
Message is RLP encoded, fields:
  - `type :: int` = 2
  - `subtree :: path`

### KEY_NODE
Message is RLP encoded, fields:
  - `type :: int` = 3
  - `key  :: byte_array`


