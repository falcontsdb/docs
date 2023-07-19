# 数据表管理

海东青的表 `measurement`是位于 `retention policy`下一级的资源，是二维的结构。

海东青也同时支持MySQL表管理语法。

### 查看表

1. `SHOW MEASUREMENTS <measurement_name> [ON database_name]`

```sql
/*示例： */
> show measurements on test
name: measurements
name
----
air
cpu
```

2. `show tables`

```sql
mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| t_1            |
+----------------+
1 row in set (0.00 sec)

mysql>
```

3. `show create table` 查看表结构

### 删除表

1. `DROP MEASUREMENT measurement_name`

```sql
/*示例： */
> use datax
Using database datax
> drop measurement air
```

2. `DROP TABLE table_name`

```sql
mysql> drop table t_1;
Query OK, 0 rows affected (0.34 sec)

mysql>
```


### 删除measurement部分数据(指定tag)

与`DROP SERIES`不同，它不会从索引中删除series，并且它支持`WHERE`子句中的时间间隔。

该查询采用以下格式，必须包含`FROM`子句或`WHERE`子句，或两者都有：

`DELETE FROM <measurement_name> WHERE [<tag_key>='<tag_value>'] | [<time interval>]`

