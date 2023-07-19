# 转换函数
转换函数根据一个标量值计算得到一个标量值。

## 数学函数

---

### abs

* 函数定义

abs(arg : int64/uint64/float64)

* 函数描述

计算参数的绝对值

---

### sin

* 函数定义

sin(arg : int64/uint64/float64) 

* 返回值类型

float64

* 函数描述

计算参数的正弦值

---

### cos

* 函数定义

cos(arg : int64/uint64/float64)

* 返回值类型

float64

* 函数描述

计算参数的余弦值

---

### tan

* 函数定义

tan(arg : int64/uint64/float64)

* 返回值类型

float64

* 函数描述

计算参数的正切值

---

### asin

* 函数定义

asin(arg : int64/uint64/float64)

* 返回值类型

float64

* 函数描述

计算参数的反正弦值

---

### acos

* 函数定义

acos(x : int64/uint64/float64)

* 返回值类型

float64

* 函数描述

计算参数的反余弦值

---

### atan

* 函数定义

atan(x : int64/uint64/float64)

* 返回值类型

float64

* 函数描述

计算参数的反正切值

---

### exp

* 函数定义

exp(arg : int64/uint64/float64)

* 返回值类型

float64

* 函数描述

计算以e为底的指数函数值

---

### pow

* 函数定义

pow(arg1 : int64/uint64/float64, arg2 : int64/uint64/float64)

* 返回值类型

float64

* 函数描述

计算arg1的arg2次幂

---

### ln

* 函数定义

ln(arg : int64/uint64/float64)

* 返回值类型

float64

* 函数描述

计算以e为底的对数值

---

### log

* 函数定义

ln(arg1 : int64/uint64/float64, arg2 : int64/uint64/float64)

* 返回值类型

float64

* 函数描述

计算以arg1为底的对数

---

### floor

* 函数定义

floor(arg : int64/uint64/float64)

* 函数描述

返回参数向下取整的值

---

### round

* 函数定义

round(arg : int64/uint64/float64)

* 函数描述

返回参数向上取整的值

---

### sqrt

* 函数定义

sqrt(arg : int64/uint64/float64)

* 函数描述

返回参数的平方根

---

### ceil

* 函数定义

  ceil(arg : int64/uint64/float64)

* 函数描述

返回大于或等于参数的最小整数值

---

## 字符串函数

### concat

* 函数定义

  concat(...args)

* 函数描述

将多个参数args链接成一个字符串

---

### substr

* 函数定义

substr(str: string, start: int64, [ len: int64 ]) // len 是可选

* 函数描述

返回字符串str从start开始位置的子字符串，若指定len，则字串值取len长度。

---

### tostring

* 函数定义

tostring(arg : int64/uint64/float64/string/boolean)

* 函数描述

将参数格式化为字符串。

---

# 时间函数

### from_unixtime_nano

* 函数定义

from_unixtime_nano(arg : int64)

* 函数描述

将arg参数UNIX纳秒时间戳按照`2006-01-02 15:04:05.999999999`格式化为字符串。

---

### unix_timestamp_nano

* 函数定义

unixtime_timestamp_nano(arg: string)

* 函数描述

将arg时间字符串参数转换为对应的UNIX纳秒时间戳。依次按照 `2006-01-02 15:04:05`、`RFC3339`、`RFC3339Nano`三个格式进行尝试。

---

### auto_parse_time

* 函数定义

unixtime_timestamp_nano(arg: string/int64)

* 函数描述

若参数类型为int64，则直接返回arg。若arg类型为字符串，则行为与`unixtime_timestamp_nano`一致。

---

# 逻辑函数

### ternary

* 函数定义
* 函数描述

模拟三目运算符。若arg1值为true，则函数返回arg2，若为false则返回arg3。若arg为null，则函数返回null。

---



