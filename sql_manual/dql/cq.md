# 连续查询

连续查询语法格式为：

```SQL
create continuous query cq_name on db_name begin select_stmt end;
```

其中要求select_stmt语句中，需要还有Into从句。而select_stmt语句详见[数据查询](query) 



