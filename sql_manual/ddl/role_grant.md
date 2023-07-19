# 角色权限管理

权限保证业务的隔离性、实用性与安全性，降低数据库的使用与运维成本。

### 权限列表

  可以用 `SHOW PRIVILEGES`查看所有权限的说明。

* **ALL | ALL PRIVILEGES**

所属资源的所有权限，如果没有指定资源，则拥有所有资源的所有权限，即超级管理员权限。

* **SHOW USERS**

查看用户信息: `SHOW USERS /  GRANTS`

* **CREATE USER**

管理用户，包括创建用户、删除用户、用户授权等权限:

```SQL
CREATE/ALTER/DROP USER 
SET PASSWORD/BINDHOST 
```

* **SHOW ROLES**

查看角色信息和权限信息:

```SQL
SHOW ROLES                      -- 查看所有的角色信息
SHOW PRIVILEGES                 -- 查看所有的权限说明
SHOW PRIVILEGES FOR <role>      -- 查看某角色的权限
```

* **CREATE ROLE**

管理角色，包括创建角色、删除角色、角色授权等权限：

```SQL
CREATE/ALTER/DROP  ROLE ...
```

* **GRANT**

管理授权权限: `GRANT / REVOKE` 

* **SHOW DATABASES**

查看数据库信息的权限，支持的SQL语句：

```SQL
SHOW   DATABASES / RETENTION POLICIES / SERIES / MEASUREMENTS / SHARDS/ TAG KEYS / 
           TAG VALUES / FIELD KEYS / SUBSCRIPTIONS
```

* **CREATE DATABASE**

管理数据库，包括创建数据库、删除数据库权限。可以执行以下SQL:

```SQL
CREATE   DATABASE / RETENTION POLICY 
DROP   DATABASE / RETENTION POLICY / SUBSCRIPTION / SHARD
ALTER   RETENTION POLICY
```

* **SHOW SYSINFO**

查看一些系统全局信息： `SHOW VARIABLES/STATS/DIAGNOSTICS`

* **SET SYSINFO**

系统管理，包括设置系统变量、主从切换

* **AUDIT**

审计记录查询权限。

* **SHQO QUERIED**

慢查询管理

* **SHOW CQS**

 查看权限

* **CREATE CQ**

`create/drop continue queries`权限

* **INSERT**

插入数据权限

* **SELECT**

查询数据，包括 `SELECT`和 `EXPLAIN`语句需要此权限

* **DELETE**

删除数据

* **DROP**

删除 `series` 或 `measurement`

* **SENSITIZE**

拥有查询敏感数据的权限，可避免被数据脱敏

    

除了 `SHOW DATABASES`外，其他权限之间是相互独立的、正交的；当拥有权限  `CREATE CQ`, `INSERT`, `SELECT`, `DELETE`, `DROP` 之一时，将自动拥有 `SHOW DATABASES`权限。



### 角色管理

角色是一组权限的集合。

```sql
/* 创建角色 */
CREATE ROLE <role_name>
create role writer**/* 查看角色 */
SHOW ROLES
SHOW PRIVILEGE FOR <role_name>**/* 停/启用角色 */
ALTER ROLE <role_name> DISABLE/ENABLE**/* 删除角色 */
DROP ROLE <role_name>**/* 删除角色后，被赋予此角色的用户的权限也将一同被删除！ */
```

### 授权方法

海东青中，权限可以授予给`USER`和 `ROLE`, 同时 `ROLE`表示是一组权限的集合，也可以授权给`USER`：

`GRANT/REVOKE <some_privige> [ON <some_source>] TO ROlE/USER`

```shell
/*示例： */
grant select on city.rp0.air to worker_user
grant role0 to worker_user
```

`CREATE CQ`, `INSERT`, `SELECT`, `DELETE`, `DROP` 权限是和资源相关的，可以通过 `ON`指定权限所属资源， 如果忽略则表示对所有资源生效。其中除了权限

1. `CREATE CQ` 只能限定至 database 层级
1. `INSERT` 最多只能限定至 retention policy

之外，最低资源限制颗粒度为 `MEASUREMENT`。

### 内置角色

1. `admin`:  管理员，拥有所有权限。
1. `observer`: 观察者，拥有一切资源的可读权限。
1. `security_guard`: 安全管理员，不具备用户数据访问权限。
1. `dbadmin`:  数据库管理员，只允许访问用户数据。
1. `auditor`: 审计员
