# 版本兼容性升级说明

# v1.3.0升级到v1.3.1

只是若干bug修复，可直接二进制替换升级。



# v1.3.1升级到v1.3.2

此版本除了若干bug修复，还加入了慢日志配置以及可作为单独文件保存，因此多加了两个配置项。

```TOML
[coordinator]
  log-queries-after = "0s"
  slow-queries-log-file = ""
```

`log-queries-after：` 表示超过多久一个query没有执行完成就打印慢日志。

`slow-queries-log-file：` 表示慢日志保存的文件，完整路径。



将上面的配置项新加到配置文件中，替换二进制，重启即可。



# v1.3.2升级到v1.3.3

此版本除了若干bug修复和性能优化之外，还加入了延迟加载功能，数据库启动时，可以不去读取老数据，从而实现快速启动。项目可根据实际情况判断是否开启此特性，如果数据库总数据量超过200GB(即data目录大小)，建议开启此特性。开启方法如下：

```TOML
[data]
  lazy-loading-enabled = false
  hot-shards = 5
```

在配置文件的data块，加入上面两项配置。

`lazy-loading-enabled：`表示是否开启延迟加载，true即为开启

`hot-shards：`加载最近的多少个shard，默认为5



# v1.3.3升级到v1.3.4

只是若干bug修复和性能优化，可直接二进制替换升级。



# v1.3.4升级到v1.3.5

只是若干bug修复和新增增量备份工具，可直接二进制替换升级。



# v1.3.5升级到v1.3.6

只是若干bug修复和性能优化，可直接二进制替换升级。



# v1.3.6升级到v1.3.7

新增数据列类型强校验，默认开启；若干bug修复和性能优化，可直接二进制替换升级。



# v1.3.7升级到v1.3.8

优化最近时间数据查询性能；若干bug修复和性能优化，可直接二进制替换升级。



# v1.3.8升级到v1.3.9

添加window函数，可直接二进制替换升级。

# v1.3.9升级到v1.3.10

可直接二进制替换升级。

# v1.3.10升级到v1.3.11

可直接二进制替换升级。

# v1.3.11升级到v1.4.0

由于1.4.0新增预计算优化和添加隐藏列，因此属于**非兼容**升级，有两种方式进行升级。

#### 停机升级

1. 停止数据库服务
2. 执行 fctools 升级命令

  其中 src 选项为存放数据库实例的目录，dst 为转换后的数据库实例存放目录，concurrency 选项指定适当协程数量可加快处理速度。

  注意必须添加 --all 选项迁移数据库实例中的全部文件。

3. 根据下表按需修改配置文件

```yaml
新增配置项:
security.tls.min-version
security.tls.max-version
security.tls.ca-path
security.tls.cert-path
security.tls.key-path
security.tls.verify-peer
security.tls.insecure-verify

security.entryption.provider        // 密码算法提供着
security.entryption.public-key      // 存储层加密组密钥公钥 166进制
security.entryption.type            // 私钥存储类型,只支持'file'，即私钥保存在文件
security.entryption.path            // 私钥文件路径

rpc-server.tls-enable                // rpc 服务是否开启 tls 加密

http.enabled                         // 数据库是否启用 http 服务

删除配置项:
oplog-replica.auth-enabled
oplog-replica.certificate
oplog-replica.private-key

subscriber.ca-certs

http.https-certificate
http.https-private-key

tls    // tls 段全部移到 security 段之下

新增系统变量:
security-password-expire-time        // 用户密码过期时间
```

4. 启动 v1.4.0 新版本数据库程序。
5. 在 fcshell 中执行 init 指令创建超级用户(或者将原有的 admin 用户提拔为超级用户)。

#### 在线升级

1. 升级线上数据库为 v1.3.11 版本并启动(可能需要修改相应配置)。
2. fcshell 中执行命令 `set variable "data-level-compaction-disabled" = true` 关闭 level compaction 并重启数据库服务以生效。
3. 系统磁盘扩容，建议新挂载磁盘的可用空间至少为数据库实例已有数据量的 1.5 倍。
4. 执行 fctools 升级命令，将转换后的 tsm 保存至新挂载磁盘，原有的 tsm 文件保持不变，注意须指定 dst 路径至新挂载磁盘。(该步骤可能执行很长时间)

`fctools fix precompute --src [source path] --dst [dest path] --concurrency 16`

5. 停止数据库服务。
6. 再次执行 fctools 升级命令：

`fctools fix precompute --src [source path] --dst [dest path] --all --concurrency 16`

7. 根据需要修改配置文件，(参考注意事项)，重点确保如下配置项的路径正确:

```YAML
meta.dir
data.dir
data.snapshot-dir
data.wal-dir
logging.filename
license.license-path
```

8. 启动 v1.4.0 新版本数据库程序。
9. 在 fcshell 中执行 init 指令创建超级用户(或者将原有的 admin 用户提拔为超级用户)。
10. fcshell 中执行命令 set variable "data-level-compaction-disabled" = false允许 level compaction 并重启数据库服务以生效。
11. unmount 原有的数据盘作为备份

# v1.4.0升级到v1.4.1

可直接二进制替换升级。

# v1.4.1升级到v1.4.2

可直接二进制替换升级。

# v1.4.2升级到v1.5.0

1. 可直接二进制替换升级。
1. 若需要配置主从，则需要修改主从相关配置，请参考：[https://rockontrol.yuque.com/si6a0z/plw06o/xkr5wx](https://rockontrol.yuque.com/si6a0z/plw06o/xkr5wx)

# 注意事项

如果是跨多个版本（超过一个）升级，则需要一次性把上面跨多个版本的事项做完，然后重启。 注意：升级前请备份数据。

