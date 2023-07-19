# 保留策略管理

保留策略 `retention policy`是资源路径中的一层；MySQL中 `database`的下一层是 `table`, 而在海东青中，`database`的下一层是 `retention policy`。一个数据库可以拥有多个保留策略。

### 创建保留策略

`CREATE RETENTION POLICY <retention_policy_name> ON <database_name> DURATION <duration> REPLICATION <n> [DEFAULT]`

```sql
/* 示例： */
create retention policy "onemonth" on "environment" duration 1d replication 1
```

`DURATION`

确定海东青数据库保留数据的时间。`<duration>`是持续时间字符串或INF（无限）。 保留策略的最短持续时间为1小时，最大持续时间为INF。

`REPLICATION`

确定每个点的多少独立副本存储在集群中，其中`n`是数据节点的数量。该子句不能用于单节点实例。

`DEFAULT`

 将新的保留策略设置为数据库的默认保留策略；此设置是可选的。 写入数据时若没有指定 `retention policy`，数据将会被写入默认保留策略下。每个数据库的初始默认保留策略是 `autogen`, 其数据保存时间是无穷。

### 查看保留策略

`SHOW RETENTION POLICIES on <database_name>`

```sql
/*示例： */
> show retention policies on environment
name    duration shardDuration replicaN default
----    -------- ------------- -------- -------
autogen 0s       168h0m0s      1        true
```

### 修改保留策略

`ALTER RETENTION POLICY <retention_policy_name> ON <database_name> DURATION <duration> REPLICATION <n> DEFAULT`

```sql
/* 示例： */
alter retention policy "onemonth" on "environment" duration 3w shard duration 30m default
```

### 删除保留策略

`DROP RETENTION POLICY <retention_policy_name> ON <database_name>`

```sql
/* 示例： */
drop retention policy "onemonth" on "environment"
```

