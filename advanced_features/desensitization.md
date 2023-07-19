# 数据脱敏

数据脱敏是指对于数据库中存储的一些敏感数据的查询结果进行一些改造，避免敏感数据的泄漏。本数据库提供静态脱敏功能，具有使用简单灵活等特点。

# 使用

通过 `ALTER`设置一个脱敏规则：

```sql
/* 
  对资源db.[rp].measurement的field列设置脱敏规则desensitize-rule
*/
ALTER SENSITIZE ON db.[rp].measurement FIELD field FUNCTION desensitize-rule
/* 示例：*/
alter sensitize on datax.autogen.cpu field phone function  concat(substr(phone,0,3),'***',substr(phone,6))
```

脱敏规则中的函数只能是 `concat`或 `substr`, 且脱敏规则中的字段必须和 `FIELD`关键字的值相对应。示例脱敏规则的含义是：凡是查询中涉及到了 `phone`列，那么从存储层获取到的 `phone` 的原始数据，首先进行 `concat(substr(phone,0,3),'***',substr(phone,6))`处理，之后才进入其他处理逻辑。

对同一个资源设置多次不同的脱敏规则，当然是最后一次有效。脱敏操作是基于字符串操作函数的，所以当资源的数据类型不是字符串时，脱敏之前会被强制转换成字面量：数字转化为十进制字符串，Boolean则转换成 `true/false`；数字在脱敏后就变成了字符串，因此不能在之后对其进行数字操作，这是本功能的最大缺陷！

删除一个资源的脱敏规则：

```sql
ALTER SENSITIZE ON db.rp.measurement FIELD field DISABLE
/*示例： */
alter sensitize on datax.autogen.cpu field phone disable
```

脱敏规则实际存储在内部数据库的一个表中，查看所有脱敏规则：

```sql
SELECT * FROM _fctsdb.autogen.desensitize_info
```

# 权限

权限系统中有与数据脱敏相对应的全局权限：`SensitizePrivilege`,  当用户拥有此权限时，数据将不会被脱敏、能够获取到原始数据。

```sql
GRANT SENSITIZE TO user_name
REVOKE SENSITIZE FROM user_name
```

# 缺陷

* 对非字符串的强制类型转换，如上文所述。
* 对数据同质化有要求，如同时存在 `+86 888888` 和 `666666` 数据时，设置脱敏规则将会存在困难。
* 不能对 `Tag` 进行脱敏，继承了函数只能处理 `Field`数据的缺陷。
