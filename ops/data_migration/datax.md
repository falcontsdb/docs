# Datax海东青版本介绍

我们的潜在客户可能正在使用其他数据库（譬如MySQL、PG等），当用户选择海东青作为储存时，极有可能存在需要将已有系统的数据迁移到海东青的需求；此外，当客户储存数据到海东青后，也可能存在需要将数据导出到其他数据系统。

因此我们需要实现海东青数据库迁移工具，以让客户更好的选择和使用海东青。通过调研决定基于Datax开源项目作为迁移工具的基础，并进行改造。

# Datax

Datax项目源于阿里巴巴，其开源地址为：[https://github.com/alibaba/DataX](https://github.com/alibaba/DataX)，其插件化的设计使得各种数据系统的迁移变得非常方便和灵活，详细介绍可点击：[https://github.com/alibaba/DataX/blob/master/introduction.md](https://github.com/alibaba/DataX/blob/master/introduction.md)。由于其没有支持海东青数据库的插件，因此基于DataX插件机制，实现了支持海东青版本的DataX。



Datax海东青版本下载地址：http://fctsdb.rockontrol.com:20080/datax/

# 海东青导入插件配置说明

fctsdbwriter 插件实现将从其他数据库读取的数据写入到海东青的功能。底层实现通过将源数据转换为行协议写入到海东青数据库。

## 特性说明

* 允许源数据不带时间列，即支持那些不带时间字段的源数据，在迁移时使用系统当前时间作为数据的时间点写入到海东青。源数据的时间字段类型支持关系数据库中的DATE、DATETIME、TIMESTAMP。
* 允许指定写入的目标retention policy，因为海东青的数据库可能具备多个retention policy，我们可以将数据表迁移到海东青中某个数据库的某个retention policy的数据表下。

## 配置参数说明

以下面的JSON配置作为举例

```JSON
{
        "job": {
                "setting": {
                        "speed": {
                                "channel": 1,
                                "bytes": -1,
        "record": -1
                        }
                },
                "content": [{
                        "reader": {
                                "name": "mysqlreader",
                                "parameter": {
                                        "username": "root",
                                        "password": "111111",
                                        "column": [
                                                "col1",
                                                "col3",
                                                "col2",
                                                "col5"
                                        ],
                                        "connection": [{
                                                "jdbcUrl": ["jdbc:mysql://127.0.0.1:3306/test?useSSL=false"],
                                                "table": [
                                                        "datax_tbl"
                                                ],
                                                "driver": "com.mysql.jdbc.Driver"
                                        }]
                                }
                        },
                        "writer": {
                                "name": "fctsdbwriter",
                                "parameter": {
                                        "connection": [{
                                                "endpoint": "http://localhost:8086",
                                                "database": "datax",
                                                "table": "datax_tb2"
                                        }],
                                        "connTimeout": 15,
                                        "readTimeout": 20,
                                        "writeTimeout": 20,
                                        "username": "jdoe",
                                        "password": "abcABC123!@#",
                                        "column": [{
                                                        "name": "user_name",
                                                        "type": "TAG"
                                                },
                                                {
                                                        "name": "time"
                                                },
                                                {
                                                        "name": "user_id",
                                                        "type": "int"
                                                },
                                                {
                                                        "name": "xxxxxxxx",
                                                        "type": "string"
                                                }
                                        ],
                                        "batchSize": 1000
                                }
                        }
                }]
        }
}
```

此示例用于实现从mysql迁移datax_tb1表里的col1,col2,col3,col4四个字段的数据迁移到海东青的datax数据库的datax_tb2表里，其中mysql的col1字段对应海东青的user_name，而col2则作为时间。



`speed`里可以配置迁移速度：其中`channel`表示并发数（只有在未配置bytes或record时才生效），`bytes `限制每个并发每秒传输的字节流大小（-1表示不限制），`record`则限制每个并发每秒读取的记录行数（-1表示不限制）。



下面详细介绍writer部分的参数，即fctsdbwriter的参数，说明如下:

|配置项     |是否必须   | 数据类型    |默认值 |    描述|
| ----------- | ----------- |----------- |----------- |----------- |
|endpoint     |   是       |string  |无       |fctsdb连接串|
|username     |   是      | string  |无       |数据源的用户名|
|password    |    否       |string  |无       |数据源指定用户名的密码|
|database    |    是      | string  |无       |写入的目标数据库|
|table   |是      | string  |无       |要写入的表（指标）|
|column  |是       |list    |无       |所配置的表中需要同步的列名集合|
|connTimeout     |否      | int     |15      |设置连接超时值，单位为秒|
|readTimeout    | 否       |int     |20      |设置读取超时值，单位为秒|
|writeTimeout    |否      | int     |20      |设置写入超时值，单位为秒|
|preSql  |否       |list    |无       |插入数据前执行的SQL语句|
|postSql |否       |list    |无       |数据插入完毕后需要执行的语句|
|retentionPolicy |否      | string  |无       |写入的目标数据库的过期策略|

`column`字段说明：

1. 指定name为time的字段即作为point的时间列，此时无需额外指定type。且我们不会约束time字段必须在column的开头。
1. 可以使用type指定字段的类型，若没有指定或没有找到匹配的类型则抛出异常。可选的type值为:"BOOL"、"STRING"、"FLOAT"、"INT"、"LONG"、"DATE"、"DOUBLE"、"DECIMAL"、"BINARY"、"TAG"。 其中"TAG"表示此字段会作为数据的tag。
1. 因为配置的每个字段都具备name属性，所以column的定义顺序并不需要与源数据创建table时指定的字段顺序一致。但**注意**：此column的顺序需要和输入的源数据reader的column顺序保持一一对应。

# 海东青导出插件配置说明

fctsdbreader插件实现读取海东青数据库的数据迁移到其他数据库。

## 特性说明

* 在导出海东青数据库时，支持确定性的指定column列顺序（因为海东青读取的数据结果默认采用列名的字典排序）。如果column为空则表示读取全部列。（注意：不支持手动在column里指定` * `来读取数据）
* 可以指定读取海东青数据库里某retention policy里的数据
* 可以通过指定 `querySql  `实现迁移指定时间段的数据



## 配置文件说明

```JSON
{
  "job": {
    "content": [
      {
        "reader": {
          "name": "fctsdbreader",
          "parameter": {
            "column": [
              "time"
            ],
            "connection": [
              {
                "endpoint": "http://localhost:8086",
                "database": "NOAA_water_database",
                "table": "h2o_feet",
                "where": "1=1"
              }
            ],
            "connTimeout": 15,
            "readTimeout": 20,
            "writeTimeout": 20,
            "username": "fctsdb",
            "password": "fctsdb123"
          }
        },
        "writer": {
          "name": "streamwriter",
          "parameter": {
            "print": "true"
          }
        }
      }
    ],
    "setting": {
      "speed": {
        "bytes": -1,
        "channel": 1
      }
    }
  }
}
```

在此我们只关注reader部分的参数，即fctsdbreader的参数，说明如下:

|配置项    | 是否必须  |  数据类型 |   默认值  |   描述|
| ----------- | ----------- |----------- |----------- |----------- |
|endpoint       | 是      | string  | 无 |      指定数据库服务器地址|
|username       | 是      | string | 无  |     数据源的用户名|
|password       | 否      | string | 无  |     数据源指定用户名的密码|
|database       | 是      | string | 无  |     数据源指定的数据库|
|retentionPolicy |否      | string | 无  |     数据源指定的过期策略,仅在没有指定querySql时有效
|table   |否    |   string | 无   |    所选取的需要同步的表名,仅在没有指定querySql时有效|
|column  |否      | list    |无   |    所配置的表中需要同步的列名集合,仅在没有指定querySql时有效|
|connTimeout    | 否   |    int |    15      |设置连接超时值，单位为秒|
|readTimeout   |  否    |   int   |  20      |设置读取超时值，单位为秒|
|writeTimeout    |否   |    int |    20      |设置写入超时值，单位为秒|
|where   |否     | string | 无   |    针对表的筛选条件,仅在没有指定querySql时有效     | 
|querySql      |  否    |   string | 无    |   使用自定义的SQL而不是指定表来获取数据    |

`column`字段说明：

1. 在没有指定querySql时，根据column指定的列来读取数据。若column和querySql都没有指定,则采用*作为column来读取所有列的数据。
1. 若指定了column,则根据column指定的列名的顺序输出数据，此顺序需要和迁移的writer目标数据库的column的顺序保持一致。
1. 若没有指定column，但因为querySql返回数据列的顺序是按照字典序，所以务必检查writer目标数据库的column和实际的字典序是否一致。



