# 集群部署

# 主从复制配置文件说明

海东青数据库支持一主多从日志复制模式，以满足高可用特性，并且支持主从故障自动切换。
主从复制相关的配置在配置文件的 `rpc-server` 和 `oplog-replica` 段，配置文件示例请参考[参数配置](./configuration.md)。

### rpc-server

`bind-addr`：海东青数据库主从节点复制使用 rpc 交互，所以开启数据库的 rpc 服务是前提，需要配置 rpc 服务监听地址。

###  oplog-replica

`role`：海东青数据库实例在主从复制中的角色，分为 leader 和 follower，默认为 leader。follower 会连接至 leader 并从 leader 拉取数据。follower 只支持读数据，而 leader 可读可写。

`min-isr`：该项指定海东青数据库主从复制时数据一致性的强度，默认为 1，即日志只用写入本节点即可成功返回。该值表明日志至少同步至多少个节点时才算写入成功，值越大，集群间数据的一致性就越强，但可用性也会相应降低。

`replica-enabled`：是否开启主从日志复制

`cluster-addrs`：集群中其他节点的 rpc 地址。当本数据库实例切换至 follower 时，会从该项配置的地址列表中寻找 leader。如果不需要使用主从故障自动切换，那么主节点的`cluster-addrs`可以直接留空，从节点的`cluster-addrs`则也只需要填写主节点`rpc-server`中的`bind-addr`选项值即可。此外，海东青支持运行时使用API进行动态修改节点。

# API 介绍

### 设置数据库在主从复制中的角色

Method：POST 

URI：/command/role

Body：

```json
{
  "role": "leader"    // leader or follower
}
```

Response：NoContent

> Note：该 api 会切换当前数据库的角色，配置文件中的 `oplog-replica.role` 字段也会被同步更改。

### 查询数据库在主从复制中的角色

Method：GET

URI：/command/role

Response：

```json
{
  "role": "leader"                                                            // leader or follower
  "node_id": "1656404646276332000"        // 数据库实例随机生成的 id，用以区分其他实例
  "clusterAddrs": [                                                                        // 集群中其他实例的 rpc 地址列表
    "192.168.10.2:8090",
    "192.168.10.3:8090",
  ]
}
```

### 向 clusterAddrs 列表中新增 rpc 地址

Method：POST 

URI：/command/nodes

Body：

```JSON
{
  "addrs": [
    "192.168.10.2:8090",
    "192.168.10.3:8090",
  ]
}
```

Response：NoContent

Note：配置文件中的 `oplog-replica.cluster-addrs`字段也会被同步更改。

### 从 clusterAddrs 列表中删除 rpc 地址

Method：DELETE

URI：/command/nodes

Body：

```JSON
{
  "addrs": [
    "192.168.10.2:8090",
  ]
}
```

# 管理命令

api 提供客户端或者工具调用，但不便于人工调用，因此为了方便管理员管理主从集群，fcshell 也相应新增了若干管理命令。

### 设置数据库在主从复制中的角色

```Bash
> set state leader
> set state follower
```

### 查看数据库在主从复制中的角色

```Bash
> show state
state: leader
nodes: [172.19.192.37:18092 172.19.192.37:18091]
```

### 向 clusterAddrs 列表中新增 rpc 地址

```Bash
> append nodes "172.19.192.37:18092" "172.19.192.37:18091"
```

### 从 clusterAddrs 列表中删除 rpc 地址

```
> drop nodes "172.19.192.37:18092" "172.19.192.37:18091"
```

### 查看某 shard 的锚点记录

```Bash
> show anchorpoints for shard db_autogen_20220718000000_20220725000000
NodeID: 1658301619406535600, Value: 1
NodeID: 1658301621632950800, Value: 3
NodeID: 1658301621632950800, Value: 6
NodeID: 1658301621632950800, Value: 7
```

数据库发生主从切换时，shard 会记录锚点，用于日志冲突检测，用户一般不需要查询 shard 的锚点记录，只是管理员在手动处理数据冲突时可以通过此命令获得一些帮助。

# Keepalived 高可用模式

海东青数据库不具备主从自动切换能力，要么由运维手动通过 api 或者命令进行切换，要么可由第三方高可用工具驱动切换。

以 Keepalived 为例，海东青数据库提供了 API 供 keepalived 探活和状态切换，调用这些 API 的 shell 脚本参考如下：

### 健康检测脚本

```bash
#!/bin/bash

# usage
# healthcheck.sh ip:port

addr=$1
result=`curl http://${addr}/command/role 2>/dev/null`
if [[ $result =~ "role" ]]
then
exit 0
else
exit -1
fi
```


需要通过命令行参数向该脚本传递数据库实例的 http 地址(`ip:port` 形式)。


### 角色切换脚本

```bash
#!/bin/bash

# usage
# switchstate.sh ip:port role

addr=$1
role=$2
curl -X POST -H "'Content-type':'application/json'" -d "{\"role\":\"${role}\"}" http://${addr}/command/role
```


海东青数据库本身的主从复制和自动寻找主节点的能力，搭配上 keepalived，可实现数据库服务的高可用。

# 冲突处理

因为海东青数据库的主从复制并不是强一致的，而是优先保证高可用，故不同节点间数据发生冲突在所难免。在日常使用场景中，大部分的冲突都能被自动处理，无需应用和管理员介入。只存在如下少量场景需要管理员人工处理冲突。

### metadata 冲突

若主从复制时 metadata 存在冲突，则从节点的 meta 目录下有个文件 `merge_meta_desc` 会对冲突进行描述。

若是 users，roles 之类的元数据存在冲突，则运维可自行根据需要通过 fcshell 命令进行手动修复，当然即便搁置不处理也不会影响数据库的正常运行。

若是 database 之类的元数据存在冲突，则运维根据冲突描述中的指示进行操作。基本步骤分为如下三步：

1. 从节点上导出存在冲突的数据，若 db 或 rp 存在冲突，则导出整个 db 或 rp 的数据，若 shard 存在冲突，则导出整个 shard 的数据。
1. 从节点删除存在冲突的 db、rp 或 shard。
1. 将第一步中导出的数据 import 至主节点。

### shard 数据冲突

在 shard 自动处理日志冲突时，若发现部分存在冲突的日志已被删除，则会跳过冲突处理，继续后续的日志复制，尽可能保证可用性。这时就需要运维来手动处理主从 shard 数据的冲突。

一方面，若冲突的日志中包含一些删除语句，follower 会将这些删除语句过滤出来并写入文件，文件位于 wal 路径中该 shard 的目录下，文件名以 `conflict `为前缀。运维可酌情将这些语句在 leader 上重新执行。

另一方面，若日志报告有 shard 的日志冲突被跳过而未处理，运维可通过如下三步骤处理该 shard 的数据冲突：

1. 从节点导出该 shard 的所有数据
1. 从节点删除该 shard
1. 将第一步导出的数据 import 至主节点。
