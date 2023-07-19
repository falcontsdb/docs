# 数据写入

海东青支持InfluxDB行协议语法写入和MySQL `insert into`语法写入。

1. 行协议写入

此方式只允许在InfluxDB协议中执行。写入语法为：

```sql
insert table_name[,tag1_name=tag1,...] field1_name=value1[,field2=name=value2,...] [timestamp]
```

示例：

```sql
insert t_1,tag1=a,tag2=b f1=1
```

2. `MySQL`写入方式，写入语法有两种格式：

```sql
insert into table_name(column_name1[,column_name2,...]) values (col_value1,col_value2,...)[,(col_value1,col_value2,...)...];
```

和

```sql
insert into table_name set column_name1=col_value1[,column_name2=col_value2....];
```

由于此语句属于SQL语法，因此InfluxDB协议和MySQL协议都可以使用此语句写入数据。

示例：

```sql
insert into t_1(tag1, f1, f2) values('a', 1, 1);
insert into t_1(tag1, f1, f2, time) values('a', 1, 1, 1686272495485783300);
insert into t_1(tag1, f1, f2, time) values('a', 1, 1, '2022-12-29 12:14:42.126');
```