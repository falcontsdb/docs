# HTTP API

# API 鉴权

大多数API提供鉴权访问，若需开启鉴权，需要禁止安全模式，这可通过设置配置文件中 `security.safe-mode-enabled` 为false进行禁止：

```TOML
[security]
  # HTTP/HTTPS API服务器是否开启安全模式，在安全模式下不会启用auth鉴权认证
  safe-mode-enabled = false
```



当开启鉴权认证之后，用户访问API时需提供鉴权信息。

可以有两种方式提供鉴权：

1. 指定用户名密码

通过在HTTP URL 参数里指定u和p参数，例如：

```shell
$ curl -i -XPOST "http://localhost:8086/write?u=admin&p=abcABC123!%40%23&db=mydb&precision=s" --data-binary 'mymeas,mytag=1 myfield=90 1463683075'
```

注：其中u和p的参数需要URLEncode（因为密码里可能存在某些特殊字符）。

2. 指定Token

通过在HTTP Header里添加 `Authorization`参数，设置authorization，例如：

```shell
curl -X GET \
http://127.0.0.1:8000/query \
-H 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTMzNDMwMjcsInVzZXJuYW1lIjoiamRvZSJ9.3ZO2jZKnBDMfb6gqALvxC2PRxnxFAx3zBmVBawOcIS0' \
```

注：

1. 若同时指定了用户名密码和Token，则海东青数据库服务选择使用用户提供的用户名和密码。
1. 指定authorization Token时，请求的Header里需要添加 `Bearer ` 前缀。
1. authorization需要通过 `/auth-token` API 获得。

# API 列表

---

## auth-token

* 用途

此接口用于用户使用账户密码获取authorization，authorization可代替账户密码用于其他API访问请求中提供鉴权信息。

* HTTP方法

POST

* 请求路径

/auth-token

* 请求参数

请求参数放在body中，格式为JSON，格式如下：

```json
{
  "username":"admin",
  "password":"abcABC123!@#":
}
```
  - username指定用户名
  - password指定密码。

示例：

```shell
curl -X POST \
  http://127.0.0.1:8086/auth-token \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -d '{
        "username":"admin",
        "password":"abcABC123!@#"
}'
```

* 响应结果

此API返回JSON，格式如下：
```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTQ3MTA0ODksInVzZXJuYW1lIjoiamRvZSJ9.dgtKsmydxcWlmaXkWDnluaVDtkvblvSdLr0XlO6aSGY",
    "is_admin": true,
    "unrestricted": false,
    "expire_at": 1594710489
}
```

  - token 即为获取的authorization，可用于其他API请求时的认证信息放入其header中
  - is_admin 表示用户是否超级管理员
  - unrestricted 表示用户是否不受限任何权限，当用户为超级管理员或用户拥有所有权限时，此值为true
  - expire_at 表示此authorization的过期时间，其值为UNIX时间戳

---

## ping

* 用途

返回数据库服务基本信息。

* HTTP方法

GET

* 请求路径

/ping

* 请求参数

请求参数在HTTP query中：

|参数名    |可选值    | 用途|
| ----------- | ----------- |
|verbose |0、1、true、false|  若不为0和false，则表示输出JSON格式的详细信息，否则返回204 HTTP code。|

* 响应结果
```json
{
    "version": "devel",
    "is_initialized": true,
    "node_id": "1594695701613884000",
    "hostname": "SKY-20200327IVH",
    "role": "leader",
    "digest": ""
}
```

  - version：海东青数据库的版本
  - is_initialized：海东青数据库是否初始化，若为false，则可使用fcshell或Web控制台进行初始化
  - node_id：海东青数据库的节点ID
  - hostname：海东青数据库的所在机器的hostname
  - role：海东青数据库节点的角色，其值可为leader或follower
  - digest：海东青数据库license文件的id digest，若没有有效的license文件，其值为空

---

## init

* 用途

初始化数据库。

* HTTP方法

POST

* 请求路径

/init

* 请求参数

请求参数放在body中，格式为JSON，格式如下：
```json
{
  "username": "root",
  "password":"3454015ff4f000a4d75bea5320bd484f ",
  "bindhost":"127.0.0.1"
}
```

  - username：指定初始化用的用户名，此用户即为超级管理员
  - password：根据用户密码进行hash编码后的字符串
  - bindhost：在初始化时即设置超级管理员的授权访问地址，此用户只有在符合bindhost规则的机器下才能访问数据库。

* 响应结果

此API不返回数据。

若初始化成功，则返回HTTP code 204，否则返回400.

---

## InfluxDB query

* 用途

接口用于查询数据和管理数据库。

* HTTP方法

GET、POST

* 请求路径

/query

* 请求参数

请求参数在HTTP query中：

|参数名     |可选值     |用途|
| ----------- | ----------- |----------- |
|chunked |可选[true | <number_of_points>] |  使用流式批处理而不是单次返回所有的点数据。如果设置为true，则默认是每次返回10000个数据点。|
|db      |对于依赖数据库的请求，此参数是必须的（大多数SELECT和SHOW查询都需要此参数）     | 为此次查询设置目标数据库。|
 |rp     | 可选    |  指定查询语句操作的所属rp，若为空，则操作数据库的默认rp|
|epoch   |可选，[ns,u,µ,ms,s,m,h]   | 返回指定精度的时间戳。默认返回RFC3339格式的精度为纳秒的时间戳。 u 和 µ，都代表微妙。|
|u      | 如果没有启用认证则是可选参数。反之则是必选参数。    |    设置请求用户名。|
|p      | 如果没有启用认证则是可选参数。反之则是必选参数。    |    如果启用了认证，则使用此参数设置密码。|
|pretty  |可选，false 或 true |使用更友好的方式打印JSON输出。此设置对调试很有用，生产环境不建议使用。|
|q       |必选      |需要执行的查询语句。|

* 响应结果

```JSON
{
        "Results": [{
                "statement_id": 1,
                "Series": [{
                        "columns": ["num"],
                        "values": [
                                [1],
                                [2]
                        ]
                }],
                "Messages": null
        }]
}
```

---

## InfluxDB write

* 用途

使用此接口给一个已经存在的数据库写入数据

* HTTP方法

POST

* 请求路径

/write

* 请求参数

1. 写入请求的控制参数在HTTP query中：

|参数名  |   可选值   |  用途|
| ----------- | ----------- |----------- |
|db   |   必选   |   设置数据写入的目标数据库。|
|u    |   如果没有启用认证则是可选参数。反之则是必选。 | 设置请求用户名。|
|p    |   如果没有启用认证则是可选参数。反之则是必选。 | 如果启用了认证，则使用此参数设置密码。|
|precision  |     可选，[ns,u,ms,s,m,h]     | 给Unix时间戳设置精度。默认为ns，如果没有指定的话。|
|rp   |   可选   |   给写入的数据指定保留策略。不指定则使用默认的保留策略。|

1. 写入请求所要写入的数据在body中，格式为行协议

请求示例：

```shell
$ curl -i -XPOST "http://localhost:8086/write?db=mydb&precision=s" --data-binary 'mymeas,mytag=1 myfield=90 1463683075'
```

* 响应结果

  对于Write接口，依据Http StatusCode 判断是否写入成功。如果StatusCode值为StatusNoContent(204)或者StatusOK(200)，即为写入成功，否则表示失败，失败时 body为字符串且其中包含错误信息。

## Prometheus Write

* 用途

支持Prometheus远程写协议。Prometheus协议数据可使用此API接口写入到海东青

* HTTP方法

POST

* 请求路径

/api/v1/prom/write

* 请求参数

1. 写入请求的控制参数在HTTP query中：

|参数名  |   可选值   |  用途|
| ----------- | ----------- |----------- |
|db   |   必选  |    设置数据写入的目标数据库。|
|u    |   如果没有启用认证则是可选参数。反之则是必选。 | 设置请求用户名。|
|p    |   如果没有启用认证则是可选参数。反之则是必选。 | 如果启用了认证，则使用此参数设置密码。|
|rp   |   可选  |    给写入的数据指定保留策略。不指定则使用默认的保留策略。|

2. 写入请求所要写入的数据在body中，格式为Prometheus格式。

## Prometheus Read

* 用途

支持按照Prometheus查询协议读取海东青

* HTTP方法

POST

* 请求路径

/api/v1/prom/read

* 请求参数

1. 查询请求的控制参数在HTTP query中：

|参数名 |    可选值 |    用途|
| ----------- | ----------- |----------- |
|db     | 必选  |    设置数据写入的目标数据库。|
|u      | 如果没有启用认证则是可选参数。反之则是必选。 | 设置请求用户名。|
|p      | 如果没有启用认证则是可选参数。反之则是必选。 | 如果启用了认证，则使用此参数设置密码。|
|rp     | 可选     | 给写入的数据指定保留策略。不指定则使用默认的保留策略。|

2. 查询请求在body中，格式为Prometheus Read 协议 。



## Prometheus Metrics

* 用途

支持Prometheus直接读取海东青监控指标。

* HTTP方法

GET、POST

* 请求路径

/prom_scrape

* 请求参数

  无
* 响应

  海东青将按照Prometheus监控格式返回海东青的内置监控指标。

## Compact

* 用途

强制对某些DB或者某些Shard执行Compact操作。

* HTTP方法

GET、POST

* 请求路径

/compact

* 请求参数

|参数名     |可选值    | 用途|
| ----------- | ----------- |----------- |
|db      |可选 |     设置compact的目标数据库，若没有指定，则表示compact所有的数据库。|
|u       |如果没有启用认证则是可选参数。反之则是必选。|  设置请求用户名。|
|p       |如果没有启用认证则是可选参数。反之则是必选。|  如果启用了认证，则使用此参数设置密码。|
|rp      |可选|      设置compact的目标RP，若没有设定，则表示所有RP。|
|sids    |可选|      若没有设置，则表示所有shard，若设定，则只能是sids之内的shard才能compact。|

示例：

* 响应
成功则返回StatusOK(200)