# ALTER FUNCTION

ALTER FUNCTION — change the definition of a function

### Synopsis

```text
ALTER FUNCTION name [ ( [ [ argmode ] [ argname ] argtype [, ...] ] ) ]
    action [ ... ] [ RESTRICT ]
ALTER FUNCTION name [ ( [ [ argmode ] [ argname ] argtype [, ...] ] ) ]
    RENAME TO new_name
ALTER FUNCTION name [ ( [ [ argmode ] [ argname ] argtype [, ...] ] ) ]
    OWNER TO { new_owner | CURRENT_USER | SESSION_USER }
ALTER FUNCTION name [ ( [ [ argmode ] [ argname ] argtype [, ...] ] ) ]
    SET SCHEMA new_schema
ALTER FUNCTION name [ ( [ [ argmode ] [ argname ] argtype [, ...] ] ) ]
    DEPENDS ON EXTENSION extension_name

where action is one of:

    CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT
    IMMUTABLE | STABLE | VOLATILE | [ NOT ] LEAKPROOF
    [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER
    PARALLEL { UNSAFE | RESTRICTED | SAFE }
    COST execution_cost
    ROWS result_rows
    SET configuration_parameter { TO | = } { value | DEFAULT }
    SET configuration_parameter FROM CURRENT
    RESET configuration_parameter
    RESET ALL
```

### Description

`ALTER FUNCTION` changes the definition of a function.

You must own the function to use `ALTER FUNCTION`. To change a function's schema, you must also have `CREATE` privilege on the new schema. To alter the owner, you must also be a direct or indirect member of the new owning role, and that role must have `CREATE` privilege on the function's schema. \(These restrictions enforce that altering the owner doesn't do anything you couldn't do by dropping and recreating the function. However, a superuser can alter ownership of any function anyway.\)

### Parameters

_`name`_

The name \(optionally schema-qualified\) of an existing function. If no argument list is specified, the name must be unique in its schema._`argmode`_

The mode of an argument: `IN`, `OUT`, `INOUT`, or `VARIADIC`. If omitted, the default is `IN`. Note that `ALTER FUNCTION` does not actually pay any attention to `OUT` arguments, since only the input arguments are needed to determine the function's identity. So it is sufficient to list the `IN`, `INOUT`, and `VARIADIC` arguments._`argname`_

The name of an argument. Note that `ALTER FUNCTION` does not actually pay any attention to argument names, since only the argument data types are needed to determine the function's identity._`argtype`_

The data type\(s\) of the function's arguments \(optionally schema-qualified\), if any._`new_name`_

The new name of the function._`new_owner`_

The new owner of the function. Note that if the function is marked `SECURITY DEFINER`, it will subsequently execute as the new owner._`new_schema`_

The new schema for the function._`extension_name`_

The name of the extension that the function is to depend on.`CALLED ON NULL INPUT`  
`RETURNS NULL ON NULL INPUT`  
`STRICT`

`CALLED ON NULL INPUT` changes the function so that it will be invoked when some or all of its arguments are null. `RETURNS NULL ON NULL INPUT` or `STRICT` changes the function so that it is not invoked if any of its arguments are null; instead, a null result is assumed automatically. See [CREATE FUNCTION](https://www.postgresql.org/docs/10/static/sql-createfunction.html) for more information.`IMMUTABLE`  
`STABLE`  
`VOLATILE`

Change the volatility of the function to the specified setting. See [CREATE FUNCTION](https://www.postgresql.org/docs/10/static/sql-createfunction.html) for details.`[ EXTERNAL ] SECURITY INVOKER`  
`[ EXTERNAL ] SECURITY DEFINER`

Change whether the function is a security definer or not. The key word `EXTERNAL` is ignored for SQL conformance. See [CREATE FUNCTION](https://www.postgresql.org/docs/10/static/sql-createfunction.html) for more information about this capability.`PARALLEL`

Change whether the function is deemed safe for parallelism. See [CREATE FUNCTION](https://www.postgresql.org/docs/10/static/sql-createfunction.html) for details.`LEAKPROOF`

Change whether the function is considered leakproof or not. See [CREATE FUNCTION](https://www.postgresql.org/docs/10/static/sql-createfunction.html) for more information about this capability.`COST` _`execution_cost`_

Change the estimated execution cost of the function. See [CREATE FUNCTION](https://www.postgresql.org/docs/10/static/sql-createfunction.html) for more information.`ROWS` _`result_rows`_

Change the estimated number of rows returned by a set-returning function. See [CREATE FUNCTION](https://www.postgresql.org/docs/10/static/sql-createfunction.html) for more information._`configuration_parameter`_  
_`value`_

Add or change the assignment to be made to a configuration parameter when the function is called. If _`value`_ is `DEFAULT` or, equivalently, `RESET` is used, the function-local setting is removed, so that the function executes with the value present in its environment. Use `RESET ALL` to clear all function-local settings. `SET FROM CURRENT` saves the value of the parameter that is current when `ALTER FUNCTION` is executed as the value to be applied when the function is entered.

See [SET](https://www.postgresql.org/docs/10/static/sql-set.html) and [Chapter 19](https://www.postgresql.org/docs/10/static/runtime-config.html) for more information about allowed parameter names and values.`RESTRICT`

Ignored for conformance with the SQL standard.

### Examples

To rename the function `sqrt` for type `integer` to `square_root`:

```text
ALTER FUNCTION sqrt(integer) RENAME TO square_root;
```

To change the owner of the function `sqrt` for type `integer` to `joe`:

```text
ALTER FUNCTION sqrt(integer) OWNER TO joe;
```

To change the schema of the function `sqrt` for type `integer` to `maths`:

```text
ALTER FUNCTION sqrt(integer) SET SCHEMA maths;
```

To mark the function `sqrt` for type `integer` as being dependent on the extension `mathlib`:

```text
ALTER FUNCTION sqrt(integer) DEPENDS ON EXTENSION mathlib;
```

To adjust the search path that is automatically set for a function:

```text
ALTER FUNCTION check_password(text) SET search_path = admin, pg_temp;
```

To disable automatic setting of `search_path` for a function:

```text
ALTER FUNCTION check_password(text) RESET search_path;
```

The function will now execute with whatever search path is used by its caller.

### Compatibility

This statement is partially compatible with the `ALTER FUNCTION` statement in the SQL standard. The standard allows more properties of a function to be modified, but does not provide the ability to rename a function, make a function a security definer, attach configuration parameter values to a function, or change the owner, schema, or volatility of a function. The standard also requires the `RESTRICT` key word, which is optional in PostgreSQL.

### See Also

[CREATE FUNCTION](create-function.md), [DROP FUNCTION](drop-function.md)

