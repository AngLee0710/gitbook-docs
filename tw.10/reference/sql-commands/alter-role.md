---
description: 版本：10
---

# ALTER ROLE

ALTER ROLE — 變更資料庫角色

### 語法

```text
ALTER ROLE role_specification [ WITH ] option [ ... ]

where option can be:

      SUPERUSER | NOSUPERUSER
    | CREATEDB | NOCREATEDB
    | CREATEROLE | NOCREATEROLE
    | INHERIT | NOINHERIT
    | LOGIN | NOLOGIN
    | REPLICATION | NOREPLICATION
    | BYPASSRLS | NOBYPASSRLS
    | CONNECTION LIMIT connlimit
    | [ ENCRYPTED ] PASSWORD 'password'
    | VALID UNTIL 'timestamp'

ALTER ROLE name RENAME TO new_name

ALTER ROLE { role_specification | ALL } [ IN DATABASE database_name ] SET configuration_parameter { TO | = } { value | DEFAULT }
ALTER ROLE { role_specification | ALL } [ IN DATABASE database_name ] SET configuration_parameter FROM CURRENT
ALTER ROLE { role_specification | ALL } [ IN DATABASE database_name ] RESET configuration_parameter
ALTER ROLE { role_specification | ALL } [ IN DATABASE database_name ] RESET ALL

where role_specification can be:

    role_name
  | CURRENT_USER
  | SESSION_USER
```

### Description

`ALTER ROLE` changes the attributes of a PostgreSQL role.

The first variant of this command listed in the synopsis can change many of the role attributes that can be specified in [CREATE ROLE](https://www.postgresql.org/docs/10/static/sql-createrole.html). \(All the possible attributes are covered, except that there are no options for adding or removing memberships; use [GRANT](https://www.postgresql.org/docs/10/static/sql-grant.html) and [REVOKE](https://www.postgresql.org/docs/10/static/sql-revoke.html) for that.\) Attributes not mentioned in the command retain their previous settings. Database superusers can change any of these settings for any role. Roles having `CREATEROLE` privilege can change any of these settings, but only for non-superuser and non-replication roles. Ordinary roles can only change their own password.

The second variant changes the name of the role. Database superusers can rename any role. Roles having `CREATEROLE` privilege can rename non-superuser roles. The current session user cannot be renamed. \(Connect as a different user if you need to do that.\) Because `MD5`-encrypted passwords use the role name as cryptographic salt, renaming a role clears its password if the password is `MD5`-encrypted.

The remaining variants change a role's session default for a configuration variable, either for all databases or, when the `IN DATABASE` clause is specified, only for sessions in the named database. If `ALL` is specified instead of a role name, this changes the setting for all roles. Using `ALL` with `IN DATABASE` is effectively the same as using the command `ALTER DATABASE ... SET ...`.

Whenever the role subsequently starts a new session, the specified value becomes the session default, overriding whatever setting is present in `postgresql.conf` or has been received from the `postgres` command line. This only happens at login time; executing [SET ROLE](https://www.postgresql.org/docs/10/static/sql-set-role.html) or [SET SESSION AUTHORIZATION](https://www.postgresql.org/docs/10/static/sql-set-session-authorization.html) does not cause new configuration values to be set. Settings set for all databases are overridden by database-specific settings attached to a role. Settings for specific databases or specific roles override settings for all roles.

Superusers can change anyone's session defaults. Roles having `CREATEROLE` privilege can change defaults for non-superuser roles. Ordinary roles can only set defaults for themselves. Certain configuration variables cannot be set this way, or can only be set if a superuser issues the command. Only superusers can change a setting for all roles in all databases.

### Parameters

_`name`_

The name of the role whose attributes are to be altered.

`CURRENT_USER`

Alter the current user instead of an explicitly identified role.

`SESSION_USER`

Alter the current session user instead of an explicitly identified role.

`SUPERUSER`  
`NOSUPERUSER`  
`CREATEDB`  
`NOCREATEDB`  
`CREATEROLE`  
`NOCREATEROLE`  
`INHERIT`  
`NOINHERIT`  
`LOGIN`  
`NOLOGIN`  
`REPLICATION`  
`NOREPLICATION`  
`BYPASSRLS`  
`NOBYPASSRLS`  
`CONNECTION LIMIT` _`connlimit`_  
\[ `ENCRYPTED` \] `PASSWORD` _`password`_  
`VALID UNTIL` '_`timestamp`_'

These clauses alter attributes originally set by [CREATE ROLE](https://www.postgresql.org/docs/10/static/sql-createrole.html). For more information, see the `CREATE ROLE` reference page.

_`new_name`_

The new name of the role.

_`database_name`_

The name of the database the configuration variable should be set in.

_`configuration_parameter`_  
_`value`_

Set this role's session default for the specified configuration parameter to the given value. If _`value`_ is `DEFAULT` or, equivalently, `RESET` is used, the role-specific variable setting is removed, so the role will inherit the system-wide default setting in new sessions. Use `RESET ALL` to clear all role-specific settings. `SET FROM CURRENT` saves the session's current value of the parameter as the role-specific value. If `IN DATABASE` is specified, the configuration parameter is set or removed for the given role and database only.

Role-specific variable settings take effect only at login; [SET ROLE](https://www.postgresql.org/docs/10/static/sql-set-role.html) and [SET SESSION AUTHORIZATION](https://www.postgresql.org/docs/10/static/sql-set-session-authorization.html) do not process role-specific variable settings.

See [SET](https://www.postgresql.org/docs/10/static/sql-set.html) and [Chapter 19](https://www.postgresql.org/docs/10/static/runtime-config.html) for more information about allowed parameter names and values.

### 注意

使用 [CREATE ROLE](create-role.md) 增加新角色，使用 [DROP ROLE](drop-role.md) 移除角色。

ALTER ROLE 無法變更角色的成員資格。請使用 [GRANT](grant.md) 和 [REVOKE](revoke.md) 來做到這一點。

使用此指令指定未加密的密碼時必須小心。密碼將以明文形式傳輸到伺服器，也可能記錄在用戶端的指令歷史記錄或伺服器日誌中。psql 包含一個指令 \password，可用於變更角色的密碼而不暴露明文密碼。

也可以將連線預設值綁定到特定資料庫而不是角色；請參閱 ALTER DATABASE。 如果存在衝突，則特定於資料庫角色的設定會覆蓋特定於角色的設定，而這些設定又會覆蓋特定於資料庫的設定。

### 範例

變更角色的密碼：

```text
ALTER ROLE davide WITH PASSWORD 'hu8jmn3';
```

移除角色的密碼：

```text
ALTER ROLE davide WITH PASSWORD NULL;
```

變更密碼到期日期，指定密碼將於 2015 年 5 月 4 日中午到期，使用比 UTC 提前一小時的時區：

```text
ALTER ROLE chris VALID UNTIL 'May 4 12:00:00 2015 +1';
```

使密碼永久有效：

```text
ALTER ROLE fred VALID UNTIL 'infinity';
```

賦予角色建立其他角色和新資料庫的能力：

```text
ALTER ROLE miriam CREATEROLE CREATEDB;
```

讓某個角色的 [maintenance\_work\_mem](../../server-administration/server-configuration/resource-consumption.md#19-4-1) 參數使用非預設值：

```text
ALTER ROLE worker_bee SET maintenance_work_mem = 100000;
```

為 [client\_min\_messages ](../../server-administration/server-configuration/error-reporting-and-logging.md#client_min_messages-enum)參數指定一個非預設的，特定於某資料庫的設定：

```text
ALTER ROLE fred IN DATABASE devel SET client_min_messages = DEBUG;
```

### 相容性

ALTER ROLE 語句是 PostgreSQL 的延伸功能。

### 參閱

[CREATE ROLE](create-role.md), [DROP ROLE](drop-role.md), [ALTER DATABASE](alter-database.md), [SET](set.md)

