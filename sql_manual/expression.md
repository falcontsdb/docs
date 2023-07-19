# 表达式

海东青支持如下表达式：

* `=`、`>`、`>=`、`<`、`<=`、`!=`
* `and` 、`or`
* `in`、`not in`
* `between and`、`not between and`
* `is null`、`is not null`
* `like`、`not like`

	like表达式采用[MySQL语法格式](https://dev.mysql.com/doc/refman/8.0/en/pattern-matching.html)

* `regexp`、`not regexp`

	正则语法采用[Golang正则语法](https://pkg.go.dev/regexp)