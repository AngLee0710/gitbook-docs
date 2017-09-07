# 5.9. 繼承[^1]

PostgreSQL 實作了資料表的繼承方式，對於資料庫設計人員來說，將會是很好用的功能。（SQL:1999 之後定義了型別繼承的功能，但和這裡所介紹的方向有許多不同。）

我們直接以一個例子作為開始：假設我們嘗試建立「城市（city）」的資料模型。每一個州（state）都會有許多城市（city），但只會有一個首都（capital）。我們想要很快地可以找到某個州的首都。這件事我們需要建立兩個資料表，一個存放首都，而另一個記載非首都的城市。只是，當我們想要取得的是所有城市，不論是否為首都，似乎會有些麻煩？這時候繼承功能就可以幫助我們解決這個問題。我們可以定義一個資料表 capitals，它是由資料表 cities 繼承而來：

```
CREATE TABLE cities (
    name            text,
    population      float,
    altitude        int     -- in feet
);

CREATE TABLE capitals (
    state           char(2)
) INHERITS (cities);
```

在這個例子中，資料表 capitals 會繼承父資料表 cities 的所有欄位。只是 capitals 會多一個欄位 state，表示它是哪個州的首都。

在 PostgreSQL 裡，一個資料表可以繼承多個資料表，而一個查詢可以引用該資料表裡的所有資料列或在其所屬的資料表的資料列，後者的行為是預設的。舉個例子，下面的查詢可以列出所有海沷在 500 英呎以上的城市名稱，州的首都也包含在內：

```
SELECT name, altitude
    FROM cities
    WHERE altitude > 500;
```

使用 [2.1 節](/the-sql-language/21-introduction.md)中的範例資料，將會回傳：

```
   name    | altitude
-----------+----------
 Las Vegas |     2174
 Mariposa  |     1953
 Madison   |      845
```

換句話說，下面的查詢就會查出非首都且海沷超過 500 英呎以上的城市：

```
SELECT name, altitude
    FROM ONLY cities
    WHERE altitude > 500;

   name    | altitude
-----------+----------
 Las Vegas |     2174
 Mariposa  |     1953
```

這裡「ONLY」關鍵字指的是查詢只需要包含資料表 cities 就好，而不是任何繼承 cities 的資料表都包含在內。我們先前介紹過的指令：SELECT

Here the`ONLY`keyword indicates that the query should apply only to`cities`, and not any tables below`cities`in the inheritance hierarchy. Many of the commands that we have already discussed —`SELECT`,`UPDATE`and`DELETE`— support the`ONLY`keyword.

You can also write the table name with a trailing`*`to explicitly specify that descendant tables are included:

```
SELECT name, altitude
    FROM cities*
    WHERE altitude 
>
 500;
```

Writing`*`is not necessary, since this behavior is always the default. However, this syntax is still supported for compatibility with older releases where the default could be changed.

In some cases you might wish to know which table a particular row originated from. There is a system column called`tableoid`in each table which can tell you the originating table:

```
SELECT c.tableoid, c.name, c.altitude
FROM cities c
WHERE c.altitude 
>
 500;
```

which returns:

```
 tableoid |   name    | altitude
----------+-----------+----------
   139793 | Las Vegas |     2174
   139793 | Mariposa  |     1953
   139798 | Madison   |      845
```

\(If you try to reproduce this example, you will probably get different numeric OIDs.\) By doing a join with`pg_class`you can see the actual table names:

```
SELECT p.relname, c.name, c.altitude
FROM cities c, pg_class p
WHERE c.altitude 
>
 500 AND c.tableoid = p.oid;
```

which returns:

```
 relname  |   name    | altitude
----------+-----------+----------
 cities   | Las Vegas |     2174
 cities   | Mariposa  |     1953
 capitals | Madison   |      845
```

Another way to get the same effect is to use the`regclass`alias type, which will print the table OID symbolically:

```
SELECT c.tableoid::regclass, c.name, c.altitude
FROM cities c
WHERE c.altitude 
>
 500;
```

Inheritance does not automatically propagate data from`INSERT`or`COPY`commands to other tables in the inheritance hierarchy. In our example, the following`INSERT`statement will fail:

```
INSERT INTO cities (name, population, altitude, state)
VALUES ('Albany', NULL, NULL, 'NY');
```

We might hope that the data would somehow be routed to the`capitals`table, but this does not happen:`INSERT`always inserts into exactly the table specified. In some cases it is possible to redirect the insertion using a rule \(see[Chapter 40](https://www.postgresql.org/docs/10/static/rules.html)\). However that does not help for the above case because the`cities`table does not contain the column`state`, and so the command will be rejected before the rule can be applied.

All check constraints and not-null constraints on a parent table are automatically inherited by its children, unless explicitly specified otherwise with`NO INHERIT`clauses. Other types of constraints \(unique, primary key, and foreign key constraints\) are not inherited.

A table can inherit from more than one parent table, in which case it has the union of the columns defined by the parent tables. Any columns declared in the child table's definition are added to these. If the same column name appears in multiple parent tables, or in both a parent table and the child's definition, then these columns are“merged”so that there is only one such column in the child table. To be merged, columns must have the same data types, else an error is raised. Inheritable check constraints and not-null constraints are merged in a similar fashion. Thus, for example, a merged column will be marked not-null if any one of the column definitions it came from is marked not-null. Check constraints are merged if they have the same name, and the merge will fail if their conditions are different.

Table inheritance is typically established when the child table is created, using the`INHERITS`clause of the[CREATE TABLE](https://www.postgresql.org/docs/10/static/sql-createtable.html)statement. Alternatively, a table which is already defined in a compatible way can have a new parent relationship added, using the`INHERIT`variant of[ALTER TABLE](https://www.postgresql.org/docs/10/static/sql-altertable.html). To do this the new child table must already include columns with the same names and types as the columns of the parent. It must also include check constraints with the same names and check expressions as those of the parent. Similarly an inheritance link can be removed from a child using the`NO INHERIT`variant of`ALTER TABLE`. Dynamically adding and removing inheritance links like this can be useful when the inheritance relationship is being used for table partitioning \(see[Section 5.10](https://www.postgresql.org/docs/10/static/ddl-partitioning.html)\).

One convenient way to create a compatible table that will later be made a new child is to use the`LIKE`clause in`CREATE TABLE`. This creates a new table with the same columns as the source table. If there are any`CHECK`constraints defined on the source table, the`INCLUDING CONSTRAINTS`option to`LIKE`should be specified, as the new child must have constraints matching the parent to be considered compatible.

A parent table cannot be dropped while any of its children remain. Neither can columns or check constraints of child tables be dropped or altered if they are inherited from any parent tables. If you wish to remove a table and all of its descendants, one easy way is to drop the parent table with the`CASCADE`option \(see[Section 5.13](https://www.postgresql.org/docs/10/static/ddl-depend.html)\).

[ALTER TABLE](https://www.postgresql.org/docs/10/static/sql-altertable.html)will propagate any changes in column data definitions and check constraints down the inheritance hierarchy. Again, dropping columns that are depended on by other tables is only possible when using the`CASCADE`option.`ALTER TABLE`follows the same rules for duplicate column merging and rejection that apply during`CREATE TABLE`.

Inherited queries perform access permission checks on the parent table only. Thus, for example, granting`UPDATE`permission on the`cities`table implies permission to update rows in the`capitals`table as well, when they are accessed through`cities`. This preserves the appearance that the data is \(also\) in the parent table. But the`capitals`table could not be updated directly without an additional grant. In a similar way, the parent table's row security policies \(see[Section 5.7](https://www.postgresql.org/docs/10/static/ddl-rowsecurity.html)\) are applied to rows coming from child tables during an inherited query. A child table's policies, if any, are applied only when it is the table explicitly named in the query; and in that case, any policies attached to its parent\(s\) are ignored.

Foreign tables \(see[Section 5.11](https://www.postgresql.org/docs/10/static/ddl-foreign-data.html)\) can also be part of inheritance hierarchies, either as parent or child tables, just as regular tables can be. If a foreign table is part of an inheritance hierarchy then any operations not supported by the foreign table are not supported on the whole hierarchy either.

### 5.9.1. Caveats

Note that not all SQL commands are able to work on inheritance hierarchies. Commands that are used for data querying, data modification, or schema modification \(e.g.,`SELECT`,`UPDATE`,`DELETE`, most variants of`ALTER TABLE`, but not`INSERT`or`ALTER TABLE ... RENAME`\) typically default to including child tables and support the`ONLY`notation to exclude them. Commands that do database maintenance and tuning \(e.g.,`REINDEX`,`VACUUM`\) typically only work on individual, physical tables and do not support recursing over inheritance hierarchies. The respective behavior of each individual command is documented in its reference page \([SQL Commands](https://www.postgresql.org/docs/10/static/sql-commands.html)\).

A serious limitation of the inheritance feature is that indexes \(including unique constraints\) and foreign key constraints only apply to single tables, not to their inheritance children. This is true on both the referencing and referenced sides of a foreign key constraint. Thus, in the terms of the above example:

* If we declared`cities`.`name`to be`UNIQUE`or a`PRIMARY KEY`, this would not stop the`capitals`table from having rows with names duplicating rows in`cities`. And those duplicate rows would by default show up in queries from`cities`. In fact, by default`capitals`would have no unique constraint at all, and so could contain multiple rows with the same name. You could add a unique constraint to`capitals`, but this would not prevent duplication compared to`cities`.

* Similarly, if we were to specify that`cities`.`nameREFERENCES`some other table, this constraint would not automatically propagate to`capitals`. In this case you could work around it by manually adding the same`REFERENCES`constraint to`capitals`.

* Specifying that another table's column`REFERENCES cities(name)`would allow the other table to contain city names, but not capital names. There is no good workaround for this case.

These deficiencies will probably be fixed in some future release, but in the meantime considerable care is needed in deciding whether inheritance is useful for your application.

---

[^1]: [PostgreSQL: Documentation: 10: 5.9. Inheritance](https://www.postgresql.org/docs/10/static/ddl-inherit.html)

