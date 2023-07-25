# 筛选函数
筛选函数根据某一列的值进行计算，以定位到某一行，从而可以得到这一行的所有列的值。


### bottom

* 函数定义  
   `bottom(arg Number|string, n int64|uint64)`

* 函数说明  
    返回最小的n个值，相同的值优先返回较早的值。在`select into`中使用、和其他函数不一样。

* 示例
    ```sql
    > select * from data order by time asc;
    name: data
    +------+----+----+----+
    | time | f1 | f2 | t1 |
    +------+----+----+----+
    | 1    | 1  |    | 1  |
    | 2    | 3  |    | 1  |
    | 3    | 1  |    | 1  |
    | 4    |    | 0  | 1  |
    +------+----+----+----+
    4 rows in set (0.00 sec)
    > select bottom(f1,1),time from data
    name: data
    +---------------+------+
    | bottom(f1, 1) | time |
    +---------------+------+
    | 1             | 3    |
    +---------------+------+
    1 rows in set (0.00 sec)
    ```


### first

* 函数定义  
   `first(arg Number|string|bool)`

* 函数说明  
   返回字段最早的值，可能为null。

### last

* 函数定义  
  `last(arg Number|string|bool)`

* 函数说明  
  返回字段最晚的值，可能为null

### mode

* 函数定义  
`mode(arg Number|string|bool) `

* 函数说明  
返回字段中出现频率最高的值。

* 示例
    ```sql
    > select * from data order by time asc;
    name: data
    +------+----+----+----+
    | time | f1 | f2 | t1 |
    +------+----+----+----+
    | 1    | 1  |    | 1  |
    | 2    | 3  |    | 1  |
    | 3    | 1  |    | 1  |
    | 4    |    | 0  | 1  |
    | 5    | 3  |    | 1  |
    +------+----+----+----+
    5 rows in set (0.00 sec)
    > select mode(f1),time from data;
    name: data
    +----------+------+
    | mode(f1) | time |
    +----------+------+
    | 1        | 1    |
    +----------+------+
    1 rows in set (0.00 sec)
    ```

### sample

* 函数定义  
   `sample(arg Number|string|bool, n int64|uint64)`

* 函数说明  
  返回`N`个随机抽样的字段值。

### top

* 函数定义  
    `top(arg Number|string[, n int64|uint64])`

* 函数说明  
    返回最大的N个字段值，相同的值优先返回最早的值。



### cumulative_sum

* 函数定义  
  ` cumulative_sum(arg Number) `

* 函数说明  
  计算累加和，当前行之前所有行的累加值，null值不参与计算(或视为0)。


### derivative

* 函数定义  
    `derivative(arg Number [, unit int64])`

* 函数说明  
  求相邻两行的增率，如果两值中存在null，那么结果位null。第二个参数是可选参数，表示时间单位，是纳秒时间duration。 

### difference

* 函数定义  
  `difference(arg Number)`

* 函数说明  
  求相邻行的差值，如果两值中存在null，那么结果位null。

### elapsed

* 函数定义  
  `elapsed(arg Number|string|bool [, unit int64])`

* 函数说明  
  求相邻两行的时间差；第二个参数是可选参数，表示时间单位，是纳秒时间duration。

### moving_average

* 函数定义  
  `moving_average(arg Number[, n int64])`

* 函数说明  
  计算滑动平均值，当前行及前n行的平均值，null值不参与计算(或视为0)。第二个参数是可选参数，表示滑动窗口的大小，默认值位2。