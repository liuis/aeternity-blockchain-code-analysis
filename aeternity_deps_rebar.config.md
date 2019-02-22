## aenetiny deps

rebar.config

enoise提供了一种通用的握手机制，可以通过几种不同的方式使用。 还有一个简单的gen_tcp-wrapper，您可以将TCP套接字“升级”为Noise套接字，并以与使用gen_tcp相同的方式使用它。
当使用enoise进行交互式握手时，enoise将只处理消息组成/分解和加密/解密 - 即用户必须进行实际的发送和接收。

{enoise, {git, "https://github.com/aeternity/enoise.git",
          {ref, "1e6ee67"}}},
          
process dictionary
 {gproc, "0.6.1"},
 
Jobs是Erlang应用程序负载调节的作业调度程序。它提供了一个排队框架，可以为每个队列配置吞吐率，信用池和反馈补偿。可以在运行时添加和修改队列，并且可自定义的“采样器”在系统中的所有节点之间传播负载状态。

具体而言，作业提供三个功能：

作业调度：根据某些约束安排作业。例如，您可能希望定义不超过9个特定类型的作业可以同时执行，并且您可以启动此类作业的最大速率为每秒300个。

作业排队：当加载高于调度限制时，系统将其他作业排队等待以后在加载清除时运行。某些规则控制队列：它们是以FIFO还是LIFO顺序出列？队列在满员之前可以完成多少个工作？是否有截止日期，之后应该拒绝工作。当我们达到队列限制时，我们拒绝该工作。这在队列的客户端上提供了反馈机制，以便您可以采取措施。

采样和抑制：Erlang VM的定期样本可以提供有关系统健康状况的信息。如果我们有高CPU负载或高内存使用率，我们对调度规则应用阻尼：我们可以降低并发计数或执行作业的速率。当健康问题清除时，我们移除阻尼器并再次以全速运行。

 {jobs, "0.9.0"},
 
Exometer Core软件包允许简单有效地检测Erlang代码，从而将关键系统性能数据导出到各种监控系统。

Exometer Core带有一组预定义的监视器组件，可以使用自定义组件进行扩展，以处理新类型的度量标准，以及与其他外部系统（如数据库，负载平衡器等）的集成。

 {exometer_core, "1.5.7"},
 {poolboy, "1.5.1"},
 YAML 1.2 and JSON parser in pure Erlang
 {yamerl, "0.7.0"},
 Erlang performance and debugging tools
 {eper, "0.99.1"},

 {lager, {git, "https://github.com/erlang-lager/lager.git",
         {ref, "69b4ada"}}}, % tag: 3.6.7
 {cowboy, {git, "https://github.com/ninenines/cowboy.git",
          {ref, "8d49ae3"}}}, % tag: 2.2.2"
Sortable Erlang Term Serialization
{sext, {git, "https://github.com/uwiger/sext.git",
        {ref, "615eebc"}}},
 应用程序中的国际化域名
 {idna, {git, "https://github.com/benoitc/erlang-idna",
        {ref, "6cff727"}}}, % tag: 6.0.0
 Network Address Translation Port Mapping Protocol
 {nat, {git, "https://github.com/aeternity/erlang-nat.git",
       {ref, "dcdfb9c"}}},

 %% deps originally from aeternity

 % The rocksdb dependencies are removed on win32 to reduce build times,
 % because they are currently not working on win32.
 {mnesia_rocksdb, {git, "https://github.com/aeternity/mnesia_rocksdb.git",
                  {ref,"ad8e7b6"}}},

 {aecuckooprebuilt, {aecuckooprebuilt_app_with_priv_from_git,
                     {git, "https://github.com/aeternity/cuckoo-prebuilt.git",
                     {ref, "90afb699dc9cc41d033a7c8551179d32b3bd569d"}}}},

 Aeternity virtual machines byte code modules
 {aebytecode, {git, "https://github.com/aeternity/aebytecode.git",
              {ref,"99bf097"}}},
 内置的虚拟机语言
 {aesophia, {git, "https://github.com/aeternity/aesophia.git",
              {ref,"b61e3270f91129667d5948c4e0ce4b214a9d180c"}}},

 %% forks

 % waiting for https://github.com/jlouis/enacl/pull/40 to be merged
 {enacl, {git, "https://github.com/aeternity/enacl.git",
         {ref, "26180f4"}}},


 % waiting for https://github.com/for-GET/jesse/pull/75 to be merged
 {jesse, {git, "https://github.com/tolbrino/jesse.git",
         {ref, "b0a3bae"}}}, % tag 1.5.2 + http_uri patch

 % upstream is not maintained anymore
 {base58, {git, "https://github.com/aeternity/erl-base58.git",
          {ref,"60a3356"}}},

 % upstream is not maintained anymore
 {sha3, {git, "https://github.com/aeternity/erlang-sha3",
        {ref, "c818ddc"}}}
]}.
