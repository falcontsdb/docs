# 术语介绍

### server

一个运行FalconTSDB的服务器，可以是虚拟机也可以是物理机。每个server上应该只有一个FalconTSDB的进程。

### database

database是数据根据逻辑业务进行划分。一个server下可以有多个database。

### retention policy(RP)

FalconTSDB数据结构的一部分，描述了FalconTSDB保存数据的长短(duration)，以及shard 的时间范围(shard duration)。RPs在每个database里面是唯一的，连同measurement和tag set定义一个series。

当你创建一个database的时候，FalconTSDB会自动创建一个叫做autogen的retention policy，其duration为永远，shard 的duration设为七天。

### schema

schema描述数据在FalconTSDB里的逻辑结构。FalconTSDB的schema的基础是database，retention policy，series，measurement，tag key，tag value以及field keys。

### duration

retention policy中的一个属性，决定FalconTSDB中数据保留多长时间。在duration之前的数据会自动从database中删除掉。

### measurement

measurement的概念与关系数据库中的table一样，表示一个数据表。

### shard

shard包含实际的编码和压缩数据，并由磁盘上的TSM文件表示。每个shard包含一组特定的series，每个shard可包含多个measurement的数据。

### shard duration

shard duration决定了每个shard跨越多少时间。具体间隔由retention policy中的`SHARD DURATION`决定。

例如，如果retention policy的`SHARD DURATION`设置为1w，则每个shard group将跨越一周，并包含时间戳在该周内的所有点。

### tag

FalconTSDB数据结构中的键值对，tags在FalconTSDB的数据中是可选的，但是它们可用于存储常用的metadata; tags会被索引，因此tag上的查询是很高效的。

### tag key

组成tag的键值对中的键部分，tag key是字符串，存在metadata中。

### tag set

数据点上tag key和tag value的集合。

### tag value

组成tag的键值对中的值部分，tag value是字符串，存在metadata中。

### timestamp

数据点关联的日期和时间，在FalconTSDB里的所有时间都是UTC的。

### field

FalcoTSnDB数据中记录metadata和真实数据的键值对。fields在FalconTSDB的数据结构中是必须的且不会被索引。如果要用field做查询条件的话，那就必须遍历所选时间范围里面的所有数据点，这种方式对比与tag效率会差很多。

### field key

组成field的键值对里面的键的部分。field key是字符串且保存在metadata中。

### field set

数据点上field key和field value的集合。

### field value

组成field的键值对里面的值的部分。field value才是真正的数据，可以是字符串，浮点数，整数，布尔型数据。一个field value总是和一个timestamp相关联。

field value不会被索引，如果要对field value做过滤，那就必须遍历所选时间范围里面的所有数据点，这种方式查询效率比用tag查询会差很多。

### aggregation

这是一个FQL的函数，可以返回一堆数据的聚合结果，可以看FQL函数中现有的以及即将支持的聚合函数列表。

### batch

用换行符分割的数据点的集合，这批数据可以使用HTTP请求写到数据库中。用这种HTTP接口的方式可以大幅降低HTTP的负载。尽管不同的场景下更小或更大的batch可能有更好的性能，建议每个batch的大小在1000~5000个数据点。

### continuous query（CQ）

一个在数据库中自动周期运行的FQL的查询。continuous query在`select`语句里需要一个函数，并且一定会包含一个`GROUP BY time()`的语法。

### function

包括FalconTSDB中的聚合，查询和转换，可以在FQL函数中查看FalconTSDB中的完整函数列表。

### identifier

涉及continuous query的名字，database名字，field keys，measurement名字，retention policy名字，subscription 名字，tag keys以及user名字的一个标记。

### line protocol

写入FalconTSDB时的数据点的文本格式。

### metastore

包含了系统状态的内部信息。metastore包含了用户信息，database，retention policy，shard metadata，continuous query以及subscription。

### node

一个独立的fctsdb进程。

### now()

本地服务器的当前纳秒级时间戳。

### point

FalconTSDB数据基本单元，每个数据点对应特定的series的特定field，series、field加上timestamp是数据点的唯一标识。

对于同一series中的相同field具有相同timestamp的点，新写入的point会覆盖旧的point值。

### query

从FalconTSDB里面获取数据的一个操作。

### replication factor

暂无实际作用，保持默认即可。

### selector

一个FQL的函数，从特定范围的数据点中返回一个点。可以看FQL函数中现有的以及即将支持的selector函数列表。

### series

FalconTSDB数据结构的集合，一个特定的series由measurement，tag set和retention policy组成。

注意：field set不是series的一部分

### series cardinality

在FalconTSDB实例上唯一database，measurement和tag set组合的数量。

例如，假设一个FalconTSDB实例有一个单独的database，一个measurement。这个measurement有两个tag key：email和status。如果有三个不同的email，并且每个email的地址关联两个不同的status，那么这个measurement的series cardinality就是6(3\*2=6)：

	
| email      | status  |
| :--------  | :-----  |
|lorr@fctsdb.com  |	start  |
|lorr@fctsdb.com  |	finish |
|marv@fctsdb.com  |	start  |
|marv@fctsdb.com  |	finish |
|cliff@fctsdb.com |	start  |
|cliff@fctsdb.com |	finish |



注意到，因为所依赖的tag的存在，在某些情况下，简单地执行该乘法可能会高估series cardinality。 依赖的tag是由另一个tag限定的tag并不增加series cardinality。 如果将tagfirstname添加到上面的示例中，则系列基数不会是18（3*2*3 = 18）。 它将保持不变为6，因为`firstname`已经由`email`覆盖了：

|email	| status |	firstname |
| :--------  | :-----  |:-----  |
|lorr@fctsdb.com  |	start	|   lorraine |
|lorr@fctsdb.com  |	finish  |	lorraine |
|marv@fctsdb.com  |	start	|   marvin   |
|marv@fctsdb.com  |	finish	|   marvin   |
|cliff@fctsdb.com |	start	|   clifford |
|cliff@fctsdb.com |	finish	|   clifford |


### transformation

一个FalconTSDB的函数，返回一个值或是从特定数据点计算后的一组值。但是不是返回这些数据的聚合值。

### tsm(Time Structured Merge tree)

FalconTSDB的专用时序数据存储格式。 TSM可以比现有的B+或LSM树实现更大的压缩和更高的写入和读取吞吐量。

### user

在FalconTSDB里有两种类型的用户：

* admin用户对所有数据库都有读写权限，并且有管理查询和管理用户的全部权限。
* 非admin用户有针对database的可读，可写或者二者兼有的权限。

当认证开启之后，FalconTSDB只执行使用有效的用户名和密码发送的HTTP请求。

### values per second

对数据持续到FalconTSDB的速率的度量，写入速度通常以values per second表示。

要计算每秒速率的值，将每秒写入的点数乘以每点存储的值数。 例如，如果这些点各有四个field，并且每秒写入batch是5000，那么values per second是[4个field]*[batch 5000]*[10个batch/秒]=每秒200,000个值。

### wal(Write Ahead Log)

最近写的点数的临时缓存在磁盘上的持久化镜像。FalconTSDB首先将数据写入到cache中，当cache总大小或最后一次写入时间超过阈值，则触发flush到持久化的TSM文件中。为了实现内存cache数据的持久化，在写入cache前引入写WAL。当发生重启等异常时，通过回放WAL数据得到完整的cache。

由于WAL的写入是追加顺序写，可以充分利用磁盘性能，代价总体较低而可以接受。

