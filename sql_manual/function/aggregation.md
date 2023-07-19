# 聚合函数
聚合函数根据一组标量值计算得到一个标量值。


### **changes**

* 函数定义  

    `changes(arg int64|float64|uint64) int64`

* 函数说明  

  计算相连值变化的次数，会忽略null值。

### **count**

* 函数定义  

    `count(arg int64|float64|uint64|string|bool|*) int64`

* 函数说明  

    计算非null值的个数。`count(*)` 在运行时会被转换成`count(1)`。

### **delta**

* 函数定义  

    `delta(arg Number) Number`

* 函数说明  

    返回field values一段时间范围内最后一个时间点的值和第一个时间点值之间的差。参数和返回值具有相同的类型。

### **deriv**

* 函数定义  

    `deriv(arg int64|float64|uint64) float64`

* 函数说明  

  使用简单线性回归的方法计算field values一段时间范围内的每秒变化率。斜率**k**计算方法如下：

### **idelta**

* 函数定义  

    `idelta(arg Number) Number`

* 函数说明  

  返回field values一段时间范围内最后两个时间点的差；如果最后一个值小于前一个值，那么说明发生了重置，直接返回最后一个时间的值。

### **increase**

* 函数定义  

    `increase(arg Number) Number`    

* 函数说明  

  返回field values一段时间范围内最后一个时间点的值和第一个时间点值之间的差；如果时间范围内数据出现了下降，那么会认为出现了重置。会忽略NULL行。

* 示例



### **integral**

* 函数定义  

    `integral(arg Number, [step Time]) float64`

* 函数说明  

  返回字段曲线下的面积，即是积分。可以指定第二个参数作为时间精度, 默认精度为1s。

* 实例

### **irate**

* 函数定义  

    `irate(arg Number) float64`

* 函数说明  

  返回field values一段时间范围内最后两个时间点的数据的每秒变化率；如果最后一个值小于前一个值，那么说明发生了重置，直接用最后一个值除以最后两个时间差。

### **max**

* 函数定义  

    `max(arg Number) Number`

* 函数说明  

  返回字段的最大值。

### **mean**

* 函数定义  

    `mean(arg Number) float64`

* 函数说明  

  返回字段的平均值，不会统计null。

### **median**

* 函数定义  

    `median(arg Number) float64`

* 函数说明  

  返回字段的中位数， 数据个数为偶数时，取中间两个值的平均数。

### **min**

* 函数定义  

    `min(arg Number) Number`

* 函数说明  

  返回字段的最小值，不会统计null。

### **percentile**

* 函数定义  

    `percentile(arg Number, perc Number) float64`。

* 函数说明  

  返回字段处于perc位置的值。先将字段所有值排序，然后返回处于perc位置处的值；perc的取值范围是`[0, 100]`, 如果perc位置没有命中字段值，则会根据最近两个数据点进行线性拟合。

### **quantile**

* 函数定义  

    `quantile(arg Number, perc float64) float64`

* 函数说明  

  等价于`percentile`, 只是`quantile`的perc取值范围为 `[0, 1]`。

### **rate**

* 函数定义  

    `rate(arg Number) float64`

* 函数说明  

  返回字段一段时间范围内最后一个时间点的值和第一个时间点值之间的每秒变化率；如果时间范围内数据出现了下降，那么会认为出现了重置。

* 示例

### **spread**

* 函数定义  

    `spread(arg Number) Number`

* 函数说明  

  返回字段中最大和最小值的差值。

### **stddev**

* 函数定义  

    `stddev(arg Number) flat64`

* 函数说明  

  返回字段的标准差。

### **sum**

* 函数描述  

   `sum(arg Number) Number `

* 函数说明

  对字段求和。

