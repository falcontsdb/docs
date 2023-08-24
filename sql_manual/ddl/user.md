# 用户管理

## 用户管理

### 创建用户

`CREATE USER <username> WITH PASSWORD <password> [WITH BINDHOST <host>] [WITH ALL PRIVILEGES]`

```sql
/*示例： */
create user jdoe with password '1qaz!QAZ' with ALL PRIVILEGES /*创建拥有超级管理员权限的用户*/
create user tom with password '1qaz!QAZ' 
```

### 查看用户

查看所有用户:
`SHOW USERS`

查看用户权限:
`SHOW GRANTS [FOR <user-name>]`
       
```sql
/*示例：*/
> SHOW GRANTS FOR worker
Roles
-----

Source IsGlobal Has  Privilege
------ -------- ---  ---------
       true     true ALL PRIVILEGES
```

通过 SHOW GRANTS 查看当前用户权限时，不需要管理员权限；通过 SHOW GRANTS FOR <user_name> 查看任意用户（包括当前用户）均需要管理员权限。

### 删除用户

`DROP USER <username>`

```sql
/*示例：*/
drop user "jdoe"
```

### 修改用户密码

`SET PASSWORD FOR <username> = <password>`

```sql
/*示例 */
set password for "jdoe" = 'hLwuDhXymTK5Q*KwoD'
```

## 用户授权IP

为了提高数据库的安全，可以限制用户只能在某些机器上才能访问数据库。

使用如下命令即可设置用户的授权IP：

`SET BINDHOST FOR <username> = <host>`

```sql
/*示例： */
set bindhost for "admin" = '127.0.0.1'
```

查询用户绑定IP , 可通过`show users`:

`SHOW USERS [WHERE "user" = <user-name>]`

```sql
> SHOW USERS WHERE "user" = 'admin'
user  bindhost  admin
----  --------  -----
admin 127.0.0.1 true
```

更改用户绑定IP：

`SET BINDHOST FOR <username> = <host> /*<host> 支持通配符 '%'，如 192.168.2.% */`

```sql
/*示例：*/
set bindhost for "user1" = '%'
```

## 用户锁定

为了防止恶意用户暴力穷举用户的密码，需要在用户登录失败到达一定次数时锁定用户，禁止用户再次登录，直到一定时间后才解锁。这种锁定并非全局，只是锁定登录 IP，比如用户通过 IP 192.168.0.3 登录数据库连续失败而锁定，那么在锁定时间到期或手动解锁前用，户无法通过 192.168.0.3 继续登录数据库，但并不限制用户通过其他可用 IP 登录，这样用户不会被恶意锁定。



在fcshell里的测试效果如下所示：

```shell
> auth
username: admin
password:
ERR: unauthorized: {"error":"authorization failed"}

> auth
username: admin
password:
ERR: unauthorized: {"error":"user auth already locked"}

> show users
user  bindhost admin locked
----  -------- ----- ------
admin           true 192.168.0.3
```

默认失败次数为10次，默认锁定登录的时长为24小时。可以通过SHOW VARIABLES命令进行查询当前配置，如下所示：

```shell
> SHOW VARIABLES WHERE var_name = 'security-login-failure-max-count'
var_name                         var_type value restart_value range  desc
--------                         -------- ----- ------------- -----  ----
security-login-failure-max-count SYSTEM   10    10            [5,20]
> SHOW VARIABLES WHERE var_name = 'security-login-failure-lock-auth-interval'
var_name                                  var_type value   restart_value range            desc
--------                                  -------- -----   ------------- -----            ----
security-login-failure-lock-auth-interval SYSTEM   24h0m0s 24h0m0s       [1h0m0s,24h0m0s]
>
```

可以通过SET VARIABLE命令去修改它们，如下所示（示例为设置最大失败次数为5次，锁定登录时间为2小时）：

`SET VARIABLE "security-login-failure-max-count" = 5`
`SET VARIABLE "security-login-failure-lock-auth-interval" = 2h`

拥有管理员权限的用户可以手动锁定其他用户或者为其他用户解锁。

锁定用户:

`ALTER USER "jdoe" ACCOUNT LOCK`

```shell
/*示例：*/
> ALTER USER "jdoe" ACCOUNT LOCK
> SHOW USERS
user  bindhost admin locked
----  -------- ----- ------
jdoe           true  all
```

解锁:

`ALTER USER "jdoe" ACCOUNT UNLOCK`

手动锁定的用户是被全局锁定的，在手动解锁前无法通过任何 IP 地址登录；手动解锁用户也是全局的，会清除用户的一切锁定状态。

