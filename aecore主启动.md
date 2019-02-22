# aecore

### 1.主启动

aecore.app.src

```erlang
%% -*- mode: erlang; erlang-indent-level: 4; indent-tabs-mode: nil -*-
{application, aecore,
 [{description, "Blockchain for aeapps"},
  {vsn, {cmd, "cat ../../VERSION"}},
  {registered, []},
  {mod, { aecore_app, []}},
  {start_phases, [
                  {create_metrics_probes, []},
                  {start_reporters, []}
                 ]},
  {applications,
   [kernel,
    stdlib,
    crypto,
    sext,
    rocksdb,
    mnesia_rocksdb,
    eper,
    syntax_tools,
    compiler,
    goldrush,
    gproc,
    jobs,
    exometer_core,
    yamerl,
    lager,
    aeutils,
    base58,
    sha3,
    enacl,
    enoise,
    jsx,
    unicode_util_compat, %% to be removed, not needed in OTP 20
    idna,
    inets,
    nat,
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%    
    aecuckoo,
    aecuckooprebuilt,
    aetx,
    aevm,
    aeoracle,
    aechannel,
    aecontract,
    aesophia,
    aebytecode,
    aens,
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%% 
    ranch
  ]},
  {env, [
         %核心监控模块，输出到监控平台
         {exometer_predefined,
          {script, "$PRIV_DIR/exometer_predefined.script"}},
         {exometer_subscribers,
          [
           {select,
            {[
              {{[ae,epoch|'_'],counter,enabled},[],['$_']},
              {{[ae,epoch|'_'],gauge,enabled},[],['$_']}
             ],
             aec_metrics_main, [value], default, true}},
           {select,
            {[
              {{[ae,epoch|'_'],'$1',enabled},[{'=/=','$1',counter},
                                              {'=/=','$1',gauge}], ['$_']}
             ],
             aec_metrics_main, default, default, true}}
          ]},
         {metrics_probes,
          [{[ae,epoch,aecore,chain], aec_chain_metrics_probe},
           {[ae,epoch,aecore,tx_pool], aec_tx_pool_metrics_probe},
           {[ae,epoch,aecore,eper] , aec_eper_metrics_probe}]},
         {'$setup_hooks',
          [
           {normal, [
                     {110, {aecore_app, check_env, []}},
                     {110, {aehttp_app, check_env, []}},
                     {110, {aec_hard_forks, check_env, []}},
                     {110, {aec_pow_cuckoo, check_env, []}},
                     {200, {aec_db, check_db, []}}
                    ]}
          ]}
        ]},
  {modules, []},

  {maintainers, []},
  {licenses, []},
  {links, []}
 ]}.
```

### 2.Supervisor for the core application

```erlang
%%%    Supervisor for the core application
%%%
%%%  Full supervision tree is
%%%```
%%%       aecore_sup
%%%     (one_for_one)
%%%           |
%%%           -----------------------------------------
%%%           |         |         |           |       |
%%%           |   aec_metrics  aec_keys  aec_tx_pool  |
%%%           |                                       |
%%%   aec_connection_sup                      aec_conductor_sup
%%%     (one_for_all)                          (rest_for_one)
%%%           |                                       |
%%%           |                                       ---------------------
%%%           |                                       |                   |
%%%           |                                 aec_block_generator  aec_conductor
%%%           |
%%%           -------------------------------------------------------------------
%%%           |                    |         |            |                     |
%%%   aec_peer_connection_sup  aec_peers  aec_sync  aec_tx_pool_sync  aec_connection_listener
%%%     (simple_one_for_one)
%%%           |
%%%           --------------------
%%%           |             |
%%%   aec_peer_connection  ...
```



​	实际启动：

```
	?CHILD(aec_metrics_rpt_dest, 5000, worker),
    ?CHILD(aec_keys, 5000, worker),
    ?CHILD(aec_tx_pool_gc, 5000, worker),
    ?CHILD(aec_tx_pool, 5000, worker),
    ?CHILD(aec_conductor_sup, 5000, supervisor),
    ?CHILD(aec_connection_sup, 5000, supervisor)
```

### 3.aec_connection_sup

one_for_all

Supervisor for servers dealing with inter node communication.

 Individual connections cannot bring down the central servers (aec_peers, aec_sync), but if one of those goes down, all connections are brought down, and the whole peer/sync handling is restarted.

It is the responsibility of aec_peers to handle that a connection
 (aec_peer_connections) goes down. And the strategy there is to
 re-establish connections where the node is the initiator, and delegate
 the corresponding responsibility to the remote node.





aec_peer_connection_sup   ----->outgoing connections ------>
			simple_one_for_one --->  												    aec_peer_connection/模块封装一个P2P通信通道

aec_tx_pool_sync

模块持有一个短暂的缓存为gossip TX  - 理由因为大多数TX到达时间接近，因此mempool / DB
  可以通过第一级过滤器卸载。
  
aec_tx_gossip_cache

aec_peers

aec_sync

aec_peer ranch ranch_tcp   aec_peer_connection  ------->    incoming connections