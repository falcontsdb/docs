# 数据库管理

### 创建数据库

1. `普通创建`

`CREATE DATABASE <database_name>`

```sql
/*示例： */
create database "environment"
```

数据库创建成功，或者数据库已经存在，将会返回一个空的结果。

2. 创建加密数据库

`CREATE DATABASE <database_name> encryption <ON/OFF> WITH NAME <RP_NAME> DURATION <duration> REPLICATION 1 SHARD DURATION <duration>`

```sql
create database test encryption on with name rp1 duration 1s replication 2 shard duration 2s
```

`encryption ` 从句后指定`on`表示创建加密数据库，默认为`off`。

### 删除数据库

`DROP DATABASE <database_name>`

```sql
/*示例： */
drop database "environment"
```

 `drop database`将删除数据库所属的所有资源，包括 `retention policies`, `measurement`, `series`, `continuous queries` 。

