# 筛选函数
筛选函数根据某一列的值进行计算，以定位到某一行，从而可以得到这一行的所有列的值。


### bottom

* 函数定义

`bottom(arg Number|string, n int64|uint64)`

* 函数说明

返回最小的n个值，相同的值优先返回较早的值。在`select into`中使用、和其他函数不一样。

* 实例

### first

* 函数定义
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

* 实例

### sample

* 函数定义

`sample() `

* 函数说明

返回`N`个随机抽样的字段值。

### top

* 函数定义

`top() `

* 函数说明

返回最大的N个字段值，相同的值优先返回最早的值。



