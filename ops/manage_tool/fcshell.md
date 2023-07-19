# Fcshell

# 介绍

fcshell 是数据库 falcontsdb 服务的命令行客户端，能以交互模式和批量模式向数据库服务发送操作指令和查询语句。使用 `-help` 选项可以查看命令支持的选项：

```Shell
Usage of fcshell:
  -version
        Print version (default: false)
  -host 'host name'
        Host to connect to. (default: localhost)
  -port 'port #'
        Port to connect to. (default: 8086)
  -database 'database name'
        Database to connect to the server.
  -username 'username'
        Username to connect to the server.
  -password 'password'
        Password to connect to the server.  Leaving blank will prompt for password (--password '').
  -ssl
        Use https for requests. (default: false)
  -unsafeSsl
        Set this when connecting to the cluster using https and not use SSL verification. (default: false)
  -ca
        Path of ca certification
  -cert
        Path of certification
  -key
        Path of private key
  -format 'json|csv|column'
        Format specifies the format of the server responses:  json, csv, or column. (default: column)
  -precision 'rfc3339|h|m|s|ms|u|ns'
        Precision specifies the format of the timestamp:  rfc3339, h, m, s, ms, u or ns. (default: ns)
  -socket 'unix domain socket'
        Unix socket to connect to.
  -outfile
        Which file results write to. If outfile is not set, then write to stdout
  -execute 'command'
        Execute instruction and quit.

Examples:

        # Use fcshell in a non-interactive mode to query the database "metrics" and pretty print json:
        $ fcshell -database 'metrics' -execute 'select * from cpu' -format 'json'

        # Connect to a specific database on startup and set database context:
        $ fcshell -database 'metrics' -host 'localhost' -port '8086'
```

所有这些选项都是可选的，若未在命令行中指定则使用默认值。`-username` 和 `-password` 选项还可以通过环境变量设置。选项取值的优先级由高到低为: 命令行显式指定 > 环境变量配置 > 默认值。

`-username` 和 `-password` 环境变量设置示例如下:

```Bash
// set
export FCTSDB_USERNAME = "joe"
export FCTSDB_PASSWORD = "strong-password"

// unset
unset FCTSDB_USERNAME
unset FCTSDB_PASSWORD
```

# 交互模式

fcshell 默认以交互模式执行，简单执行 `fcshell` 不带任何选项，即会连接至默认的 localhost:8086 数据库:

```Bash
Connected to http://localhost:8086 version 1.3.0
fcshell 1.3.0 (git: 1.5 766af55)
> 
```

交互模式下执行 `help` 指令可以查看所有的交互命令:

```Bash
> help
Usage:
init                          enter username and password for initialize FalconTSDB instance
auth                          enter username and password for authentication
connect <host:port>           connects to another node specified by host:port
set chunk <size>              sets the size of the chunked responses. Set as 0 to reset to the default chunked size
set format <format>           specifies the format of the server responses: json, csv, or column
set precision <precision>     specifies the format of the timestamp: rfc3339, h, m, s, ms, u or ns
set outfile <filepath>        sets outfile to write selection output
use <db_name.rp_name>         sets current database and retention policy
history                       displays history commands
show settings                 outputs the current settings for the shell
show readonly                 show whether server is readonly
set readonly <bool>           set server readonly or not
show state                    show server replication state
set state <leader/follower>   set server replication state as leader or follower
append nodes 'ip:port'        add several replication nodes to server
drop nodes 'ip:port'          remove serveral replication nodes from server
show anchorpoints for shard <shardID>
clear <db/rp>                 clears settings such as database or retention policy.  run 'clear' for help
exit/quit/ctrl+d              quits program

show databases        show database names
show series           show series information
show measurements     show measurement information
show tag keys         show tag key information
show field keys       show field key information
```

交互模式下不仅可以执行所有交互命令，还能执行所有支持的 SQL 语句。

执行 `exit` 或 `quit` 或按键 `ctrl + d` 可退出 fcshell 程序；如果 fcshell 空闲超过 1 小时未执行命令，也会自行退出。

# 批量执行

fcshell 可以通过选项 `-execute` 批量执行一至多条指令而不用进入交互模式，这样可以方便编排脚本任务。例如创建数据库并插入数据示例如下:

```Bash
./fcshell -username joe -password 'strong-password' -execute '
    create database testdb
    use testdb
    insert cpu,core=1 value=1
'
```

