# 加密通信

数据库为了安全考虑，需要在数据通信链路上避免数据泄密，因此海东青数据库提供SSL通信，使得在加密链路上进行安全通信，

整个系统可使用一套SSL证书，SSL证书的指定路径由如下配置来指定

```TOML
[security]
  [security.tls]
    ca-path = ""
    cert-path = ""
    key-path = ""
```

在三个通信模块中使用SSL，通信模块模块如下

1. 在API服务器里提供SSL通信。可通过设置配置文件中的http部分中的https-enabled为true来启用，并需要指定security配置模块下子配置模块tls中的ca-path，cert-path，key-path。配置举例：

```toml
[http]
  # 数据库API服务器的监听地址
  bind-address = ":8086"
  # 是否开启HTTPS通信
  https-enabled = true
[security]
  [security.tls]
    ca-path = "./ca"
    cert-path = "./cert"
    key-path = "./key"
```
2. 在主从复制的RPC通信里提供SSL通信。通过设置配置文件中rpc-server里的tls-enable为true来启用，并需要指定security配置模块下子配置模块tls中的ca-path，cert-path，key-path。配置举例： 
```toml
[rpc-server]
  bind-addr = "127.0.0.1:8090"
  tls-enable = true
[security]
  [security.tls]
    ca-path = "./ca"
    cert-path = "./cert"
    key-path = "./key"
```
3. MySQL服务可以提供SSL通信，将mysql节区中mysql.security里的`require-secure-transport`设置为true即可打开SSL加密通信。可以配置MySQL服务的SSL证书，如果。为空，则使用security配置模块中的SSL配置。
```toml
[mysql]
  # MySQL协议服务绑定的监听地址
  bind-address = ":9000"
  concurrent-count = 500
  # 是否开启MySQL协议服务
  enabled = true
  max-conn-count = 1000
  wait-timeout = 9000
  [mysql.performance]
    tcp-keep-alive = true
    tcp-no-delay = false
  [mysql.security]
    # 是否要求安全传输, 若开启，则需要配置安全证书，若没有配置则使用security中的tls配置
    require-secure-transport = false
    ssl-ca = ""
    ssl-cert = ""
    ssl-key = ""
```
