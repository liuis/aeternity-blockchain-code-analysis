# Gossip-中文

As explained in the [Sync](SYNC.md) document, any blockchain implementation
needs communication with peers in order to be truly useful. Apart from the lower
level details described in that document, there is the added dimension of
how peers are selected for communication, how connections to those peers are
maintained over time. This is the purpose of gossiping functionality.

如同Sync文档中所述，任何区块链实现
需要与peers沟通才能真正有用。 除了下层
该文档中描述的级别详细信息，增加了维度
如何选择对等体进行通信，以及如何与这些peers建立连接
保持一段时间。 这是gossipig功能的目的。

## Initial Configuration 初始化配置

The initial set of peers is defined by configuration when starting the node.
This is a predefined set of peers that are automatically connected to upon
startup.

初始peers集由启动节点时的配置定义。
这是一组自动连接的预定义的peers启动。

## Peers

A peer is identified by a URI consisting of the protocol 'aenode://', the public
key, an '@' character, the hostname or IP number, a ':' character and the Noise
port number the node is using. If the address contains a hostname, it will be
resolved to an IP. Example:

```
aenode://pp_HryRGHJ7Ct3trkktVyVBgfhHL1J4EYSD9cScuMZDV61eSHrCZ@mynode.example.com:3015
```

A node is uniquely identified by its public key, and it can only have one IP and
port at the same time. This means one IP can have several instances of Aeternity node
started at the same time listening on different ports if they have different
public keys. This key corresponds to the private key that is used for the Noise
protocol listener associated with the IP and port.

Apart from being used in encryption via the Noise protocol, the public key is
also used to determine which node should keep its initiated connection open in
case two connections are opened (see
[Gossiping of New Peers](#gossiping-of-new-peers)).

节点由其公钥唯一标识，并且它只能同时具有一个IP和端口。 这意味着一个IP可以有多个Aeternity节点实例，如果它们具有不同的公钥，则同时在不同的端口上进行监听。 此密钥对应于用于与IP和端口关联的Noise协议侦听器的私钥。

除了通过Noise协议用于加密之外，公钥还用于确定在打开两个连接的情况下哪个节点应保持其启动的连接打开。

### Gossiping of New Peers

Whenever a ping message is exchanged between peers, either a ping request or
ping response, the peers also attach a subset of neighboring peers they know of.
The node that generates the message populates the neighbor list with 30 peers
that are randomly selected from its pools (see
[Peers Maintenance](#peers-maintenance)).

每当在peers之间交换ping消息时，无论是ping请求还是
ping响应，peers还附加了他们知道的邻居peers的子集。
生成消息的节点使用30个peers填充邻居列表
从池中随机选择。

When a ping request is received from another peer, that peer is first checked
for acceptance. This includes verifying

1. It is not the local node itself
2. The peer is not on the blocked list
3. It doesn't already have an existing connection

(3) refers to the case where two peers have learned about each other separately
and are both trying to initiate a connection. The connections are both initiated
via Noise, but once they are connected each node checks if they already have a
connection to that same peer. If they do, they will keep it if their public key
is larger than their peers, and drop it if not. This ordering of keys is
arbitrary but achieves the effect of only ever being true at one node or the
other. Thus, only one connection is kept.

Once the peer is accepted, all the neighbors not already verified are added to
the unverified pool. If this is the first ping from an inbound connection from
an unverified peer, the peer itself is added to the unverified pool too
(see [Peers Maintenance](#peers-maintenance) for more details).

当从另一个对等体收到ping请求时，为了接受这个请求需要进行一系列检查， 这包括验证

1.它本身不是本地节点
2.peers不在阻止列表中
3.它还没有existing connection

（3）指的是两个对等体分别相互学习并且都试图发起连接的情况。 连接都是通过Noise启动的，但是一旦连接它们，每个节点都会检查它们是否已经连接到同一个对等体。 如果他们这样做，如果他们的公钥比他们的同行大，他们将保留它，如果没有，则放弃它。 密钥的这种排序是任意的，但实现了在一个节点或另一个节点上永远存在的效果。 因此，仅保留一个连接。

一旦接受了对等体，所有尚未验证的邻居都将被添加到未验证的池中。 如果这是来自未验证对等体的入站连接的第一次ping，则对等体本身也会添加到未验证的池中。



### Peers Maintenance

The objectives of peer maintenance are:

1. Prevent peer poisoning (Sybil/eclipse attack)
2. Limit the number of active peer connections
3. Cache the known peers between restart to make peer poisoning harder.

To prevent an attacker from poisoning the list of peers, isolating the
node from the rest of the network, we make it impossible to predict
the set of peer's IP/port the attacker should control to fill enough of the list
for the attack to be statistically successful. In addition, we randomize the
peer eviction to prevent an attacker from predicting it and replace all good
peers for their own compromised ones.

This is achieved by first categorizing peers in two groups:

1. The unverified pool contains all the peers received through gossip.
It ensures no Byzantine node can fill it completely by limiting the subset
of the pool it can affect; it prevents attackers from predicting the set of
IP/port they must use to successfully eclipse the node.

2. The verified pool contains the peers the node connected to explicitly after
passing through the filter of the unverified pool. It randomizes the peer
distribution to the eye of attackers, preventing them from predicting the
eviction algorithm.

Peers are considered verified when the node has been able to connect to
them using the Noise protocol.

Both groups clusterize peers into multiple buckets; they select which one
an added peer belongs to based on a combination of the address of the node the
information is coming from, the address of the peer to be added and a secret
generated by the node for the purpose of randomizing the selection process.
See [Unverified Pool](#unverified-pool) and [Verified Pool](#verified-pool) for
more details on how the peers are added to the pools.

The peers received from configuration are added directly to the verified pool
and are marked as "trusted", meaning that they will never get downgraded to the
unverified pool even if the node cannot connect to them.

### Peer Groups

Peers are grouped by the 16 most significant bits of a peer IP (\16 mask).
This group is used when selecting a bucket and establishing the node outbound
connections to prevent nodes from being connected only to local nodes; this
make block forwarding more effective. NOTE: If all the nodes are in the same
group, they may not reach their maximum number of outbound connection and this
should probably be disabled.

1. 防止peer 攻击（Sybil / eclipse攻击）

2. 限制活动对等连接的数量

3. 在重启之间缓存已知对等体以使peer攻击更难。

为了防止攻击者中毒对等体列表，将节点与网络的其余部分隔离，我们无法预测攻击者应控制的对等体IP /端口集，以便在统计上填充足够的攻击列表成功的。此外，我们将对等驱逐随机化，以防止攻击者预测它并替换所有好的同伴以替换他们自己的被攻击者。

这是通过首先对两组中的同伴进行分类来实现的：

未验证的池包含通过八卦收到的所有对等体。它确保没有拜占庭节点可以通过限制它可能影响的池的子集来完全填充它;它可以防止攻击者预测他们必须使用的IP /端口集来成功地消除节点。

经验证的池包含通过未验证池的过滤器后显式连接的节点的对等体。它将对等分布随机化到攻击者眼中，阻止他们预测驱逐算法。

当节点能够使用Noise协议连接到它们时，认为对等方已经过验证。

两个组都将对等体聚类为多个桶;它们基于信息来自的节点的地址，要添加的对等体的地址以及由节点生成的秘密来选择添加的对等体属于哪一个，以便随机化选择过程。有关如何将对等方添加到池中的更多详细信息，请参阅未验证池和已验证池。

从配置接收的对等体直接添加到已验证的池中并标记为“可信”，这意味着即使节点无法连接到它们，它们也永远不会降级到未验证的池。

### Peer Groups

对等体按对等IP的16个最高有效位（\ 16掩码）分组。选择存储桶并建立节点出站连接时，将使用该组，以防止节点仅连接到本地节点;这使得块转发更有效。注意：如果所有节点都在同一组中，则它们可能无法达到其最大出站连接数，并且可能应禁用此节点。

### Connections

There are two pools of connections, inbound connection and outbound connections.

Both inbound and outbound connections are used for mempool, gossip and sync
protocol; but only outbound connections are used for relaying new blocks to
reduce the network load.

There is a hard limit on the maximum number of outbound connection
(default to 10).

There is a soft limit on the maximum of inbound connections (default to 100).
When it is reached, the node will still accept inbound connections, but they will
be closed right after responding to the first ping. This is a soft limit to allow
nodes to join the cluster even if the trusted nodes inbound limit has been
reached. In addition, this prevent a node, in particular a trusted node, to
have so much inbound connections that it cannot reach its number of outbound
connections; a guaranteed number of outbound connection is crucial for proper
propagation of new blocks. For all nodes to reach their established number of
outbound connections, the soft maximum limit for inbound connections must be
smaller than the maximum number of nodes minus the wanted number of outgoing
connections.

The node will first burst-connect to the trusted peers and then periodically
connects to more peers until it reaches the maximum number of outbound
connections; the delay between connections to new peers should be small enough
so the nodes has enough outbound connections to work properly, but large enough
so it has enough peers to choose from. This is done by having a variable delay
between connections of `2^(OUTBOUND_CONNECTION_COUNT - 1)` seconds when
`OUTBOUND_CONNECTION_COUNT` is greater than `0` with a maximum of `30` seconds;
this allow the node to reach 5 outbound connection under `15` seconds and
connect to the last peer when having received around `20` gossip messages.

The node iteratively picks a random peer (not yet connected) from either
the verified pool or the unverified pool (`0.5` probability by default),
that is from a different group ([See Peer Groups](#peer-groups)) than
any actual outbound connection. In case there are no more peers available in the
selected pool the other one will be tried. If the connection succeeds, an
unverified peer is upgraded to the verified pool and removed from the unverified
pool. If the connection fails, the peer retry counter and last retry time are
updated.

The node will periodically check a random peer from the pool without sending
a ping message, only establishing the Noise connection. This ensure the pool
contains enough reachable peers to reduce the chance of a Sybil attack were
most of the good peers are unreachable augmenting the probability of only
hostile node getting selected. (Not yet implemented).

If a peer changes its IP address but not its key, the new address will **never**
be updated through gossip. This is to prevent an hostil node from gossiping
bad addresses for known good nodes and making them unreachable. The peer will be
removed after the normal retry policy is exausted; then the new address will be
added through the usual gossip exchanges.

The peer retry counter and last retry time (initialized to '0' and 'infinity'),
are used to filter out peers when picking them from the pools providing
exponential backoff. After N failed attempts, a verified peer that is not
marked as trusted is downgraded to the unverified pool and the counter and time
is reset; unverified peers are simply removed.

Connected peers will periodically be pinged, and their connection state
monitored. The ping interval is configurable and defaults to once every 120
seconds. If a ping fails, this is logged and the peer is scheduled for another
ping in the normal interval.



有两个连接池，入站连接和出站连接。

入站和出站连接都用于mempool，八卦和同步协议;但只有出站连接用于中继新块
减少网络负载。

出站连接的最大数量有一个硬限制（默认为10）。

入站连接的最大值有一个软限制（默认为100）。
到达时，节点仍将接受入站连接，但它们将在响应第一次ping后立即关闭。即使已达到受信任节点的入站限制，这也是允许节点加入群集的软限制。此外，这可以防止节点（特别是受信任节点）拥有如此多的入站连接，使其无法达到其出站连接数;保证的出站连接数对于正确传播新块至关重要。对于所有节点达到其已建立的数量
出站连接时，入站连接的软最大限制必须小于最大节点数减去所需的传出连接数。

该节点将首先突发连接到可信对等体，然后定期连接到更多对等体，直到达到最大出站连接数;连接到新对等体之间的延迟应该足够小，以便节点有足够的出站连接才能正常工作，但足够大
所以它有足够的同行可供选择。这是通过在2 ^（OUTBOUND_CONNECTION_COUNT  -  1）秒的连接之间具有可变延迟来实现的
OUTBOUND_CONNECTION_COUNT大于0，最多30秒;
这允许节点在15秒内达到5个出站连接，并在收到大约20个八卦消息时连接到最后一个对等点。

该节点迭代地从已验证的池或未验证的池（默认为0.5概率）中挑选一个随机对等体（尚未连接），该对等体来自不同的组（参见对等组）。
任何实际的出站连接。如果所选池中没有更多可用对等体，则将尝试另一个对等体。如果连接成功，则未验证的对等方将升级到已验证的池，并从未验证的池中删除。如果连接失败，则更新对等重试计数器和最后重试时间。

节点将定期检查池中的随机对等体，而不发送ping消息，仅建立Noise连接。这确保了池包含足够的可达对等体，以减少Sybil攻击的可能性，因为大多数好的对等体都无法访问，从而增加了仅选择敌对节点的概率。 （尚未实现）。

如果对等体更改其IP地址而不更改其密钥，则新地址永远不会通过八卦更新。这是为了防止hostil节点为已知的良好节点闲置坏地址并使它们无法访问。在正常重试策略执行后，对等体将被删除;然后新的地址将是
通过通常的八卦交流加入。

对等重试计数器和最后重试时间（初始化为'0'和'无穷大'）用于在从提供指数退避的池中选择对等体时过滤掉对等体。尝试N次失败后，未标记为受信任的已验证对等体将降级为未验证池，并重置计数器和时间;未经验证的同伴只是被删除。

将定期ping通连接的对等端，并监视其连接状态。 ping间隔是可配置的，默认为每120秒一次。如果ping失败，则会记录此信息，并且会在正常时间间隔内为对等方安排另一次ping操作。

#### Connection Cleanup

Connections are monitored and cleaned when inactive to prevent an attacker
from isolating the node by blocking chain traffic. If there is actually no
traffic, it will just accelerate the connection rotation.

- All connections without any chain-related activity (not counting gossip) for
more than 180 seconds will get disconnected. (Not yet implemented)
- All inbound connections that don't send a ping after 30 seconds will be
disconnected.
- All recent inbound connections (less than 90 seconds old) without any
chain-related activity (not counting gossip) will be closed. (Not yet implemented)
- When the maximum number of outbound connections has been reached, a random
peer is checked every minute (only Noise handcheck).

#### Unverified Peers

The unverified pool is composed of 1024 buckets of up to 64 peers, resulting in
a maximum capacity of 65536 peers.

To prevent a rogue node to fill it with compromised peers, it uses the address
group of the node gossiping the peers to select a subset of the buckets; then
the added peer address group is used to select a smaller subset and then the
rest of the address is finally used to select the bucket it belongs to.
A secret known only by the node is used to randomize the selection process
so it cannot be predicted by the attacker.

If a peer is not already in the verified pool, the steps to add it to the
unverified pool are:

1. If there is already 8 references of the peer in the pool, no more
references are added; if there is N references, a random probability check of
'1/2^N' is performed to decide if another reference to the peer should be added.
2. A subset of 64 buckets are selected based on the secret and the group of the
node that gossiped the peer (IP '/16' mask); this limits the part of the pool
that can be changed by any given rogue node IP.
3. From this subset, 4 buckets are selected based on the secret and the IP of
the peer to be added; this randomizes the peer distribution preventing the
prediction of the way peers will be evicted.
4. From these 4 buckets, a single one is selected randomly; this reduces the
collisions between peers that share the same IP.
5. If the bucket already contains a reference to the peer, it is not added again.
6. If the bucket the peer has to be added to is full, one existing peer has to
be evicted. This is done by first cleaning all the peers that weren't gossiped
for a certain amount of time, then if the bucket is still full, by selecting a
random peer with a bias favoring peers that were added the longest time ago.
7. The eventually evicted peer is completely removed from the pool.

Only the IP of the peers are used to select a subset of the pool because
IP is an expensive resource while ports are comparatively cheap.

See [Bucket Selection](#bucket-selection) for more details on how the buckets
are selected from the secret and other discriminators.

#### Verified Peers

The verified pool is composed of 256 buckets of up to 32 peers, resulting in a
maximum capacity of 8192 peers.

To prevent attackers from predicting how good peers are evicted and replace them
by compromised peers, the eviction is done per-bucket; the buckets are selected
from the peer address and a secret to randomize the selection process.

When a peer is verified the first time by connecting to it using the Noise
protocol, it is removed from the unverified pool and the steps to add it to
the verified pool are:

1. A subset of 8 buckets are selected based on the secret and the address group
of the peer to be added.
2. From this subset, a single bucket is selected based on the secret and the
rest of the IP of the peer to be added.
3. If the bucket is full, one peer has to be evicted. This is done by first
cleaning the peers that weren't gossiped for a long time, and if the bucket is
still full, by selecting a random one with a bias toward the ones that are not
connected and the last connection was the longest time ago. Trusted peers and
connected peers are never evicted.
4. The eventually evicted peer is added to the unverified pool; its retry
counter and last retry time are reset.

See [Bucket Selection](#bucket-selection) for more details on how the buckets
are selected from the secret and other discriminators.

#### Technical Details

##### Constants

List of constants with their current default values:

- Maximum number of outbound connections: `10`
- Soft maximum number of inbound connections: `100`
- Number of buckets in the verified pool: `2^8` (`256`)
- Number of peer per verified buckets: `2^5` (`32`)
- Number of buckets in the unverified pool: `2^10` (`1024`)
- Number of peer per unverified buckets: `2^6` (`64`)
- Maximum number of duplicated peers in the unverified pool: `8`
- Probability of adding a Nth duplicated reference of an existing peer to the
unverified pool: `1/2^N`
- Period of verified peer random peer check when the maximum number
of verified connections has been reached: `60 seconds`
- Period of new peer connection up to the maximum number of verified
connections: `min(30, 2^(OUTBOUND_CONNECTION_COUNT - 1)) seconds`.
- Gossip ping frequency: `120 seconds`
- Maximum time to get a ping from an inbound connection: `30 seconds`
- Maximum time without activity (besides gossip) for inbound connections less
than 90 seconds old: `30 seconds`
- Maximum time without activity (besides gossip) for all connections: `180 seconds`
- Number of unverified buckets selected based on the source address group: `64`
- Number of unverified sub-buckets selected based on the peer address group: `4`
- Number of verified buckets selected based on peer address group: `8`

With these default values:

- The maximum cumulated number of distinct peers in the pools is from
16384 to 73728 depending on peer duplication in the unverified pool.
- The maximum number of peers that can be added through gossip by peers sharing
the same address group is from 512 to 4096 depending on peer duplication in the
unverified pool.
- The maximum number of neighboring peers sharing the same IP a single node
can add through gossip is from 23 to 184 depending on peer duplication in the
unverified pool.
- The minimum time to reach 10 outbound connections is 2 minutes and 31 seconds.

##### Bucket Selection

The bucket selection is done by hashing the secret with the discriminator and
use the result as an integer; this integer modulo is used to restrict the
subset.

For the unverified bucket selection:

```Erlang
<<N1:160>> = crypto:hash(sha, <<Secret/binary, PeerGroup/binary>>).
<<N2:160>> = crypto:hash(sha, <<Secret/binary, PeerAddress/binary>>).
<<N3:160>> = crypto:hash(sha, <<Secret/binary, SourceGroup/binary, (N1 rem 16):8, (N2 rem 4):8>>).
BucketIdx = N3 rem 1024.
```

For the verified bucket selection:

```Erlang
<<N1:160>> = crypto:hash(sha, <<Secret/binary, PeerAddress/binary>>).
<<N2:160>> = crypto:hash(sha, <<Secret/binary, PeerGroup/binary, (N1 rem 8):8>>).
BucketIdx = N2 rem 256.
```

#### Persistence

(Not yet implemented)

Persistence of the known peers is important in preventing Sybil/eclipse attacks.

If the node rebuild its list of peers from scratch every time it is restarted,
an attacker could use any other attack vector to crash the node or wait for
a scheduled restart and be the first to spam it with compromised addresses.

To support persistence, both verified and unverified pools are dumped to disk
periodically and loaded on startup.
