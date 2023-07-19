# 常见故障

# 写入超时

当出现写入超时时，服务器可能存在CPU瓶颈或者IO瓶颈。可以通过监控查看服务器的IO和CPU消耗信息。可能的原因可以分为：

1. 若IO吞吐和IOPS过高，则可以查看数据库监控信息里 `compaction `的相关统计次数，因为 compaction 过多时会造成很多的IO消耗，而可能导致写入超时。若出现此问题，通常建议提升机器配置来解决。
1. IOPS过高也可能由于小数据的写入请求太多，那么可以尝试提高单次写入的batch来减小请求数，从而减小写入磁盘的次数，从而提升性能。
1. 若CPU过高，则可能由于并发请求写入的数据量太多，通常建议在云环境下动态提升机器配置。

# OOM

当程序自动退出，可能因为OOM。可以通过下面三种方式来获得是否程序出现了OOM而被系统kill：

1. dmesg
1. /var/log/messages
1. 查看数据库的日志文件（可能包含OOM日志）



如果发生OOM则可使用以下步骤排查问题：

1. 数据库OOM通常由于用户的查询请求导致，所以在日志里找到出现OOM之前的较短时间内的query 请求语句。这需要这配置文件中打开记录query日志：

```TOML
[data]
  query-log-enabled = true
[logging]
  format = "auto"
  level = "info"
  enable-log-to-file = true
  filename = "fctsdb.log"
```

`query-log-enabled` 为true表示在日志里记录query语句，日志的路径则采用logging的相关配置。

那么在日志里记录的query日志类似如下（即日志包含：Executing query）：

```PlainText
ts=2021-07-25T11:39:57.541168Z lvl=info msg="Executing query" log_id=0VUUAUeG000 service=query query="SELECT time AS f_time, f_device_id AS f_device_id FROM iot_rockontrol.autogen.t_1 WHERE f_device_id IN ('78376448') AND time >= 451984h AND time <= 1627213197s"
```

2. 当找到可能触发OOM的语句时，可以使用`explain `语句分析此SQL语句是否访问了大量的shard以及series key。通常可以在where条件里添加time范围以及指定tag的查询来优化性能，从而避免由于访问大量数据而造成OOM。



如果暂时无法通过优化SQL以避免使用过多内存，那么可以开启全局内存控制，限制单个查询的最大内存使用，以此避免某些查询导致服务器OOM，详细使用请参考`高级特性`中的`内存限制`章节。

# 磁盘占用过高

当出现磁盘占用过高时，可以使用 df 命令查看目录大小。

1. 若是data目录过大，则可以考虑是否大量老的数据没有被删除，若是此情况，那么有价值的老数据可以通过备份工具进行备份（详细使用请参考`运维管理`中`管理工具`的`fctools`章节），若已经没有价值，则可能需要考虑修改retention policy，从而让数据库自动删除老的数据
1. 若是log日志过大，则检查是否开启了query log，或者是否开启了access log。当确认log已经可以删除时，则进行删除操作来减小磁盘占用；且当已经不需要打印query log时，则可以在fcshell或者dashbaord里动态关闭它。（通常，不会因为开启了慢查询日志导致log多大，若真如此，则应该优化慢查询且临时关闭慢日志输出）
1. 若是wal日志过多，则可检查oplog的清理逻辑相关配置是否设置合理，请参考`运维管理`中`参数配置`的`系统变量`章节，查看其中的如下变量：oplog-replica-clearup-interval 、oplog-replica-clearup-max-duration、oplog-replica-clearup-max-size、oplog-replica-clearup-timeout。



