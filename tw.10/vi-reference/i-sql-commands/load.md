# LOAD[^1]

LOAD — load a shared library file

## Synopsis

```
LOAD '
filename
'
```

## Description

This command loads a shared library file into thePostgreSQLserver's address space. If the file has been loaded already, the command does nothing. Shared library files that contain C functions are automatically loaded whenever one of their functions is called. Therefore, an explicit`LOAD`is usually only needed to load a library that modifies the server's behavior through“hooks”rather than providing a set of functions.

The library file name is typically given as just a bare file name, which is sought in the server's library search path \(set by[dynamic\_library\_path](https://www.postgresql.org/docs/10/static/runtime-config-client.html#GUC-DYNAMIC-LIBRARY-PATH)\). Alternatively it can be given as a full path name. In either case the platform's standard shared library file name extension may be omitted. See[Section 37.9.1](https://www.postgresql.org/docs/10/static/xfunc-c.html#XFUNC-C-DYNLOAD)for more information on this topic.



Non-superusers can only apply`LOAD`to library files located in`$libdir/plugins/`— the specified_`filename`_must begin with exactly that string. \(It is the database administrator's responsibility to ensure that only“safe”libraries are installed there.\)

## Compatibility

`LOAD`is aPostgreSQLextension.

## See Also

[CREATE FUNCTION](https://www.postgresql.org/docs/10/static/sql-createfunction.html)

---



[^1]:  [PostgreSQL: Documentation: 10: LOAD](https://www.postgresql.org/docs/10/static/sql-load.html)

