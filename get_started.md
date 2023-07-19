# 快速上手

本文介绍搭建单机环境，并使用`fcshell`和`MySQL shell`执行基本的数据库操作。

## 获取二进制程序

#### 1. 下载

您可以先从试用版下载地址下载各个平台的二进制包

免费版下载地址：https://github.com/falcontsdb/release/releases

#### 2. 解压

以Linux版本为例，将获取的二进制安装包放入任意目录。

在所在目录下执行解压：

```Shell
> tar -xvf fctsdb-master-nightly.tar.gz 
./
./fcshell
./fctsdb
```

解压后得到2个二进制命令行工具：`fcshell` 是命令行交互式工具、`fctsdb` 是数据库主程序。

#### 3. 启动数据库

```Shell
> ./fctsdb -password Abc_123456
```

海东青支持不指定配置文件方式启动。第一次启动数据库时可以指定`password`初始化数据库及创建超级管理员账户。
 > 如果没有指定password，那么需要运行 `fcshell`，在其中执行init进行初始化

## 数据库基本操作

下面我们分别介绍使用Fcshell和MySQL shell操作数据库。

### 链接数据库

#### **Fcshell**

```Shell
$./fcshell -username root -password Abc_123456
Connected to http://localhost:8086 version devel
fcshell devel (git:  )
> 
```

#### MySQL shell

```SQL
$ mysql -uroot -h 127.0.0.1 -p -P9000 --default-character-set=utf8 -pAbc_123456
Welcome to the MySQL monitor.  Commands end with ; or \g.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
```

### 数据库管理

1. 创建数据库
	
	`create database test`

1. 显示所有数据库
	
	`show databases`

1. 切换数据库
	
	`use test`

### 数据读写

#### **Fcshell**

```Shell
> create database test;
0 rows in set (0.02 sec)
> use test;
Using database test
> insert t_1,tag1=a,tag2=b f1=1
> insert t_1,tag1=a,tag2=b f1=1,f2=1
> insert t_1,tag1=a f1=1,f2=1
> insert t_1,tag1=a f1=1,f2=1 1
> select * from t_1 order by time desc;
name: t_1
+---------------------+----+----+------+------+
| time                | f1 | f2 | tag1 | tag2 |
+---------------------+----+----+------+------+
| 1686039453603080600 | 1  | 1  | a    |      |
| 1686039443713534900 | 1  | 1  | a    | b    |
| 1686039430746010200 | 1  |    | a    | b    |
| 1                   | 1  | 1  | a    |      |
+---------------------+----+----+------+------+
4 rows in set (0.00 sec)
>
```

上面的示例中，我们写入的表为`t_1`，并且写入了一条纳秒时间戳为`1`的数据。从示例中也可以看到，一条数据不必指定全部的`tag`和`field`。

#### **MySQL shell**

使用MySQL协议时需要手动创建数据库表，语法为：`CREATE TABLE <table_name>([<tag-name> TAG], [<field-name> <TYPE> FIELD);` 其中TYPE值可以是：int、string、bool、uint、float。

使用MySQL shell简单操作海东青的完整示例如下：

```Shell
mysql> create database test;
Query OK, 0 rows affected (0.02 sec)

mysql> use test;

Database changed
mysql> create table t_1(tag1 tag, tag2 tag, f1 int field, f2 int field);
Query OK, 0 rows affected (0.02 sec)

mysql> insert into t_1(tag1, tag2, f1) values('a', 'b', 1);
Query OK, 0 rows affected (0.17 sec)

mysql> insert into t_1(tag1, tag2, f1, f2) values('a', 'b', 1, 1);
Query OK, 0 rows affected (0.03 sec)

mysql> insert into t_1(tag1, f1, f2) values('a', 1, 1);
Query OK, 0 rows affected (0.03 sec)

mysql> insert into t_1(tag1, f1, f2, time) values('a', 1, 1, 1);
Query OK, 0 rows affected (0.05 sec)

mysql> select * from t_1 order by time desc;
+----------------------------+------+------+------+------+
| time                       | f1   | f2   | tag1 | tag2 |
+----------------------------+------+------+------+------+
| 2023-06-07 01:45:31.189339 |    1 |    1 | a    |      |
| 2023-06-07 01:45:17.842733 |    1 |    1 | a    | b    |
| 2023-06-07 01:45:02.973030 |    1 | NULL | a    | b    |
| 1970-01-01 00:00:00.000000 |    1 |    1 | a    |      |
+----------------------------+------+------+------+------+
4 rows in set (0.01 sec)

mysql>
```

海东青兼容MySQL协议，但也存在微小的差异，详情请参考[MySQL协议](./developer/mysql.md) 。



至此，我们介绍了简要的数据库基本操作，更详细的SQL使用请参考[SQL手册](./sql_manual/sql_manual_overview.md) 。