# 数据缓存 

# 热点查询性能优化

## 实现原理

海东青数据库得益于良好的压缩算法可以实现较高的数据压缩比，进而支持海量数据存储。存储引擎读写数据的基本单位是数据块。在具有热点查询的场景中，磁盘数据块的读取和解压缩会消耗较多的 CPU 计算和磁盘 IO 资源。海东青针对热点查询场景，引入 Block Cache 将解压后的数据块缓存在内存中，可显著提升热点数据查询性能。

## 使用方法

```shell
var_name                                  var_type value     restart_value range                 desc
--------                                  -------- -----     ------------- -----                 ----          
data-block-cache-memory-size              SYSTEM   64M       64M           [0,128M]              The default time a write request will wait until a "timeout" error is returned to the caller the maximum size a engine's block cache can reach, set to 0 when the block cache need to be turned off
```

Block Cache 在系统变量中对应的控制参数是 `data-block-cache-memory-size` ，用于控制单个 Shard 上 Block Cache 占用的内存总大小。Block Cache 的默认大小是 64MB，可根据业务场景的热点数据集调整。

# 瞬时峰值写入性能优化

海东青数据库采用基于 LSM-tree 数据组织存储引擎。数据先写入到 cache，当 cache 达到一定的大小后再刷写到磁盘。cache 中的数据通过 WAL 保证数据的持久化。当出现瞬时的峰值写入时，由于 cache 达到阈值执行刷写操作。而刷写磁盘本身是相对会消耗较多磁盘贷款的操作，容易导致写入超时。可以通过增大 cache 大小，在内存中缓存更多的数据，通过将磁盘刷写和峰值写入在时间维度上错开，以避免写入请求超时。

```sShell
var_name                          var_type  value  restart_value  range       desc
--------                          --------  -----  -------------  -----       ----          
data-cache-snapshot-memory-size   SYSTEM    25M    25M            [1.0K,250M] Memory size threshold of shard cache to snapshot and write TSM file
```

cache 的刷写阈值在系统变量中对应的控制参数是 `data-cache-snapshot-memory-size`，默认值为 25MB。在出现瞬时峰值写入导致超时时，可尝试调整该参数。

