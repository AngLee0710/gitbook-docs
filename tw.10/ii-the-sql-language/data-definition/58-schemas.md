# 5.8. Schemas[^1]

APostgreSQLdatabase cluster contains one or more named databases. Users and groups of users are shared across the entire cluster, but no other data is shared across databases. Any given client connection to the server can access only the data in a single database, the one specified in the connection request.

### Note

Users of a cluster do not necessarily have the privilege to access every database in the cluster. Sharing of user names means that there cannot be different users named, say,`joe`in two databases in the same cluster; but the system can be configured to allow`joe`access to only some of the databases.

A database contains one or more named_schemas_, which in turn contain tables. Schemas also contain other kinds of named objects, including data types, functions, and operators. The same object name can be used in different schemas without conflict; for example, both`schema1`and`myschema`can contain tables named`mytable`. Unlike databases, schemas are not rigidly separated: a user can access objects in any of the schemas in the database they are connected to, if they have privileges to do so.

There are several reasons why one might want to use schemas:

* To allow many users to use one database without interfering with each other.

* To organize database objects into logical groups to make them more manageable.

* Third-party applications can be put into separate schemas so they do not collide with the names of other objects.

Schemas are analogous to directories at the operating system level, except that schemas cannot be nested.

### 5.8.1. Creating a Schema



To create a schema, use the[CREATE SCHEMA](https://www.postgresql.org/docs/10/static/sql-createschema.html)command. Give the schema a name of your choice. For example:

```
CREATE SCHEMA myschema;

```





To create or access objects in a schema, write a_qualified name_consisting of the schema name and table name separated by a dot:

```
schema
.
table
```

This works anywhere a table name is expected, including the table modification commands and the data access commands discussed in the following chapters. \(For brevity we will speak of tables only, but the same ideas apply to other kinds of named objects, such as types and functions.\)

Actually, the even more general syntax

```
database
.
schema
.
table
```

can be used too, but at present this is just for_pro forma_compliance with the SQL standard. If you write a database name, it must be the same as the database you are connected to.

So to create a table in the new schema, use:

```
CREATE TABLE myschema.mytable (
 ...
);

```



To drop a schema if it's empty \(all objects in it have been dropped\), use:

```
DROP SCHEMA myschema;

```

To drop a schema including all contained objects, use:

```
DROP SCHEMA myschema CASCADE;

```

See[Section 5.13](https://www.postgresql.org/docs/10/static/ddl-depend.html)for a description of the general mechanism behind this.

Often you will want to create a schema owned by someone else \(since this is one of the ways to restrict the activities of your users to well-defined namespaces\). The syntax for that is:

```
CREATE SCHEMA 
schema_name
 AUTHORIZATION 
user_name
;

```

You can even omit the schema name, in which case the schema name will be the same as the user name. See[Section 5.8.6](https://www.postgresql.org/docs/10/static/ddl-schemas.html#ddl-schemas-patterns)for how this can be useful.

Schema names beginning with`pg_`are reserved for system purposes and cannot be created by users.

### 5.8.2. The Public Schema



In the previous sections we created tables without specifying any schema names. By default such tables \(and other objects\) are automatically put into a schema named“public”. Every new database contains such a schema. Thus, the following are equivalent:

```
CREATE TABLE products ( ... );

```

and:

```
CREATE TABLE public.products ( ... );

```

### 5.8.3. The Schema Search Path







Qualified names are tedious to write, and it's often best not to wire a particular schema name into applications anyway. Therefore tables are often referred to by_unqualified names_, which consist of just the table name. The system determines which table is meant by following a_search path_, which is a list of schemas to look in. The first matching table in the search path is taken to be the one wanted. If there is no match in the search path, an error is reported, even if matching table names exist in other schemas in the database.



The first schema named in the search path is called the current schema. Aside from being the first schema searched, it is also the schema in which new tables will be created if the`CREATE TABLE`command does not specify a schema name.



To show the current search path, use the following command:

```
SHOW search_path;

```

In the default setup this returns:

```
 search_path
--------------
 "$user", public

```

The first element specifies that a schema with the same name as the current user is to be searched. If no such schema exists, the entry is ignored. The second element refers to the public schema that we have seen already.

The first schema in the search path that exists is the default location for creating new objects. That is the reason that by default objects are created in the public schema. When objects are referenced in any other context without schema qualification \(table modification, data modification, or query commands\) the search path is traversed until a matching object is found. Therefore, in the default configuration, any unqualified access again can only refer to the public schema.

To put our new schema in the path, we use:

```
SET search_path TO myschema,public;

```

\(We omit the`$user`here because we have no immediate need for it.\) And then we can access the table without schema qualification:

```
DROP TABLE mytable;

```

Also, since`myschema`is the first element in the path, new objects would by default be created in it.

We could also have written:

```
SET search_path TO myschema;

```

Then we no longer have access to the public schema without explicit qualification. There is nothing special about the public schema except that it exists by default. It can be dropped, too.

See also[Section 9.25](https://www.postgresql.org/docs/10/static/functions-info.html)for other ways to manipulate the schema search path.

The search path works in the same way for data type names, function names, and operator names as it does for table names. Data type and function names can be qualified in exactly the same way as table names. If you need to write a qualified operator name in an expression, there is a special provision: you must write

```
OPERATOR(
schema
.
operator
)
```

This is needed to avoid syntactic ambiguity. An example is:

```
SELECT 3 OPERATOR(pg_catalog.+) 4;

```

In practice one usually relies on the search path for operators, so as not to have to write anything so ugly as that.

### 5.8.4. Schemas and Privileges



By default, users cannot access any objects in schemas they do not own. To allow that, the owner of the schema must grant the`USAGE`privilege on the schema. To allow users to make use of the objects in the schema, additional privileges might need to be granted, as appropriate for the object.

A user can also be allowed to create objects in someone else's schema. To allow that, the`CREATE`privilege on the schema needs to be granted. Note that by default, everyone has`CREATE`and`USAGE`privileges on the schema`public`. This allows all users that are able to connect to a given database to create objects in its`public`schema. If you do not want to allow that, you can revoke that privilege:

```
REVOKE CREATE ON SCHEMA public FROM PUBLIC;

```

\(The first“public”is the schema, the second“public”means“every user”. In the first sense it is an identifier, in the second sense it is a key word, hence the different capitalization; recall the guidelines from[Section 4.1.1](https://www.postgresql.org/docs/10/static/sql-syntax-lexical.html#sql-syntax-identifiers).\)

### 5.8.5. The System Catalog Schema



In addition to`public`and user-created schemas, each database contains a`pg_catalog`schema, which contains the system tables and all the built-in data types, functions, and operators.`pg_catalog`is always effectively part of the search path. If it is not named explicitly in the path then it is implicitly searched_before_searching the path's schemas. This ensures that built-in names will always be findable. However, you can explicitly place`pg_catalog`at the end of your search path if you prefer to have user-defined names override built-in names.

Since system table names begin with`pg_`, it is best to avoid such names to ensure that you won't suffer a conflict if some future version defines a system table named the same as your table. \(With the default search path, an unqualified reference to your table name would then be resolved as the system table instead.\) System tables will continue to follow the convention of having names beginning with`pg_`, so that they will not conflict with unqualified user-table names so long as users avoid the`pg_`prefix.

### 5.8.6. Usage Patterns

Schemas can be used to organize your data in many ways. There are a few usage patterns that are recommended and are easily supported by the default configuration:

* If you do not create any schemas then all users access the public schema implicitly. This simulates the situation where schemas are not available at all. This setup is mainly recommended when there is only a single user or a few cooperating users in a database. This setup also allows smooth transition from the non-schema-aware world.

* You can create a schema for each user with the same name as that user. Recall that the default search path starts with`$user`, which resolves to the user name. Therefore, if each user has a separate schema, they access their own schemas by default.

  If you use this setup then you might also want to revoke access to the public schema \(or drop it altogether\), so users are truly constrained to their own schemas.

* To install shared applications \(tables to be used by everyone, additional functions provided by third parties, etc.\), put them into separate schemas. Remember to grant appropriate privileges to allow the other users to access them. Users can then refer to these additional objects by qualifying the names with a schema name, or they can put the additional schemas into their search path, as they choose.

### 5.8.7. Portability

In the SQL standard, the notion of objects in the same schema being owned by different users does not exist. Moreover, some implementations do not allow you to create schemas that have a different name than their owner. In fact, the concepts of schema and user are nearly equivalent in a database system that implements only the basic schema support specified in the standard. Therefore, many users consider qualified names to really consist of_`user_name`_._`table_name`_. This is howPostgreSQLwill effectively behave if you create a per-user schema for every user.

Also, there is no concept of a`public`schema in the SQL standard. For maximum conformance to the standard, you should not use \(perhaps even remove\) the`public`schema.

Of course, some SQL database systems might not implement schemas at all, or provide namespace support by allowing \(possibly limited\) cross-database access. If you need to work with those systems, then maximum portability would be achieved by not using schemas at all.

---



[^1]: [PostgreSQL: Documentation: 10: 5.8. Schemas](https://www.postgresql.org/docs/10/static/ddl-schemas.html)

