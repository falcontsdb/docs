# Fctools

# 介绍

fctools 是一个工具箱，集合了一系列的小工具，辅助 falcontsdb 服务的诊断维护。执行 `fctools -help` 可以查看帮助信息：

```Bash
NAME:
   fctools - Tool box for falcontsdb

USAGE:
   fctools [global options] command [command options] [arguments...]

VERSION:
   devel (git: 1.4 34990fa)


COMMANDS:
   backup              backup [options] PATH
   cold-storage        tool to manage cold storage
   compact             Fully compacts the specified shard
   delete-measurement  Deletes measurement from raw tsm files
   dump-tsi            Dumps low-level details about tsi1 files
   dump-tsm            Dumps low-level details about tsm1 files
   estimate            Estimate number of points from tsm1 data
   export              Exports raw data from a shard to line protocol
   fix
   generate            Generates data
   gm
   import              Import a previous database exported from file.
   index-seriestime    Generate series time index from tsm1 data
   index-tsi           Generates tsi1 indexes from tsm1 data
   init                Creates database and retention policy metadata
   maintain-backup     maintain-backup [options] PATH
   report              Displays shard level report
   reshard             Reshapes existing shards to a new shard duration and print results to stdout, with no influence on original data
   restore             restore [options] PATH
   upgrade             smooth upgrade automatically by leader downgrade to follower and follower upgrade to leader
   verify-series       Verifies integrity of series files
   verify-tsm          Verifies integrity of TSM files
   help, h             Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version

COPYRIGHT:
   Copyright © 2019 - 2022 rockontrol. All Rights Reserved.
```

fctools 是一系列子命令的集合，帮助手册显示的 COMMANDS 即为所有的子命令。每个子命令功能彼此独立，且会根据需要进行扩充。可以执行 `fctools help command` 查看某个子命令的使用手册，如:

```Bash
fctools help upgrade
NAME:
   fctools upgrade - smooth upgrade automatically by leader downgrade to follower and follower upgrade to leader

USAGE:
   fctools upgrade [command options] [arguments...]

OPTIONS:
   --new_leader_addr value  http address of new leader
   --addrs value            http address list of all members in replica
   --interval value         interval to detect replication situations (default: 5s)
   --do-change value        execute upgrade or just watch replication progress
   --username value         the authentication username of current leader
   --password value         the authentication password of current leader
```

# 基本使用示例

## commit-wal

主动将 wal 文件合并至 tsm 文件

```Bash
./fctools commit-wal --data ~/.fctsdb/data --wal ~/.fctsdb/wal
2020-07-29T07:03:32.293099Z     info    committing database     {"log_id": "0OILz6m0000", "name": "mydb"}
2020-07-29T07:03:32.295269Z     info    committing retention policy     {"log_id": "0OILz6m0000", "db_instance": "mydb", "db_rp": "autogen"}
2020-07-29T07:03:32.295432Z     info    new engine      {"log_id": "0OILz6m0000", "data-dir": "~/.fctsdb/data/mydb/autogen/1"}
2020-07-29T07:03:32.295463Z     info    snapshot size equals 0  {"log_id": "0OILz6m0000", "data-dir": "~/.fctsdb/data/mydb/autogen/1"}
```

## compact

主动合并某 shard 的 tsm 文件

```Bash
./fctools compact --path ~/.fctsdb/data/mydb/autogen/1
opening shard at path "~/.fctsdb/data/mydb/autogen/1"

The following files will be compacted:

TSM:
  ~/.fctsdb/data/mydb/autogen/1/000000001-000000001.tsm

Proceed? [N] y
Compacting shard.
Compaction succeeded. New files:
  ~/.fctsdb/data/mydb/autogen/1/000000001-000000002.tsm
```

## delete-measurement

从 tsm 文件中删除某 measurement。

```Bash
./fctools  delete-measurement --verbose --measurement cpu ~/.fctsdb/data/mydb/autogen/1/000000001-000000002.tsm
2020/07/29 16:17:56 processing: ~/.fctsdb/data/mydb/autogen/1/000000001-000000002.tsm
2020/07/29 16:17:56 deleting block: cpu,core=1#!~#value (2020-07-29T04:04:02.983331Z-2020-07-29T04:04:06.004007Z) sz=52
2020/07/29 16:17:56 deleting block: cpu,core=2#!~#value (2020-07-29T04:04:20.662119Z-2020-07-29T04:04:24.446216Z) sz=50
no values written
```

## dump-tsm

解析 tsm 文件的详细信息

```Bash
./fctools dump-tsm --all ~/.fctsdb/data/mydb/autogen/1/000000001-000000002.tsm
Summary:
  File: ~/.fctsdb/data/mydb/autogen/1/000000001-000000002.tsm
  Time Range: 2020-07-29T04:04:02.983331Z - 2020-07-29T04:04:24.446216Z
  Duration: 21.462885s   Series: 2   File Size: 227

Index:

  Pos   Min Time                        Max Time                        Ofs     Size    Key             Field
  1     2020-07-29T04:04:02.983331Z     2020-07-29T04:04:06.004007Z     5       56      cpu,core=1      value
  2     2020-07-29T04:04:20.662119Z     2020-07-29T04:04:24.446216Z     61      54      cpu,core=2      value
Blocks:
  Blk   Chk             Ofs     Len     Type    Min Time                        Points  Enc [T/V]       Len [T/V]
  0     2961381638      5       52      float64 2020-07-29T04:04:02.983331Z     4       s8b/gor         25/25
  1     1390243130      61      50      float64 2020-07-29T04:04:20.662119Z     4       s8b/gor         25/23

Statistics
  Blocks:
    Total: 2 Size: 110 Min: 50 Max: 52 Avg: 55
  Index:
    Total: 2 Size: 104
  Points:
    Total: 8
  Encoding:
    Timestamp:  none: 0 (0%)    s8b: 2 (100%)
    Float:      none: 0 (0%)    gor: 2 (100%)
  Compression:
    Per block: 13.75 bytes/point
    Total: 28.38 bytes/point
```

## exec

将指令发送到某节点 RPC 服务执行，目前只支持 snapshot 指令

./fctools  exec --addr localhost:8090 --type snapshot

## export

将数据导出到文本文件，可以指定某个 data 目录，或者某个 database，某个 retention policy，某个 wal 目录，某个时间段进行导出，具体查看 help 信息。下面示例导出 mydb 数据库到默认文件 ~/.fctsdb/export:

```Bash
./fctools export --database mydb
writing out tsm file data for mydb/autogen...complete.

cat ~/.fctsdb/export
# FalconTSDB EXPORT: 1677-09-21T08:18:26+08:05 - 2262-04-12T07:47:16+08:00
# DDL
CREATE DATABASE mydb WITH NAME autogen
# DML
# CONTEXT-DATABASE:mydb
# CONTEXT-RETENTION-POLICY:autogen
# writing tsm data
cpu,core=1 value=30 1595995442983331000
cpu,core=1 value=31 1595995444243012000
cpu,core=1 value=32 1595995445133452000
cpu,core=1 value=33 1595995446004007000
cpu,core=2 value=40 1595995460662119000
cpu,core=2 value=41 1595995461848613000
cpu,core=2 value=42 1595995463109261000
cpu,core=2 value=43 1595995464446216000
```

## generate

根据指定的 toml 文件的描述，在给定的数据库给定的 retention policy 中自动构造生成数据，可便于构造大量测试数据。注意: 该操作会先清空给定库中的所有数据；为避免写文件冲突，也请在执行期间停止 tsdb 服务。

在生成数据前，先编辑 toml 文件，定义好数据的生成规则，若不知道 toml 文件如何写，可以打印 toml 文件样例:

```Bash
./fctools  generate --example
title = "Documented schema"

# limit the maximum number of series generated across all measurements
#
# series-limit: integer, optional (default: unlimited)

[[measurements]]
......
```

输出中有对各个字段的详细解释。一个可用的 toml 配置如下:

```TOML
[[measurements]]
name = "mem"
tags = [
    { name = "host",   source = { type = "sequence", format = "host-%s", start = 0, count = 5 } },
    { name = "region", source = ["us-west-01","us-west-02","us-east"] },
]
fields = [
    # An example of a sequence of integer values
    { name = "free",    count = 100, source = [10,15,20,25,30,35,30], time-precision = "ms" },
    { name = "low_mem", count = 100, source = [false,true,true], time-precision = "ms" },
]
```

该 toml 表示: 

生成的 measurement 为 mem。

其有两个 tags，分别为 host 和 region，host 的取值范围从 "host-0" 至 "host-4"，region 取值范围为 ["us-west-01","us-west-02","us-east"]。

有两个 fields，分别为 free 和 low_mem，取值范围分别为 [10,15,20,25,30,35,30] 和 [false,true,true]，`count 字段的机制还没弄明白`。



在准备生成数据前，可以先打印生成计划以进一步确认:

```Shell
./fctools generate --db sample --rp autogen --schema ./sample.toml --print
Data Path                ~/.fctsdb/data/sample/autogen
Shard Count              1
Database                 sample/autogen (Shard duration: 24h0m0s)
Shard Duration           24h0m0s
Start time               2020-08-02 08:00:00 +0800 CST
End time                 2020-08-03 08:00:00 +0800 CST
```

如上显示将在 sample.autogen 数据库中生成数据，数据存放的目录为 `~/.fctsdb/data/sample/autogen`，将产生一个 shard，shard duration 为 24h，起始截止时间也显示了。

在确认好之后，可以生成数据:

```Shell
./fctools generate --db sample --rp autogen --schema ./sample.toml
```

最后可以启动 tsdb，进入 fcshell 查看确认:

```Shell
> show databases
name: databases
name
----
_fctsdb
mydb
sample
> use sample
Using database sample
> show measurements
name: measurements
name
----
mem
> select * from mem limit 10
name: mem
time                free host   low_mem region
----                ---- ----   ------- ------
1596326400000000000 10   host-0 false   us-east
1596326400000000000 10   host-3 false   us-west-01
1596326400000000000 10   host-0 false   us-west-01
1596326400000000000 10   host-1 false   us-east
1596326400000000000 10   host-2 false   us-west-01
1596326400000000000 10   host-0 false   us-west-02
1596326400000000000 10   host-2 false   us-west-02
1596327264000000000 15   host-0 true    us-west-02
1596327264000000000 15   host-2 true    us-west-02
1596327264000000000 15   host-0 true    us-west-01
```

## init

根据命令行指定的参数创建新的数据库，若有现存的同名数据库则会被清空。

```Shell
./fctools init --database sample -rp forever --rf 10
Data Path                ~/.fctsdb/data/sample/forever
Shard Count              1
Database                 sample/forever (Shard duration: 24h0m0s)
Start time               2020-07-28 08:00:00 +0800 CST
End time                 2020-07-29 08:00:00 +0800 CST
```

此时进入 fcshell 可以查看到刚才新建的数据库 sample:

```Shell
./fcshell
> show databases
> show databases
name: databases
name
----
_fctsdb
sample
```

进入新建的数据库可以查看其 retention policies:

```Shell
> use sample
Using database sample
> show retention policies
name    duration shardGroupDuration replicaN default
----    -------- ------------------ -------- -------
autogen 0s       24h0m0s            10       true
```

## report

报告 shard 的详细情况

```Shell
./fctools  report --exact --detail ~/.fctsdb/data/mydb/autogen/1
DB      RP      Shard   File                    Series  New     Min Time                    Max Time                    Load Time
mydb    autogen 1       000000001-000000002.tsm 2       2       2020-07-29T04:04:02.983331Z 2020-07-29T04:04:24.446216Z 487.698µs

Summary:
  Files: 1
  Time Range: 2020-07-29T04:04:02.983331Z - 2020-07-29T04:04:24.446216Z
  Duration: 21.462885s

Statistics
  Series:
     - mydb: 2 (100%)
  Total: 2

  Measurements (est):
    - cpu: 2 (100%)

  Fields (est):
    - cpu: 1

  Tags (est):
    - core: 2
Completed in 2.22387ms
```

## verify-series

校验 series file 的完整性，可选择某数据库的 series files，或指定某个 series file，或某个目录下的 series files。

```Shell
./fctools verify-series --verbose --database mydb
2020-07-29T08:09:05.601623Z     info    Verifying series file   {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series"}
2020-07-29T08:09:05.601822Z     info    Verifying partition     {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "00"}
2020-07-29T08:09:05.601899Z     info    Verifying segment       {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "00", "segment": "0000"}
2020-07-29T08:09:05.601987Z     info    Verifying index {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "00"}
2020-07-29T08:09:05.602030Z     info    Verifying partition     {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "01"}
2020-07-29T08:09:05.602096Z     info    Verifying segment       {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "01", "segment": "0000"}
2020-07-29T08:09:05.602162Z     info    Verifying index {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "01"}
2020-07-29T08:09:05.602186Z     info    Verifying partition     {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "02"}
2020-07-29T08:09:05.602240Z     info    Verifying segment       {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "02", "segment": "0000"}
2020-07-29T08:09:05.602296Z     info    Verifying index {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "02"}
2020-07-29T08:09:05.602323Z     info    Verifying partition     {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "03"}
2020-07-29T08:09:05.602424Z     info    Verifying segment       {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "03", "segment": "0000"}
2020-07-29T08:09:05.602482Z     info    Verifying index {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "03"}
2020-07-29T08:09:05.602506Z     info    Verifying partition     {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "04"}
2020-07-29T08:09:05.602553Z     info    Verifying segment       {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "04", "segment": "0000"}
2020-07-29T08:09:05.602604Z     info    Verifying index {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "04"}
2020-07-29T08:09:05.602919Z     info    Verifying partition     {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "05"}
2020-07-29T08:09:05.602987Z     info    Verifying segment       {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "05", "segment": "0000"}
2020-07-29T08:09:05.603061Z     info    Verifying index {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "05"}
2020-07-29T08:09:05.603087Z     info    Verifying partition     {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "06"}
2020-07-29T08:09:05.603146Z     info    Verifying segment       {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "06", "segment": "0000"}
2020-07-29T08:09:05.603205Z     info    Verifying index {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "06"}
2020-07-29T08:09:05.603237Z     info    Verifying partition     {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "07"}
2020-07-29T08:09:05.603296Z     info    Verifying segment       {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "07", "segment": "0000"}
2020-07-29T08:09:05.603355Z     info    Verifying index {"log_id": "0OIPjBGG000", "path": "~/.fctsdb/data/mydb/_series", "partition": "07"}
```

## verify-tsm

校验 tsm 文件的完整性，默认校验 `.fctsdb/data` 下的所有 tsm 文件。

```Shell
./fctools  verify-tsm
~/.fctsdb/data/mydb/autogen/1/000000001-000000002.tsm: healthy
Broken Blocks: 0 / 2, in 0.00145052s
```

## reshard

以新 duration 区间重新分割 shard，按新的分组打印数据，不会对数据库做修改。

比如有 mydb.autogen 数据库，其有 2 个 shards 如下:

```Shell
{
  "Name": "mydb",
  "DefaultRetentionPolicy": "autogen",
  "RetentionPolicies": [
    {
      "Name": "autogen",
      "ReplicaN": 1,
      "Duration": 0,
      "ShardGroupDuration": 604800000000000,
      "ShardGroups": [
        {
          "ID": 1,
          "StartTime": "2020-07-27T00:00:00Z",
          "EndTime": "2020-08-03T00:00:00Z",
          "DeletedAt": "0001-01-01T00:00:00Z",
          "Shards": [
            {
              "ID": 1,
              "Owners": null
            }
          ],
          "TruncatedAt": "0001-01-01T00:00:00Z"
        },
        {
          "ID": 12,
          "StartTime": "2020-08-03T00:00:00Z",
          "EndTime": "2020-08-10T00:00:00Z",
          "DeletedAt": "0001-01-01T00:00:00Z",
          "Shards": [
            {
              "ID": 12,
              "Owners": null
            }
          ],
          "TruncatedAt": "0001-01-01T00:00:00Z"
        }
      ],
      "Subscriptions": null
    }
  ],
  "ContinuousQueries": null
}
```

其 retention policy 为 autogen，duration 为默认的 7 天，跨越 14 天，若执行 reshard 命令，将 duration 指定为 1 天 (1d)，那么所有的数据将以 14 个 shard 分块打印输出。

如下命令尝试将 mydb.autogen 数据库以 duration = 1d 重新分割，若数据有冲突则将冲突写至指定的 conflict 文件，同时加上 --print-only 选项表示只是打印执行计划而非真的执行。

```Shell
./fctools reshard --database mydb --rp autogen --duration 24h --conflict-path conflict --print-only
Source data from: 2020-07-27 00:00:00 +0000 UTC -> 2020-08-10 00:00:00 +0000 UTC

Converting source from 2 shard group(s) to 14 shard groups:

Seq #           ID              Start                           End
0               1               2020-07-27 00:00:00 +0000 UTC   2020-08-03 00:00:00 +0000 UTC
1               12              2020-08-03 00:00:00 +0000 UTC   2020-08-10 00:00:00 +0000 UTC

Seq #           ID              Start                           End
0               0               2020-07-27 00:00:00 +0000 UTC   2020-07-28 00:00:00 +0000 UTC
1               1               2020-07-28 00:00:00 +0000 UTC   2020-07-29 00:00:00 +0000 UTC
2               2               2020-07-29 00:00:00 +0000 UTC   2020-07-30 00:00:00 +0000 UTC
3               3               2020-07-30 00:00:00 +0000 UTC   2020-07-31 00:00:00 +0000 UTC
4               4               2020-07-31 00:00:00 +0000 UTC   2020-08-01 00:00:00 +0000 UTC
5               5               2020-08-01 00:00:00 +0000 UTC   2020-08-02 00:00:00 +0000 UTC
6               6               2020-08-02 00:00:00 +0000 UTC   2020-08-03 00:00:00 +0000 UTC
7               7               2020-08-03 00:00:00 +0000 UTC   2020-08-04 00:00:00 +0000 UTC
8               8               2020-08-04 00:00:00 +0000 UTC   2020-08-05 00:00:00 +0000 UTC
9               9               2020-08-05 00:00:00 +0000 UTC   2020-08-06 00:00:00 +0000 UTC
10              10              2020-08-06 00:00:00 +0000 UTC   2020-08-07 00:00:00 +0000 UTC
11              11              2020-08-07 00:00:00 +0000 UTC   2020-08-08 00:00:00 +0000 UTC
12              12              2020-08-08 00:00:00 +0000 UTC   2020-08-09 00:00:00 +0000 UTC
13              13              2020-08-09 00:00:00 +0000 UTC   2020-08-10 00:00:00 +0000 UTC
```

如上 --print-only 输出显示 reshard 命令会将原来的两个 shard 以天为单位重新划分为 14 个 shards。

现在去掉 --print-only 选项，真正执行 reshard:

```Shell
 ./fctools reshard --database mydb --rp autogen --duration 24h --conflict-path conflict
Source data from: 2020-07-27 00:00:00 +0000 UTC -> 2020-08-10 00:00:00 +0000 UTC

Converting source from 2 shard group(s) to 14 shard groups:

Seq #           ID              Start                           End
0               1               2020-07-27 00:00:00 +0000 UTC   2020-08-03 00:00:00 +0000 UTC
1               12              2020-08-03 00:00:00 +0000 UTC   2020-08-10 00:00:00 +0000 UTC

Seq #           ID              Start                           End
0               0               2020-07-27 00:00:00 +0000 UTC   2020-07-28 00:00:00 +0000 UTC
1               1               2020-07-28 00:00:00 +0000 UTC   2020-07-29 00:00:00 +0000 UTC
2               2               2020-07-29 00:00:00 +0000 UTC   2020-07-30 00:00:00 +0000 UTC
3               3               2020-07-30 00:00:00 +0000 UTC   2020-07-31 00:00:00 +0000 UTC
4               4               2020-07-31 00:00:00 +0000 UTC   2020-08-01 00:00:00 +0000 UTC
5               5               2020-08-01 00:00:00 +0000 UTC   2020-08-02 00:00:00 +0000 UTC
6               6               2020-08-02 00:00:00 +0000 UTC   2020-08-03 00:00:00 +0000 UTC
7               7               2020-08-03 00:00:00 +0000 UTC   2020-08-04 00:00:00 +0000 UTC
8               8               2020-08-04 00:00:00 +0000 UTC   2020-08-05 00:00:00 +0000 UTC
9               9               2020-08-05 00:00:00 +0000 UTC   2020-08-06 00:00:00 +0000 UTC
10              10              2020-08-06 00:00:00 +0000 UTC   2020-08-07 00:00:00 +0000 UTC
11              11              2020-08-07 00:00:00 +0000 UTC   2020-08-08 00:00:00 +0000 UTC
12              12              2020-08-08 00:00:00 +0000 UTC   2020-08-09 00:00:00 +0000 UTC
13              13              2020-08-09 00:00:00 +0000 UTC   2020-08-10 00:00:00 +0000 UTC
# new shard group start: 2020-07-27 00:00:00 +0000 UTC -> end: 2020-07-28 00:00:00 +0000 UTC
# new shard group start: 2020-07-28 00:00:00 +0000 UTC -> end: 2020-07-29 00:00:00 +0000 UTC
# new shard group start: 2020-07-29 00:00:00 +0000 UTC -> end: 2020-07-30 00:00:00 +0000 UTC
cpu,core=1 value=30 1595995442983331000
cpu,core=1 value=31 1595995444243012000
cpu,core=1 value=32 1595995445133452000
cpu,core=1 value=33 1595995446004007000
cpu,core=2 value=40 1595995460662119000
cpu,core=2 value=41 1595995461848613000
cpu,core=2 value=42 1595995463109261000
cpu,core=2 value=43 1595995464446216000
# new shard group start: 2020-07-30 00:00:00 +0000 UTC -> end: 2020-07-31 00:00:00 +0000 UTC
cpu,core=3 value=43 1596097468378122000
cpu,core=3 value=44 1596097538490074000
cpu,core=3 value=45 1596097539967307000
# new shard group start: 2020-07-31 00:00:00 +0000 UTC -> end: 2020-08-01 00:00:00 +0000 UTC
# new shard group start: 2020-08-01 00:00:00 +0000 UTC -> end: 2020-08-02 00:00:00 +0000 UTC
# new shard group start: 2020-08-02 00:00:00 +0000 UTC -> end: 2020-08-03 00:00:00 +0000 UTC
# new shard group start: 2020-08-03 00:00:00 +0000 UTC -> end: 2020-08-04 00:00:00 +0000 UTC
cpu,core=3 value=46 1596442837735645000
cpu,core=3 value=47 1596442840047650000
cpu,core=3 value=48 1596442890970384000
cpu,core=4 value=50 1596442922010383000
cpu,core=4 value=51 1596442922785631000
cpu,core=4 value=52 1596442924218587000
cpu,core=4 value=53 1596442925449395000
# new shard group start: 2020-08-04 00:00:00 +0000 UTC -> end: 2020-08-05 00:00:00 +0000 UTC
# new shard group start: 2020-08-05 00:00:00 +0000 UTC -> end: 2020-08-06 00:00:00 +0000 UTC
# new shard group start: 2020-08-06 00:00:00 +0000 UTC -> end: 2020-08-07 00:00:00 +0000 UTC
# new shard group start: 2020-08-07 00:00:00 +0000 UTC -> end: 2020-08-08 00:00:00 +0000 UTC
# new shard group start: 2020-08-08 00:00:00 +0000 UTC -> end: 2020-08-09 00:00:00 +0000 UTC
# new shard group start: 2020-08-09 00:00:00 +0000 UTC -> end: 2020-08-10 00:00:00 +0000 UTC
```

如上可见原来的数据被分配到了不同的 shards，因为只有少量的几条数据，所以有些 shards 区间没有分配到数据。

--format 选项指定数据输出的格式，默认为如上的行输出。所有格式为:

## index-tsi

为 tsm 数据建立 tsi1 索引。

--data-dir: 数据库 data 目录，如 ~/.fctsdb/data

--wal-dir: 数据库 wal 目录，如 ~/.fctsdb/wal

--database: 指明对什么数据库建索引，若不指定则对 --data-dir 下所有数据库建索引

--rp: 指明对什么 retention policy 建索引，若不指定则对 --database 下的所有 rp 建索引

--verbose: 打印详细的过程信息

示例如下:

```Shell
./fctools index-tsi --database mydb --rp autogen --data-dir ~/.fctsdb/data --wal-dir ~/.fctsdb/wal --verbose
2020-08-04T05:50:53.938093Z     info    Rebuilding database     {"log_id": "0OQ0Cix0000", "name": "mydb"}
2020-08-04T05:50:53.940918Z     info    Rebuilding retention policy     {"log_id": "0OQ0Cix0000", "db_instance": "mydb", "db_rp": "autogen"}
2020-08-04T05:50:53.941414Z     info    Rebuilding shard        {"log_id": "0OQ0Cix0000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
2020-08-04T05:50:53.941427Z     info    Checking index path     {"log_id": "0OQ0Cix0000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/1/index"}
2020-08-04T05:50:53.941426Z     info    Rebuilding shard        {"log_id": "0OQ0Cix0000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T05:50:53.941437Z     info    Checking index path     {"log_id": "0OQ0Cix0000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/12/index"}
2020-08-04T05:50:53.941441Z     info    tsi1 index already exists, skipping     {"log_id": "0OQ0Cix0000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/1/index"}
2020-08-04T05:50:53.941453Z     info    tsi1 index already exists, skipping     {"log_id": "0OQ0Cix0000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/12/index"}
```

输出显示 mydb.autogen 的索引已经存在，所以命令跳过执行。可以将 mydb.autogen 的索引手动删除然后重新执行命令:

```Shell
rm -rf ~/.fctsdb/data/mydb/autogen/1/index
rm -rf ~/.fctsdb/data/mydb/autogen/12/index
```

```Shell
 ./fctools index-tsi --database mydb --rp autogen --data-dir ~/.fctsdb/data --wal-dir ~/.fctsdb/wal --verbose

2020-08-04T06:04:39.637422Z     info    Rebuilding database     {"log_id": "0OQ0~7Kl000", "name": "mydb"}
2020-08-04T06:04:39.638523Z     info    Rebuilding retention policy     {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen"}
2020-08-04T06:04:39.638651Z     info    Rebuilding shard        {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T06:04:39.638664Z     info    Rebuilding shard        {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
2020-08-04T06:04:39.638676Z     info    Checking index path     {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/12/index"}
2020-08-04T06:04:39.638678Z     info    Checking index path     {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/1/index"}
2020-08-04T06:04:39.638703Z     info    Opening shard   {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
2020-08-04T06:04:39.638707Z     info    Cleaning up partial index from previous run, if any     {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
2020-08-04T06:04:39.638708Z     info    Opening shard   {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T06:04:39.638713Z     info    Cleaning up partial index from previous run, if any     {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T06:04:39.638777Z     info    Opening tsi index in temporary location {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/1/.index"}
2020-08-04T06:04:39.639280Z     info    Opening tsi index in temporary location {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/12/.index"}
2020-08-04T06:04:39.645735Z     info    index opened with 8 partitions  {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1, "index": "tsi"}
2020-08-04T06:04:39.645872Z     info    Iterating over tsm files        {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
2020-08-04T06:04:39.645879Z     info    Processing tsm file     {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/1/000000005-000000002.tsm"}
2020-08-04T06:04:39.646008Z     info    index opened with 8 partitions  {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12, "index": "tsi"}
2020-08-04T06:04:39.646123Z     info    Iterating over tsm files        {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T06:04:39.646144Z     info    Processing tsm file     {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12, "path": "/Users/musenwill/.fctsdb/data/mydb/autogen/12/000000001-000000001.tsm"}
2020-08-04T06:04:39.646364Z     info    Building cache from wal files   {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
2020-08-04T06:04:39.646437Z     info    Reading file    {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1, "service": "cacheloader", "path": "/Users/musenwill/.fctsdb/wal/mydb/autogen/1/00000000000000000011.log", "size": 213}
2020-08-04T06:04:39.646538Z     info    Building cache from wal files   {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T06:04:39.646575Z     info    Reading file    {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12, "service": "cacheloader", "path": "/Users/musenwill/.fctsdb/wal/mydb/autogen/12/00000000000000000001.log", "size": 497}
2020-08-04T06:04:39.646766Z     info    File corrupt    {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1, "service": "cacheloader", "error": "snappy: corrupt input", "path": "/Users/musenwill/.fctsdb/wal/mydb/autogen/1/00000000000000000011.log", "pos": 0}
2020-08-04T06:04:39.646770Z     info    File corrupt    {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12, "service": "cacheloader", "error": "snappy: corrupt input", "path": "/Users/musenwill/.fctsdb/wal/mydb/autogen/12/00000000000000000001.log", "pos": 0}
2020-08-04T06:04:39.646853Z     info    Iterating over cache    {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
2020-08-04T06:04:39.646878Z     info    Iterating over cache    {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T06:04:39.647032Z     info    compacting index        {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T06:04:39.647049Z     info    Closing tsi index       {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T06:04:39.647115Z     info    compacting index        {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
2020-08-04T06:04:39.647131Z     info    Closing tsi index       {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
2020-08-04T06:04:39.647263Z     info    Moving tsi to permanent location        {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 12}
2020-08-04T06:04:39.647417Z     info    Moving tsi to permanent location        {"log_id": "0OQ0~7Kl000", "db_instance": "mydb", "db_rp": "autogen", "db_shard_id": 1}
```

从输出可见索引已重新建立，`~/.fctsdb/data/mydb/autogen/1/index` 和 `~/.fctsdb/data/mydb/autogen/12/index` 目录下的索引文件也已创建。

## index-seriestime

根据时间对 series 建立索引。

--data-dir: 数据库 data 目录，如 ~/.fctsdb/data

--wal-dir: 数据库 wal 目录，如 ~/.fctsdb/wal

--database: 指明对什么数据库建索引，若不指定则对 --data-dir 下所有数据库建索引

--rp: 指明对什么 retention policy 建索引，若不指定则对 --database 下的所有 rp 建索引

--verbose: 打印详细的过程信息

示例如下:

```Shell
./fctools index-seriestime --data-dir ~/.fctsdb/data --wal-dir ~/.fctsdb/wal --verbose
2020-08-04T06:15:29.610333Z     info    Rebuilding database     {"log_id": "0OQ1bnI0000", "name": "_fctsdb"}
2020-08-04T06:15:29.613154Z     info    Rebuilding retention policy     {"log_id": "0OQ1bnI0000", "db_instance": "_fctsdb", "db_rp": "autogen"}
2020-08-04T06:15:29.613282Z     info    Rebuilding shard        {"log_id": "0OQ1bnI0000", "db_instance": "_fctsdb", "db_rp": "autogen", "db_shard_id": 10}
......
```

在 `~/.fctsdb/data/mydb/autogen/1` 和 `~/.fctsdb/data/mydb/autogen/12` 目录下，生成了 timeindex 目录保存 time 至 series 的索引。

## upgrade

平滑升级工具，通过监控信息实时监测主从数据复制状况，当主从数据同步完成（差异为0），将设置的从节点角色变更为主节点，对外提供服务。

--new_leader_addr     新的主节点http地址

--addrs                      复制集的所有节点http地址

--interval                   主从复制状况监测周期

--do-change              为false时，只打印主从同步延迟      

--username               数据库用户名（主从数据库一致）

--password               数据库密码（主从数据库一致）



示例如下:

```Shell
./fctools upgrade --new_leader_addr 127.0.0.1:18086 --addrs 127.0.0.1:8086 --addrs 127.0.0.1:18086 -do-change true
nodes: [127.0.0.1:8086 127.0.0.1:18086]
new leader: 127.0.0.1:18086
detect interval: 5s
do change: true
current leader address: 127.0.0.1:8086
current rpc replication address: 127.0.0.1:8090
new rpc replication address: 127.0.0.1:18090
[max replica lag shard] shard id: 3      replica id: 1604026337938411751  replica address: 127.0.0.1:33390       
[past]    ci: 1497        replica ci: 1491        replica lag: 6
[now ]    ci: 1563        replica ci: 1557        replica lag: 6
total 2 shards replicating

set leader 127.0.0.1:8086 readonly
[max replica lag shard] shard id: 4      replica id: 1604026337938411751  replica address: 127.0.0.1:33392       
[past]    ci: 1566        replica ci: 1561        replica lag: 5
[now ]    ci: 1566        replica ci: 1566        replica lag: 0
total 2 shards replicating

set 127.0.0.1:18086 as leader
set new leader 127.0.0.1:18086 writable
upgrade finished
```

整个升级过程分为3个步骤：

1. 监测主从同步延迟；
1. 当主从同步延迟足够小时，设置主节点为只读
1. 由于主停止写入，主从同步延迟为0，设置新的主节点

## instruction

指令执行工具，调用海东青数据库通过http接口暴露的命令API。

子命令：

set-readonly  设置节点只读

set-role         设置节点复制主从角色

可以输入 `instruction help [subcommand]` 来查看子命令的使用方法，如 `instruction help set-role`。



子命令使用实例：

1. set-readonly

参数：

set-readonly 节点只读开关 [true/false]

示例

```Shell
./fctools instruction  --addr [addr] --username [username] --password [password] set-readonly [true/false]
```

1. set-role

参数：

role    节点新角色 [leader/follower/learner]

remote-rpc 节点复制源（设置follower时需要）

示例

```Shell
./fctools instruction  --addr [addr] --username [username] --password [password] set-role --role [role] --remote-rpc [remote-rpc]
```

# 数据库备份和恢复

海东青数据库支持 db，rp，shard 粒度的数据备份还原，并支持设置备还原的数据时间段。下载该工具：[https://fctsdb.rockontrol.com/fctools](https://fctsdb.rockontrol.com/fctools)

数据库备份恢复从备份原理上可以分为物理备份和逻辑备份。在此基础之上，物理备份还分为全量备份和增量备份。

物理备份通常是采用快照的方式直接获得数据文件的镜像，因此可以实现高性能快速备份，但通常只能做文件级别等较粗粒度的数据备份；而逻辑备份通常通过读取数据的方式，得到数据的逻辑表达，备份性能较差，但备份的粒度可以做到更精细。

全量备份对整个数据库进行数据备份，而增量备份则会在历史备份基础之上，仅仅备份上一次备份之后数据有变化的 shard。

为描述简便，在没有特别说明的情况下，本文所描述的数据库备份/恢复均指全量物理备份/物理恢复。

## 

## 数据库备份

海东青数据库使用fctools backup命令进行备份。海东青数据库备份采用了 LSM-tree 的物理备份机制，通过对磁盘文件进行镜像，实现快速高性能备份。

```Shell
# ./fctools backup --help
NAME:
   fctools backup - Backup tool for fctsdb

USAGE:
   fctools backup [command options] [arguments...]

DESCRIPTION:
   Creates a backup copy of specified FalconTSDB database(s) and saves the files to PATH (directory where backups are saved).

OPTIONS:
   --host value      FalconTSDB host to back up from. Optional. Defaults to 127.0.0.1:8088 (default: "localhost:8088")
   --username value  Specify the user name
   --password value  Specify the user password
   --db value        FalconTSDB database name to back up. Optional. If not specified, all databases are backed up.
   --rp value        Retention policy to use for the backup. Optional. If not specified, all retention policies are used by default.
   --shard value     The identifier of the shard to back up. Optional. If specified, '-rp <rp_name>' is required.
   --start value     Include all points starting with specified timestamp (RFC3339 format). e.g. 2015-12-24T08:12:23Z
   --end value       Exclude all points after timestamp (RFC3339 format). e.g. 2015-12-24T08:12:23Z
   --skip-errors     Optional flag to continue backing up the remaining shards when the current shard fails to backup.
   --increment       Optional flag to do incremental backup.
```

**参数说明**

* host：待备份的数据库备份服务监听地址，通常而言端口号是 8088
* user：待备份的数据库用户名
* password：待备份的数据库密码
* db：待备份的 db 名,如果不指定，将备份全库
* rp：待备份的 rp 名，如果不指定rp名，将备份所有的rp
* shard：待备份的 shard,同时需要指定--db <db_name>和 -rp <rp_name> 选项
* start：待备份数据的开始时间
* end：待备份数据的结束时间
* increment：是否进行增量备份，增量备份只会备份自上一次备份以来，数据有变化的 shard

**常见示例**

1 备份整个数据库实例

```Shell
# ./fctools backup -host 127.0.0.1:8088 /data/fctsdb_backup
```

2 备份单个db

```Shell
# ./fctools backup -host 127.0.0.1:8088 -db iot_mobile /data/fctsdb_backup/iot_mobile/
```

3 备份单个shard

```Shell
./fctools backup -host 127.0.0.1:8088 --db iot_env_data --rp autogen --shard 2039 /data/fctsdb/backup/iot_env_data_shard_2039/
```

4 使用--start和--end选项备份某段时间内的数据

```Shell
./fctools backup -host 127.0.0.1:8088 --db iot_env_data --start 2019-01-01T00:00:00Z --end 2020-01-01T00:00:00Z /data/fctsdb/backup/iot_env_data_2020
```

## 数据库恢复

海东青数据库使用fctools restore命令进行恢复。基于备份得到的 LSM-tree 镜像文件，应用到存储引擎中，并重建相应的索引。

```Shell
# ./fctools restore --help
NAME:
   fctools restore - Restore tool for fctsdb

USAGE:
   fctools restore [command options] [arguments...]

DESCRIPTION:
   Uses backup copies from the specified PATH to restore databases or specific shards from FalconTSDB without user info

OPTIONS:
   --host value      FalconTSDB host to back up from. Optional. Defaults to 127.0.0.1:8088 (default: "localhost:8088")
   --username value  Specify the user name
   --password value  Specify the user password
   --db value        Name of database to be restored from the backup
   --newdb value     Name of the FalconTSDB database into which the archived data will be imported on the target system.
            Optional. If not given, then the value of '-db <db_name>' is used.  The new database name must be unique
            to the target system.
   --rp value  Name of retention policy from the backup that will be restored. Optional.
            Requires that '-db <db_name>' is specified.
   --shard value             Identifier of the shard to be restored. Optional. If specified, then '-db <db_name>' and '-rp <rp_name>' are required. (default: 0)
   --timestamp value         Specify the timestamp to restore
   --list-timestamp          List the timestamp of all backup files
   --skip-internal-database  Whether to skip restoring internal database. Default is true.
```

**参数说明**

可指定的options参数及说明如下：

* host:                        指定海东青服务器的地址(比如"localhost:8088")
* user:                        用户名
* password:                用户密码
* db:                                指定还原的源db(若未指定则表示还原备份文件中所有的db)
* newdb:                        指定还原的目标db(若为指定则使用db)
* rp:                                指定还原的源rp(若未指定则表示还原指定db下的所有rp；若指定rp则需要指定db)
* shard:                        指定还原某shard id对应的数据        
* list-timestamp:        列出所有备份的时间点
* timestamp:                指定还原某时间点的备份

**常用示例**

1 恢复指定的全库备份

```Shell
# ./fctools restore -host 127.0.0.1:8088 /data/fctsdb_backup/full
```

2 恢复指定的DB

```Shell
./fctools restore -host 127.0.0.1:8088 -db iot_env_data /data/fctsdb/backup/iot_env_data
```

3 恢复指定的shard

```Shell
./fctools restore -host 127.0.0.1:8088 -db iot_env_data -rp autogen -shard 2039 /data/fctsdb/backup/iot_env_data
```

4 使用--newdb选项将备份恢复到其他数据库中

```Shell
./fctools restore -host 127.0.0.1:8088 -db iot_env_data -newdb iot_env_data_new /data/fctsdb/backup/iot_env_data
```

# 数据库增量备份、恢复和管理

海东青数据库可以根据之前的备份来进行增量备份，增量备份只会备份2个备份之间被修改的shard。

在首次备份时，由于历史数据为空，依然会备份全量的数据。

相对于全量备份，增量备份可以缩短备份时间，减少备份占用磁盘空间，也支持多个备份时间点的数据还原。



## 数据库增量备份

海东青数据库增量备份命令依然是使用的fctools backup，除了需要打开-increment开关，参数配置可以完全参考全量备份。

```Shell
mkdir -p /data/fctsdb/backup/iot_env_data
./fctools backup -host 127.0.0.1:8088 -db iot_env_data -increment /data/fctsdb/backup/iot_env_data
```

海东青数据库使用fctools restore命令进行增量备份恢复，除了需要指定-timestamp参数设置需要恢复到的时间点以外，参数配置也可以完全参考全量备份的还原。

```Shell
./fctools restore -host 127.0.0.1:8088 -db iot_env_data --timestamp 20210820T072301Z /data/fctsdb/backup/iot_env_data
```

## 数据库增量备份数据管理

增量备份会保留若干个备份时间点的数据，对于不需要的备份点，可以通过增量备份数据管理命令fctools maintain-backup删除掉，如只保留最近4次的备份。

```Shell
# ./fctools maintain-backup --remain 4 /data/fctsdb/backup/iot_env_data
```

# 数据库逻辑备份和恢复

## 数据库逻辑备份

海东青数据库逻辑备份是使用fctools export命令，以数据文件为输入，输出行协议格式的文本文件。由于存在文件格式的转换，可能会耗时较长。通常情况下，建议使用数据库备份工具。

```Shell
# ./fctools export --help
NAME:
   fctools export - Exports raw data from a shard to line protocol

USAGE:
   fctools export [command options] [arguments...]

OPTIONS:
   --data-dir value     Data storage path (default: "/root/.fctsdb/data")
   --wal-dir value      WAL storage path (default: "/root/.fctsdb/wal")
   --database value     Database to export
   --out value          Destination file to export (default: "/root/.fctsdb/export")
   --rp value           Retention policy to export
   --measurement value  Measurement to export
   --start value        Start time to export
   --end value          End time to export
   --compress           Compress the output
   --no-ddl             Do not dump ddl schemas with the data
```

**参数说明**

* data-dir          指定数据库实例的数据目录
* wal-dir            指定数据库实例的wal目录
* database         指定待备份数据的database
* rp                   指定待备份数据的rp
* measurement    指定待备份的表名
* start                指定待备份数据的开始时间
* end                 指定待备份数据的结束时间    
* compress         压缩输出的文件
* out                  指定备份输出的文件
* no-ddl             不导出schema的ddl语句





**常用示例**

1 将iot_mobile库下的CPU表逻辑导出

```Shell
# ./fctools export --data-dir=/data/fctsdb/data/ --wal-dir=/data/fctsdb/wal/ --database iot_mobile --measurement cpu --out /data/fctsdb_backup/iot_mobile/export/cpu
```

2 使用--start和--end导出某个时间段内的数据

```Shell
./fctools export --data-dir=/data/fctsdb/data/ --wal-dir=/data/fctsdb/wal/ --database iot_env_data --measurement t_env_data_micro  --start 2020-01-01T00:00:00Z --end 2020-05-01T00:00:00Z --out /data/fctsdb/backup/iot_env_data_export/micro
```

3 --no-ddl选项，

如果使用了该选项导出的数据中将不会包含创建database的DDL，

```Shell
./fctools export --data-dir=/data/fctsdb/data/ --wal-dir=/data/fctsdb/wal/ --database iot_env_data --measurement t_env_data_micro -no-ddl  --start 2020-01-01T00:00:00Z --end 2020-05-01T00:00:00Z --out /data/fctsdb/backup/iot_env_data_export/micro
```

备份的/data/fctsdb/backup/iot_env_data_export/micro文件中将不会包含如下内容

```SQL
# DDL
CREATE DATABASE iot_env_data WITH NAME autogen
```

4 --compress选项

使用--compress选项，备份的文件会使用gzip命令进行压缩

```SQL
./fctools export --data-dir=/data/fctsdb/data/ --wal-dir=/data/fctsdb/wal/ --database iot_env_data --measurement t_env_data_micro --compress --out /data/fctsdb/backup/iot_env_data_export/micro.gz
```

## 数据库逻辑恢复

海东青数据库逻辑导入是将逻辑导出的数据进行导入，使用命令选项为fctools import。逻辑导入需要执行正常写入等一系列流程，性能相对备份还原较差。通常情况下，建议使用数据库备份恢复工具。

```Bash
# ./fctools import --help
NAME:
   fctools import - Import a previous database exported from file.

USAGE:
   fctools import [command options] [arguments...]

OPTIONS:
   --host value       Host to connect to. (default: "localhost")
   --port value       Port to connect to. (default: 8086)
   --username value   Username to connect to the server.
   --password value   Password to connect to the server.
   --unsafeSsl        Set this when connecting to the cluster using https and not use SSL verification.
   --precision value  Precision specifies the format of the timestamp:  rfc3339, h, m, s, ms, u or ns. (default: "ns")
   --socket value     Unix socket to connect to.
   --pps value        How many points per second the import will allow.  By default it is zero and will not throttle importing. (default: 0)
   --path value       Path of file to import.
   --compressed       Set to true if the import file is compressed.
```

参数说明



示例

1 导入指定的文件

```Bash
# ./fctools import --host 127.0.0.1 --path /data/fctsdb_backup/iot_mobile/export/cpu
```

2 --compressed选项

如果逻辑导出使用了--compress选项，恢复的时候需要添加--compressed选项

```Bash
./fctools import --host 127.0.0.1 --compressed --path /data/fctsdb/backup/iot_env_data_export/micro.gz
```

3 --pps选项

该选项限制每秒钟导入数据库的记录条数

```Bash
./fctools import --host 127.0.0.1 --pps 1000 --compressed --path /data/fctsdb/backup/iot_env_data_export/micro.gz
```

# 数据库冷备和恢复

## 数据库冷备

对于时序数据的历史数据，海东青数据库可以执行数据冷备将其转存到冷备介质中，从而减小当前数据库的数据量。

执行命令 `fctools cold-storage backup-config` 生成配置文件模板。

```YAML
host = "" #待执行冷备的海东青数据库地址
port = 8088 #待执行冷备的海东青数据库备份服务端口，与海东青数据库配置文件中 bind-address 保持一致。通常，使用默认值即可

http-port = 8086 #待执行冷备的海东青数据库HTTP服务端口，与海东青数据库配置文件中 http.bind-address 保持一致。通常，使用默认值即可

username = "" #数据库用户信息，需要具备数据备份权限
password = ""

database = "" #待冷备的 db，为空时会对所有 db 进行冷备
retention-policy = "" #待冷备的 rp，为空时会对所有 rp 进行冷备

#冷备数据起止时间。当 start 为 0001-01-01T00:00:00Z 时，截止 end 的所有数据都会被执行冷备。不可与 execute-for-oldest-days 同时设置。
start = 0001-01-01T00:00:00Z
end = 0001-01-01T00:00:00Z

#对最老的数据进行冷备，单位为天。不可与 start/end 同时设置。
execute-for-oldest-days = 0 

execute = false #是否执行。设置为 false 时，仅仅打印执行计划

throughput = 50331648 #网络限流，单位为 byte，通常保留默认值即可

skip-internal-database = true #是否跳过内部数据库，通常保留默认值即可

filename = "" #备份名称，为空时将使用时间戳自动生成备份名。

[file] #当备份介质为本地文件时，设置此项
dir = "" #备份保存目录
```

将配置文件中的 `execute` 配置项设置为 false，执行 `fctools cold-storage backup -config [config_file] `可查看执行计划。

```Bash
# ./fctools cold-storage backup -config ../config/cold_backup.conf
*************** Config *****************
host = "127.0.0.1"
port = 8088
http-port = 8086
username = ""
password = ""
database = ""
retention-policy = ""
start = 2000-01-01T00:00:00Z
end = 2021-09-01T00:00:00Z
execute-for-oldest-days = 0
execute = false
throughput = 50331648
skip-internal-database = true
filename = ""

[file]
  dir = "/data/fctsdb_backup/cold"
****************************************
############################## Execution Plan #####################################

Database: iot_mobile, DiskBytes: 269
RetentionPolicy: autogen,ShardID: 3, Start: 2021-08-09T00:00:00Z, End: 2021-08-16T00:00:00Z
Total Shards: 1
Total DiskBytes: 269
```

确认执行计划无误后，将`execute` 设置为true,执行` fctools cold-storage backup -config [config_file] `进行冷备.

```Bash
# ./fctools cold-storage backup -config ../config/cold_backup.conf
*************** Config *****************
host = "127.0.0.1"
port = 8088
http-port = 8086
username = ""
password = ""
database = ""
retention-policy = ""
start = 2000-01-01T00:00:00Z
end = 2021-09-01T00:00:00Z
execute-for-oldest-days = 0
execute = true
throughput = 50331648
skip-internal-database = true
filename = ""

[file]
  dir = "/data/fctsdb_backup/cold"
****************************************
backup meta filename: 20210810T060035Z.meta
backup manifest filename: 20210810T060035Z.manifest
############################## Execution Plan #####################################

Database: iot_mobile, DiskBytes: 269
RetentionPolicy: autogen,ShardID: 3, Start: 2021-08-09T00:00:00Z, End: 2021-08-16T00:00:00Z
Total Shards: 1
Total DiskBytes: 269
###################################################################################
succeeded to backing up meta, start: 2021-08-10 14:00:35.620241556 +0800 CST m=+0.017590328, cost: 8.656289ms
succeeded to backing up shard: 3, db: iot_mobile, rp: autogen, start: 2021-08-10 14:00:35.628973702 +0800 CST m=+0.026322326, cost: 10.276502ms, backup progress: 100.00%
succeeded to drop shard: 3, db: iot_mobile, rp: autogen, start: 2021-08-10 14:00:35.639568603 +0800 CST m=+0.036917188, cost: 12.605137ms, dropping progress: 100.00%
cold storage backup finished, total shards: 1, start: 2021-08-10 14:00:35.610399921 +0800 CST m=+0.007748540,cost: 41.798655ms
```

## 数据库冷备恢复

冷备数据文件格式与备份的格式兼容，将数据库冷备恢复到指定实例，使用的是fctools restore命令。

```Bash
# ./fctools restore --host 127.0.0.1:8088 /data/fctsdb_backup/cold/20210810T060035Z/
2021/08/10 14:02:36 Restoring shard 3 -> newID 4 live from backup 20210810T060035Z.s3.tar.gz, filesize: 254
2021/08/10 14:02:36 Restored shard 3 -> newID 4 live from backup, cost: 51.175702ms, skipped: 0, finished:  1, total: 1
2021/08/10 14:02:36 Restore Completed, cost: 93.945518ms
```

