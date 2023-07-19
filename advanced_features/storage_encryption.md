# 存储层加密

海东青数据库支持存储层静态加密，即数据落盘时自动加密，被加载时自动解密，可防止通过磁盘拷贝、数据备份等方式获取数据库明文数据。

加密算法采用国密算法 SM2(非对称密钥算法) 和 SM4(分组加密算法)，支持软件加密和硬件加密。使用二级密钥管理，即每个 Shard 一个 SM4 密钥，称为子密钥，对该 Shard 的数据进行分组加密，而所有子密钥都被一个 SM2(主密钥)加密后存储在数据库中。

# 配置

存储层加密的相关配置在配置文件的 `security.encryption` 段，字段描述如下：

provider: 指定加密算法提供商，目前支持 `gm-rk` 和 `gm-soft` 两种算法提供商，其中 `gm-rk` 是罗克佳华研发的硬件加密卡，`gm-soft` 是软件加密。

public-key: SM2 公钥，遵循 《SM2 密码算法使用规范》定义的格式 `04 || X || Y`，并使用十六进制编码。

type: 若算法提供商为 `gm-soft`，则 type 字段指明私钥的获取方式，目前只支持配置为 `file`，即从本地文件读取私钥。若算法提供商为 `gm-rk`，则该字段不用填。

path: 若 type 字段指定了从本地文件读取私钥，则 path 字段指定该文件的路径。



除了配置文件的配置，海东青数据库启动时还会从环境变量 `FCTSDB_PINCODE` 读取 pincode，因为不管怎么获取私钥，都必须通过 pincode 解密才能得到私钥的授权。环境变量 `FCTSDB_PINCODE`建议在数据库启动前设置，在数据库成功启动后立即删除，可防止 pincode 泄露。

# 密钥生成

若是使用硬件加密，则主密钥由硬件事先生成，并将公钥导出。若是使用软件加密，则可由 fctools 工具生成密钥。

```shell
fctools gm genkeypair
pincode:
confirm:
public key:   04dfe998c7c5febbfe7784e0c19bcd3157206152602b7268bf1dfb6b74aef6aa0e84459854298b8357b0635e1f951bc9c1530ea01c87ad562dcbe8b4cc64b10238
private key:  eecf2cb83d7e8f568732389eb49894d347db0c6dc3eec83e3529b7f49d031d67
note: private key is encrypted by pincode
```

`fctools gm genkeypair`子命令会通过交互式方式请求输入 pincode 并确认，随后打印生成的十六进制公钥和私钥，其中私钥是被 pincode 加密后的密文。管理员可将公钥配置在海东青配置文件的 public-key 字段，将私钥写入本地文件存储，同时保管好 pincode。

# 数据库加密

海东青数据库可在数据库级别指定该数据库是否被加密，如：

```sql
CREATE DATABASE noenc
CREATE DATABASE enc WITH ENCRYPTION ON
```

数据库 noenc 是不加密的，而 enc 数据库是加密的，可以通过 `SHOW DATABASES` 查看：

```sql
> show databases
name: databases
name    encrypted
----    ---------
_fctsdb false
noenc   false
enc     true
```

在创建加密数据库时，若加密相关的配置没有指定，会创建失败；同样的，在读写加密数据库时，若加密相关的配置没有指定，也会失败。

# 更换主密钥

在主密钥使用足够长时间后(比如一年)，或者怀疑主密钥已被泄露，管理员可通过 fctools 工具更换数据库的主密钥。

```shell
fctools envelop --pubkey "xxx" --prikey "xxx" --expubkey "xxx" [--hardware] path
```

`fctools envelop` 子命令需要的参数如下：

--pubkey 选项：当前公钥的十六进制字符串

--prikey 选项：当前私钥密文的十六进制字符串

--expubley 选项：新的公钥的十六进制字符串

--hardware 选项：当前的加密算法提供商是否是硬件

path 参数：数据库实例的 metadata 路径

`fctools envelop` 子命令还会交互式的提示输入当前私钥对应的 pincode。



`fctools envelop` 子命令会读取 metadata 中所有 shards 的子密钥，用当前的私钥解密，接着用 expubkey 公钥加密。

最后还需要用 expubkey 相关的信息更新配置文件中存储层加密配置，主密钥更换才算做完。

