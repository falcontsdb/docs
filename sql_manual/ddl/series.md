# Series管理

`series`与 `database`, `retention policy`和 `measurement`不一样，不属于资源路径中的一层。但它确实是一部分数据的集合。

### 查看series

`SHOW SERIES [ON <database-name>]`

```sql
/*示例： */
show series
key
---
cpu,t1=aaa,t2=bbb
cpu,t1=axx,t2=ccc
```

### 删除series

`DROP SERIES`删除一个数据库里的一个series的所有数据，并且从索引中删除series。

该查询采用以下形式，您必须指定`FROM`子句或`WHERE`子句：

`DROP SERIES FROM <measurement_name[,measurement_name]> WHERE <tag_key>='<tag_value>'`

```sql
/*示例： */
/* 删除air measurement 下所有series */
> drop series from "air"**/* 删除air measurement 下指定tag的series */
> drop series from "air" where "city" = 'beijing'**/* 删除此数据库下（可以跨measurement），所有指定tag的series */
> drop series where "city" = 'chongqing'
```

