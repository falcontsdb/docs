# 慢查询

# 定位慢查询

海东青数据库会将执行时间超过设定阈值的查询输出到日志，用于帮助用户定位慢查询语句，分析和解决查询的性能问题。

### 相关参数

1. 配置文件中配置慢日志文件路径 `monitor-log`

```toml
[logging]
  # 请求的监控日志，以及慢日志的路径
  monitor-log = ""
```

2. 系统变量中配置慢日志相关时间阈值

|参数名 |含义 | 默认值|
| ----------- | ----------- |----------- |
|memcontrol-slow-task-quota  |在日志中记录超过该阈值的查询，设置为 0s 则禁用该功能 | 5s|
|memcontrol-task-timeout |设置请求的超时时间，当超过此时间时会终止请求，设置为0表示不进行超时检测及其终止操作 | 5m|

### 执行中的慢查询示例

执行中且执行时间超过阈值的查询慢日志示例如下：

```json
2022-07-20T01:59:14.199663Z     info    slow task       {"log_id": "0bnL5vSl000", "service": "task manager", "id": 3, "name": "SELECT * FROM t1.autogen.cpu LIMIT 1", "db": "t1", "rp": "", "user": "root", "start time": "2022-07-20T01:59:09.155216Z", "hard timeout": "300000.000ms", "bytes consumed": 1160, "bytes limit": 17179869184}
```

因此可搜索monitor-log指向的文件，使用"slow task"字符串进行查找其内容中的慢查询记录。

### 字段含义说明

* id：查询对应的 ID
* name：查询对应的 sql 语句
* db：查询的database
* rp：查询的RP，若为空则表示默认RP：autogen
* user：执行查询的用户
* start time：查询执行的开始时间
* hard timeout：查询的硬超时时间，查询执行超过此时间时查询将被终止
* bytes consumed：查询的内存消耗
* bytes limit：查询的内存使用上限，当超过此内存时会终止查询请求



