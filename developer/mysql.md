# MySQL协议

海东青支持MySQL协议（通信协议和语法），用户可直接使用MySQL SDK或命令行工具操作海东青。但海东青并非百分百兼容MySQL语法，并且在使用上存在一些需要注意的点，本文将对这些点进行详细描述。

### 字符编码

海东青只支持UTF-8编码。若用户使用MySQL命令行链接海东青，可在链接命令后添加 `--default-character-set=utf8`。若使用程序代码链接海东青，则请按照SDK所提供的方式指定UTF8编码。

### 管理工具

海东青支持原生MySQL命令行工具，但尚不支持GUI管理程序，如不支持MySQL workbench，计划在后续进行支持。

### 时区

海东青时序数据中的时间储存是以纳秒为单位的UNIX时间戳，使用int64在物理介质中储存。此时间戳与任何时区无关。

但当海东青接收用户的时间字符串时（比如在写入或作为条件进行查询时），则需要考虑时区问题，因为它关系着海东青数据库如何将字符串转换为UNIX纳秒时间戳进行写入或查询计算。

海东青支持的MySQL协议允许按照以下两种方式设置时区：

1. 在链接DSN添加：`?servertimezone`参数进行指定，如 `?servertimezone=Asia/Shanghai`;  
1. 通过 `SET servertimezone`命令修改session会话中的时区，如 `SET servertimezone = UTC`，此命令会通知海东青以用户命令指定的时区处理时间。

需要注意两点：

1. 若用户没有进行时区设置，则海东青的MySQL协议默认采用UTC时区。
1. 海东青提供的InfluxDB协议则只采用UTC时区，不允许用户自定义设置。

### 创建数据库

可以简单的创建数据库（这与MySQL一致）：

`create database test`

也可以设置过期策略，示例语句：

`create database test with name rp1 duration 1s replication 2 shard duration 2s`

也能够设置是否加密：

`create database test encryption on with name rp1 duration 1s replication 2 shard duration 2s`

### 创建表

海东青的一条记录由time字段、多个TAG以及多个Fields构成（即字段类别分为三类 `Timestamp、TAG、FIELD`），其中，time字段有且只有一个。海东青不支持自定义索引，而是以Tag作为倒排索引，因此海东青的建表语句主要不同点在于：需要在建表语句中指定列的类别。

建表语句示例如下：

```sql
create table t(t1 string TAG, f1 int FIELD, time int timestamp)
```

即每个字段由 `名称 类型名 类别名` 组成。海东青支持的类型名、类别名和物理储存类型关系如下表所示：

|类型名 |类别名 |物理储存类型 |说明 |
|---|---|---|---|
|string |TAG 或 Field | string | |
|bool |Field | bool | |
|float |Field |float64 | |
|int |Field |int64 | |
|uint |Field |uint64 | |
|int |timestamp |int64 |储存时序数据的UNIX时间戳，单位：纳秒。海东青会根据timestamp进行时间分区储存，且能提高查询性能。 |
注意以下几点规则：

1. 只允许最多一个`timestamp`类别字段，且列名必须为time。此字段可以省略，如省略则海东青自动添加此字段。
1. Tag类别的字段的储存类型只能指定为string，且string可以进行省略。比如可以直接执行 `create table t(t1 TAG, f1 int FIELD)`
1. 建表语句必须至少包含一个`FIELD`类别的字段。

### 修改表

海东青支持添加列操作，但不支持删除列、修改列等其他操作。

SQL语法为：

```sql
alter table table_name add TAG tag_name [string],add field field_name bool|string|float|int|uint
```

示例：

```sql
alter table t add tag tag1;
alter table t add field field1 int;
alter table t add tag tag1, add field field1 int;
```

### 分组非聚合查询

不同于MySQL不支持分组非聚合查询，海东青支持分组非聚合查询。

1. 用户可执行`select f1 from table group by f2`对源数据进行分组，即同一个组的数据是连续的。
1. 当分组非聚合查询语句带有order by、limit时，表示分组内排序和limit。比如`select f1 from table group by f2 order by f3 limit 2`，表示按照f2进行分组，且每个分组内按照f3进行排序，每个分组只取两条。但若查询为聚合查询时，order by time和limit语义与MySQL则保持一致。

### 普通时间类型列

在某些需求下，用户可能需要储存多个时间字段，但海东青一张表只支持一个time类型的字段，且海东青普通列不支持Datetime等时间类型。

海东青采用采用int类别解决此需求，且建议用户将其视为纳秒精度的整数进行储存，因为：1. 后续海东青会提供时间相关函数，其参数只支持int，并采用纳秒作为精度。2. 海东青的time字段的精度即为纳秒。

海东青也支持将时间格式的字符串或now()函数写入到普通int列，海东青获取时间字符串对应的纳秒时间戳写入到int字段。比如用户写入 `2022-12-29 12:14:41.501` 到int普通列，其值为 `1672287281000000`。（当然，也支持写入字符串格式的时间到time列）

### 记录唯一性

海东青时序数据库里的一条数据记录的唯一性由：Time（精度纳秒）、Tags两部分组合而唯一指定。用户在正常写入数据的流程里，一定要小心同一数据源（tags相同）的数多条数据的时间是否相同，若写入的多条数据的Time和Tags组合相同，则会发生覆盖，只保留最后一次写入的相应列的数据。此特性需要开发者注意，但却能用于实现数据的修改，请参阅`“修改数据”`章节。

### 修改数据

海东青暂时没有提供update语句，当用户存在修改时序数据的情况时，用户可使用insert插入数据的形式进行覆盖写，以此达到修改数据的目的。

请注意：此insert语句必须携带所需要修改的数据行的所有tags和time，并指定所需修改列的值，对于不需要修改的Field列，则可以不需要放入insert语句。

以下面的一条数据举例：

```bash
time                       t1 t2 f1  f2
2022-12-29 12:14:41.501    a1 a2 100 200
2022-12-29 12:14:42.126    b1 b2 300 400
```

假设需要修改time为 2022-12-29 12:14:42.126 的那条记录中的f2为500，那么可以执行如下insert完成修改：

（即指定time和所有的tag：t1和t2，以及需要修改的字段 f2。注意：f1依然保持不变不会被修改）

```sql
insert into t(time,t1,t2,f2) values('2022-12-29 12:14:42.126', 'b1', 'b2', 500)
```

海东青规划在后续版本中实现Update语句。

### 删除数据

海东青支持delete语句，但只支持以time字段和tag列作为删除条件。

### 唯一ID

当用户希望有一个列作为唯一id时，可以使用海东青提供的``uuid``函数进行写入。（uuid采用https://github.com/google/uuid 库生成16字节的随机字符串）

比如：

```sql
insert into t(time,t1,t2,f2) values('2022-12-29 12:14:42.126', uuid(), 'b2', 500)
```

### 自增ID

海东青不支持自增ID。若用户需要一列作为唯一id时，可参考``唯一ID``章节部分的建议。

### JOIN

海东青暂时不支持JOIN。

### 事务

海东青仅在通信协议上支持事务相关语句（因为一些SDK在执行语句时可能会自动发送事务相关语句），但在海东青数据库内部并没有真正实际支持事务ACID特性。

## 开发接入

### Golang

用户可使用[MySQL Driver](https://github.com/go-sql-driver/mysql)操作海东青数据库。也可以使用 [gorm](https://github.com/go-gorm/gorm)操作海东青。

下面是使用gorm读取和写入的示例：

```go
package main

import (
   "database/sql/driver"
   "fmt"
   "time"

   "gorm.io/driver/mysql"
   "gorm.io/gorm"
)

type SQLTime struct {
   t   time.Time
   loc *time.Location
}

func (t *SQLTime) Value() (driver.Value, error) {
   var zeroTime time.Time
   //判断给定时间是否和默认零时间的时间戳相同
   if t.t.UnixNano() == zeroTime.UnixNano() {
      return nil, nil
   }
   return t.t, nil
}

func (t *SQLTime) Scan(v interface{}) error {
   if _, ok := v.([]byte); ok {
      v, err := time.ParseInLocation("2006-01-02 15:04:05", string(v.([]byte)), t.loc)
      if err != nil {
         return err
      }
      t.t = v
      return nil
   }
   return fmt.Errorf("can not convert %v to timestamp", v)
}

type Fields struct {
   // field f2
   F2 int64
}

// 对应表的类型定义
type Info struct {
   Time SQLTime
   Fields
}

func (i *Info) TableName() string {
   return "test_tab"
}

func main() {
   db, err := gorm.Open(mysql.Open("root:password@tcp(127.0.0.1:19000)/test_db"), &gorm.Config{})
   if err != nil {
      panic(err)
   }

   {
      var infos []Info
      db = db.Find(&infos)
      if db.Error != nil {
         panic(db.Error)
      }
   }

   {
      var newInfo Info
      newInfo.Time = SQLTime{t: time.Now().Local(), loc: time.UTC}
      newInfo.F2 = 1
      db = db.Create(&newInfo)
      if db.Error != nil {
         panic(db.Error)
      }
   }
}
```

有两点提示:

1. 当使用gorm库读取海东青时序数据的time字段时，可使用上段代码中的`SQLTime`类型，请设置好时区loc成员。
1. `gorm.DB` 读写操作返回的db**尽量不要放到全局作用域**，特别是在后续操作不同表的时候，因为这可能导致出现反射相关的panic。

### Java

用户请使用如下依赖：

1. [mysql-connector-java](https://github.com/go-sql-driver/mysql)，版本为：8.0.*
1. [spring-jdbc](https://mvnrepository.com/artifact/org.springframework/spring-jdbc)，版本为：5.3.*

下面是使用jdbc写入数据的示例：

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class main {

    public static void main(String[] args) {
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;

        // mysql8.0以上版本
        String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";
        String URL = "jdbc:mysql://localhost:19000/dbcase?user=root&password=123456&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC";

        try{
            Class.*forName*(JDBC_DRIVER).newInstance();     // 加载驱动
            conn = DriverManager.*getConnection*(URL);    // 建立连接
            stmt = conn.createStatement();      // 创建statement对象，用来执行sql
            stmt.execute("insert into test_tab (time,f1,f2,f3) values (1024,1024,1024,1024)");
            System.*out*.println("执行成功");

        }catch(ClassNotFoundException e){
            System.*out*.println("加载驱动异常");
            e.printStackTrace();
        }catch(SQLException | IllegalAccessException e){
            System.*out*.println("数据库异常");
            e.printStackTrace();
        }
        catch (InstantiationException ex) {System.*err*.println(ex.getMessage());}
        finally{   // 关闭资源
            System.*out*.println("执行结束");
            try {
                if (stmt != null) {
                    stmt.close();
                }
            }catch(SQLException e){
                e.printStackTrace();
            }
            try{
                if (conn != null){
                    conn.close();
                }
            }catch(SQLException e){
                e.printStackTrace();
            }
        }
    }
}
```

示例代码的Maven依赖文件为：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>test_mysql</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.29</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.20</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>

</project>
```

### C/C++

用户可使用 [Connector/C++](https://dev.mysql.com/downloads/connector/cpp/)（版本为：8.*.*）和ODBC操作海东青数据库。用户也可以使用[otl](https://otl.sourceforge.net/)库。

C语言示例为：

```c
#include <stdio.h>
#include <signal.h>
#include <fstream>

extern "C" {

#include "mysql.h"
}
using namespace std;

int main()
{
    MYSQL* mysql = mysql_init(nullptr);
    mysql_set_character_set(mysql, "utf8");

    printf("Connectiong to mysql...............");
    if (!mysql_real_connect(mysql, "127.0.0.1", "root", "123456", "dbcase", 19000, NULL, 0)) {
        fprintf(stderr, "Error: %s\r\n", mysql_error(mysql));
        return 1;
    }
    printf("connect success \n");
    //设置mysql连接的字符集
    mysql_query(mysql, "SET NAMES 'utf8'");
    mysql_query(mysql, "select * from test_tab");
    return 0;
}
```

使用otl库示例如下：

```cpp
#include <iostream>
#include <vector>
#include <stdio.h>
#include <signal.h>
#include <sys/types.h>
#include <fcntl.h>
#include <string>

#include <queue>

#include <cstring>
#include <set>
#include <stack>
#include <stdlib.h>
#include <fstream>
extern "C" {

#include "mysql.h"
}
#include <string>
using namespace std;

#include <stdio.h>
#define OTL_ODBC_MYSQL // Compile OTL 4.0/ODBC/MySQL
#define OTL_UNICODE
#include "otlv4.h"

int main()
{
    otl_connect db; // connect object
    otl_connect::otl_initialize(); // initialize the database API environment
    try {
        db.rlogon("root/123456@mysql_dsn"); // connect to the database
        std::cout << "connect success" << std::endl;

        otl_cursor::direct_exec
        (
            db,
            "drop table test_tab",
            otl_exception::disabled // disable OTL exceptions
        ); // drop table
        std::cout << "drop  success" << std::endl;
        otl_cursor::direct_exec
        (
            db,
            "create table test_tab(f1 string TAG, f2 int FIELD, time int timestamp)"
        );  // create table
        std::cout << "create table  success" << std::endl;

        otl_cursor::direct_exec
        (
            db,
            "insert into test_tab(f1, f2 , time) VALUES(uuid(),1,1)"
        );  // create table
        std::cout << "create table success" << std::endl;

        otl_cursor::direct_exec
        (
            db,
            "select * from test_tab"
        );  // select table
        std::cout << "select table success" << std::endl;
    }

    catch (otl_exception& p) { // intercept OTL exceptions
        cerr << p.msg << endl; // print out error message
        cerr << p.stm_text << endl; // print out SQL that caused the error
        cerr << p.var_info << endl; // print out the variable that caused the error
    }

    return 0;
}

```

