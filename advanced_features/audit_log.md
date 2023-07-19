# 审计日志

为了保证数据库的安全，需要记录用户对数据库的某些操作，以能够在出现安全问题的时候帮助审查。



# 审计表结构

|Tag|	说明|
| ----------- | ----------- |
|user|	用户|
|event_type|	操作事件类型|

|Field	|说明|
| ----------- | ----------- |
|time	|操作时间|
|role	|用户的角色类型|
|status	|操作所属阶段(start表示操作开始, end表示操作结束)|
|server_host|	服务器host(IP)|
|user_host	|用户host(IP)|
|describe	|此次操作的整体描述|
|query_statement|	用户执行的语句|
|hostname	|服务器的主机名|
|duration|	执行时长|
|result	|执行结果|

# 查询审计日志

审计日志作为数据，记录在内部_fctsdb数据库的audit_log表里，所以可通过以下三种方式查询：

1. fcshell 里执行 `SELECT * from _fctsdb..audit_log; `来查询：

```sql
> select * from _fctsdb..audit_log limit 5;
The FalconTSDB you accessed has not been initialized! You can use `init` command to initialize!
name: audit_log
time                describe                                                               duration event_type       hostname        query_statement                    result  role server_host status user         user_agent            user_host
----                --------                                                               -------- ----------       --------        ---------------                    ------  ---- ----------- ------ ----         ----------            ---------
1625479387084143400 initial user drop measurement: falcontsdb_system of database: telegraf 0        drop_measurement SKY-20200327IVH DROP MEASUREMENT falcontsdb_system success      [::1]:8086  start  initial user FalconTSDBShell/devel ::1
1625479387832121800 initial user drop measurement: falcontsdb_system of database: telegraf 646      drop_measurement SKY-20200327IVH DROP MEASUREMENT falcontsdb_system success      [::1]:8086  end    initial user FalconTSDBShell/devel ::1
1626330942171589100 initial user drop shard: 1                                             0        drop_shard       SKY-20200327IVH DROP SHARD 1                       success      [::1]:8086  start  initial user FalconTSDBShell/devel ::1
1626330942888083100 initial user drop shard: 1                                             11       drop_shard       SKY-20200327IVH DROP SHARD 1                       success      [::1]:8086  end    initial user FalconTSDBShell/devel ::1
1626330956178951200 initial user drop shard: 2                                             0        drop_shard       SKY-20200327IVH DROP SHARD 2                       success      [::1]:8086  start  initial user FalconTSDBShell/devel ::1
```

2. 在海东青控制台的左侧打开 `SQL审计` 进行查询。
3. 使用falcontsdb-client SDK进行查询
