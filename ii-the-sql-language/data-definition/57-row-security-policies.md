# 5.7. 資料列安全原則[^1]

除了透過 [GRANT 指令](/vi-reference/i-sql-commands/grant.md)設定 SQL 標準的[權限系統](/ii-the-sql-language/data-definition/56-privileges.md)之外，資料表也可以有資料列層級的安全原則，控制每個使用者在資料查詢或變更時，所能接觸到的資料列。這個功能就稱作資料列安全原則（Row-Level Security）。預設上，資料表並不會有這些安全原則，所以只要使用者能存取該資料表，就表示他能存取所有資料列的內容。

當資料列安全原則在資料表裡被啓動後（使用 [ALTER TABLE ... ENABLE ROW LEVEL SECURITY](/vi-reference/i-sql-commands/alter-table.md)），所有資料表的操作，就必須符合資料列安全原則的設定。（當然，資料表的擁有者並不受限於資料列安全原則。）如果資料表中未設定任何原則，那麼預設就是拒絕存取，意思就是任何資料列都不能被看見或修改。但如果是整個資料表的操作行為，像是 TRUNCATE 或 REFERENCES，就不會受到影響。

資料列安全原則可以被設定在命令，使用者角色，或兩者兼具。安全原則也可以使用 ALL 的修飾字，或具體指出是 SELECT、INSERT、UPDATE、或 DELETE。多重角色可以共用一個安全原則，一般使用者或承繼的角色都會被同步影響到。

要設定一個安全原則來指出哪些資料列可見或可修改，是以一個回傳值為布林值的表示式來決定的。這個表示式會計算每一個資料列的結果，在使用者進行任何操作之前。（這個規則唯一的例外是 leakproof 函數，用來確保沒有洩漏資訊；查詢最佳化元件會選押在確任資料列安全原則前就先執行它。）在這個表示式沒有回傳 true 的資料列，都是不能被存取的。獨立的表示式可用於提供資料列專屬的控制，判斷其是否可供讀取或修改。安全原則表示式是查詢的一部份，和使用者執行查詢時一起執行，不過，安全原則表示式是可以存取到該使用者看不到的資料。

超級使用者因為擁有 BYPASSRLS 的屬性，所以永遠可以通過安全原則檢查而存取資料表。資料表的擁有者一般來說也是可以通過檢查，但可以使用 [ALTER TABLE ... FORCE ROW LEVEL SECURITY](/vi-reference/i-sql-commands/alter-table.md) 來強制適用安全原則。

開啓或關閉資料列安全原則的權限，只屬於資料表擁有者。

使用 CREATE POLICY 指令來建立安全原則；使用 ALTER POLICY 指令來修改；使用 DROP POLICY 指令來移除原則。要開啓或關閉安全原則的功能，請使用 [ALTER TABLE](/vi-reference/i-sql-commands/alter-table.md) 指令。

Each policy has a name and multiple policies can be defined for a table. As policies are table-specific, each policy for a table must have a unique name. Different tables may have policies with the same name.

When multiple policies apply to a given query, they are combined using either`OR`\(for permissive policies, which are the default\) or using`AND`\(for restrictive policies\). This is similar to the rule that a given role has the privileges of all roles that they are a member of. Permissive vs. restrictive policies are discussed further below.

As a simple example, here is how to create a policy on the`account`relation to allow only members of the`managers`role to access rows, and only rows of their accounts:

```
CREATE TABLE accounts (manager text, company text, contact_email text);

ALTER TABLE accounts ENABLE ROW LEVEL SECURITY;

CREATE POLICY account_managers ON accounts TO managers
    USING (manager = current_user);
```

If no role is specified, or the special user name`PUBLIC`is used, then the policy applies to all users on the system. To allow all users to access their own row in a`users`table, a simple policy can be used:

```
CREATE POLICY user_policy ON users
    USING (user_name = current_user);
```

To use a different policy for rows that are being added to the table compared to those rows that are visible, the`WITH CHECK`clause can be used. This policy would allow all users to view all rows in the`users`table, but only modify their own:

```
CREATE POLICY user_policy ON users
    USING (true)
    WITH CHECK (user_name = current_user);
```

Row security can also be disabled with the`ALTER TABLE`command. Disabling row security does not remove any policies that are defined on the table; they are simply ignored. Then all rows in the table are visible and modifiable, subject to the standard SQL privileges system.

Below is a larger example of how this feature can be used in production environments. The table`passwd`emulates a Unix password file:

```
-- Simple passwd-file based example
CREATE TABLE passwd (
  user_name             text UNIQUE NOT NULL,
  pwhash                text,
  uid                   int  PRIMARY KEY,
  gid                   int  NOT NULL,
  real_name             text NOT NULL,
  home_phone            text,
  extra_info            text,
  home_dir              text NOT NULL,
  shell                 text NOT NULL
);

CREATE ROLE admin;  -- Administrator
CREATE ROLE bob;    -- Normal user
CREATE ROLE alice;  -- Normal user

-- Populate the table
INSERT INTO passwd VALUES
  ('admin','xxx',0,0,'Admin','111-222-3333',null,'/root','/bin/dash');
INSERT INTO passwd VALUES
  ('bob','xxx',1,1,'Bob','123-456-7890',null,'/home/bob','/bin/zsh');
INSERT INTO passwd VALUES
  ('alice','xxx',2,1,'Alice','098-765-4321',null,'/home/alice','/bin/zsh');

-- Be sure to enable row level security on the table
ALTER TABLE passwd ENABLE ROW LEVEL SECURITY;

-- Create policies
-- Administrator can see all rows and add any rows
CREATE POLICY admin_all ON passwd TO admin USING (true) WITH CHECK (true);
-- Normal users can view all rows
CREATE POLICY all_view ON passwd FOR SELECT USING (true);
-- Normal users can update their own records, but
-- limit which shells a normal user is allowed to set
CREATE POLICY user_mod ON passwd FOR UPDATE
  USING (current_user = user_name)
  WITH CHECK (
    current_user = user_name AND
    shell IN ('/bin/bash','/bin/sh','/bin/dash','/bin/zsh','/bin/tcsh')
  );

-- Allow admin all normal rights
GRANT SELECT, INSERT, UPDATE, DELETE ON passwd TO admin;
-- Users only get select access on public columns
GRANT SELECT
  (user_name, uid, gid, real_name, home_phone, extra_info, home_dir, shell)
  ON passwd TO public;
-- Allow users to update certain columns
GRANT UPDATE
  (pwhash, real_name, home_phone, extra_info, shell)
  ON passwd TO public;
```

As with any security settings, it's important to test and ensure that the system is behaving as expected. Using the example above, this demonstrates that the permission system is working properly.

```
-- admin can view all rows and fields
postgres=
>
 set role admin;
SET
postgres=
>
 table passwd;
 user_name | pwhash | uid | gid | real_name |  home_phone  | extra_info | home_dir    |   shell
-----------+--------+-----+-----+-----------+--------------+------------+-------------+-----------
 admin     | xxx    |   0 |   0 | Admin     | 111-222-3333 |            | /root       | /bin/dash
 bob       | xxx    |   1 |   1 | Bob       | 123-456-7890 |            | /home/bob   | /bin/zsh
 alice     | xxx    |   2 |   1 | Alice     | 098-765-4321 |            | /home/alice | /bin/zsh
(3 rows)

-- Test what Alice is able to do
postgres=
>
 set role alice;
SET
postgres=
>
 table passwd;
ERROR:  permission denied for relation passwd
postgres=
>
 select user_name,real_name,home_phone,extra_info,home_dir,shell from passwd;
 user_name | real_name |  home_phone  | extra_info | home_dir    |   shell
-----------+-----------+--------------+------------+-------------+-----------
 admin     | Admin     | 111-222-3333 |            | /root       | /bin/dash
 bob       | Bob       | 123-456-7890 |            | /home/bob   | /bin/zsh
 alice     | Alice     | 098-765-4321 |            | /home/alice | /bin/zsh
(3 rows)

postgres=
>
 update passwd set user_name = 'joe';
ERROR:  permission denied for relation passwd
-- Alice is allowed to change her own real_name, but no others
postgres=
>
 update passwd set real_name = 'Alice Doe';
UPDATE 1
postgres=
>
 update passwd set real_name = 'John Doe' where user_name = 'admin';
UPDATE 0
postgres=
>
 update passwd set shell = '/bin/xx';
ERROR:  new row violates WITH CHECK OPTION for "passwd"
postgres=
>
 delete from passwd;
ERROR:  permission denied for relation passwd
postgres=
>
 insert into passwd (user_name) values ('xxx');
ERROR:  permission denied for relation passwd
-- Alice can change her own password; RLS silently prevents updating other rows
postgres=
>
 update passwd set pwhash = 'abc';
UPDATE 1
```

All of the policies constructed thus far have been permissive policies, meaning that when multiple policies are applied they are combined using the "OR" boolean operator. While permissive policies can be constructed to only allow access to rows in the intended cases, it can be simpler to combine permissive policies with restrictive policies \(which the records must pass and which are combined using the "AND" boolean operator\). Building on the example above, we add a restrictive policy to require the administrator to be connected over a local unix socket to access the records of the passwd table:

```
CREATE POLICY admin_local_only ON passwd AS RESTRICTIVE TO admin
    USING (pg_catalog.inet_client_addr() IS NULL);
```

We can then see that an administrator connecting over a network will not see any records, due to the restrictive policy:

```
=
>
 SELECT current_user;
 current_user 
--------------
 admin
(1 row)

=
>
 select inet_client_addr();
 inet_client_addr 
------------------
 127.0.0.1
(1 row)

=
>
 SELECT current_user;
 current_user 
--------------
 admin
(1 row)

=
>
 TABLE passwd;
 user_name | pwhash | uid | gid | real_name | home_phone | extra_info | home_dir | shell
-----------+--------+-----+-----+-----------+------------+------------+----------+-------
(0 rows)

=
>
 UPDATE passwd set pwhash = NULL;
UPDATE 0
```

Referential integrity checks, such as unique or primary key constraints and foreign key references, always bypass row security to ensure that data integrity is maintained. Care must be taken when developing schemas and row level policies to avoid“covert channel”leaks of information through such referential integrity checks.

In some contexts it is important to be sure that row security is not being applied. For example, when taking a backup, it could be disastrous if row security silently caused some rows to be omitted from the backup. In such a situation, you can set the[row\_security](https://www.postgresql.org/docs/10/static/runtime-config-client.html#guc-row-security)configuration parameter to`off`. This does not in itself bypass row security; what it does is throw an error if any query's results would get filtered by a policy. The reason for the error can then be investigated and fixed.

In the examples above, the policy expressions consider only the current values in the row to be accessed or updated. This is the simplest and best-performing case; when possible, it's best to design row security applications to work this way. If it is necessary to consult other rows or other tables to make a policy decision, that can be accomplished using sub-`SELECT`s, or functions that contain`SELECT`s, in the policy expressions. Be aware however that such accesses can create race conditions that could allow information leakage if care is not taken. As an example, consider the following table design:

```
-- definition of privilege groups
CREATE TABLE groups (group_id int PRIMARY KEY,
                     group_name text NOT NULL);

INSERT INTO groups VALUES
  (1, 'low'),
  (2, 'medium'),
  (5, 'high');

GRANT ALL ON groups TO alice;  -- alice is the administrator
GRANT SELECT ON groups TO public;

-- definition of users' privilege levels
CREATE TABLE users (user_name text PRIMARY KEY,
                    group_id int NOT NULL REFERENCES groups);

INSERT INTO users VALUES
  ('alice', 5),
  ('bob', 2),
  ('mallory', 2);

GRANT ALL ON users TO alice;
GRANT SELECT ON users TO public;

-- table holding the information to be protected
CREATE TABLE information (info text,
                          group_id int NOT NULL REFERENCES groups);

INSERT INTO information VALUES
  ('barely secret', 1),
  ('slightly secret', 2),
  ('very secret', 5);

ALTER TABLE information ENABLE ROW LEVEL SECURITY;

-- a row should be visible to/updatable by users whose security group_id is
-- greater than or equal to the row's group_id
CREATE POLICY fp_s ON information FOR SELECT
  USING (group_id 
<
= (SELECT group_id FROM users WHERE user_name = current_user));
CREATE POLICY fp_u ON information FOR UPDATE
  USING (group_id 
<
= (SELECT group_id FROM users WHERE user_name = current_user));

-- we rely only on RLS to protect the information table
GRANT ALL ON information TO public;
```

Now suppose that`alice`wishes to change the“slightly secret”information, but decides that`mallory`should not be trusted with the new content of that row, so she does:

```
BEGIN;
UPDATE users SET group_id = 1 WHERE user_name = 'mallory';
UPDATE information SET info = 'secret from mallory' WHERE group_id = 2;
COMMIT;
```

That looks safe; there is no window wherein`mallory`should be able to see the“secret from mallory”string. However, there is a race condition here. If`mallory`is concurrently doing, say,

```
SELECT * FROM information WHERE group_id = 2 FOR UPDATE;
```

and her transaction is in`READ COMMITTED`mode, it is possible for her to see“secret from mallory”. That happens if her transaction reaches the`information`row just after`alice`'s does. It blocks waiting for`alice`'s transaction to commit, then fetches the updated row contents thanks to the`FOR UPDATE`clause. However, it does\_not\_fetch an updated row for the implicit`SELECT`from`users`, because that sub-`SELECT`did not have`FOR UPDATE`; instead the`users`row is read with the snapshot taken at the start of the query. Therefore, the policy expression tests the old value of`mallory`'s privilege level and allows her to see the updated row.

There are several ways around this problem. One simple answer is to use`SELECT ... FOR SHARE`in sub-`SELECT`s in row security policies. However, that requires granting`UPDATE`privilege on the referenced table \(here`users`\) to the affected users, which might be undesirable. \(But another row security policy could be applied to prevent them from actually exercising that privilege; or the sub-`SELECT`could be embedded into a security definer function.\) Also, heavy concurrent use of row share locks on the referenced table could pose a performance problem, especially if updates of it are frequent. Another solution, practical if updates of the referenced table are infrequent, is to take an exclusive lock on the referenced table when updating it, so that no concurrent transactions could be examining old row values. Or one could just wait for all concurrent transactions to end after committing an update of the referenced table and before making changes that rely on the new security situation.

For additional details see[CREATE POLICY](https://www.postgresql.org/docs/10/static/sql-createpolicy.html)and[ALTER TABLE](https://www.postgresql.org/docs/10/static/sql-altertable.html).

---

[^1]: [PostgreSQL: Documentation: 10: 5.7. Row Security Policies](/PostgreSQL: Documentation: 10: 5.7. Row Security Policies)

