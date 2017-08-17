# 8.20. 概念型別[^1]

ThePostgreSQLtype system contains a number of special-purpose entries that are collectively called_pseudo-types_. A pseudo-type cannot be used as a column data type, but it can be used to declare a function's argument or result type. Each of the available pseudo-types is useful in situations where a function's behavior does not correspond to simply taking or returning a value of a specificSQLdata type.[Table 8.25](https://www.postgresql.org/docs/10/static/datatype-pseudo.html#datatype-pseudotypes-table)lists the existing pseudo-types.

**Table 8.25. Pseudo-Types**

| Name | Description |
| :--- | :--- |
| `any` | Indicates that a function accepts any input data type. |
| `anyelement` | Indicates that a function accepts any data type \(see[Section 37.2.5](https://www.postgresql.org/docs/10/static/extend-type-system.html#extend-types-polymorphic)\). |
| `anyarray` | Indicates that a function accepts any array data type \(see[Section 37.2.5](https://www.postgresql.org/docs/10/static/extend-type-system.html#extend-types-polymorphic)\). |
| `anynonarray` | Indicates that a function accepts any non-array data type \(see[Section 37.2.5](https://www.postgresql.org/docs/10/static/extend-type-system.html#extend-types-polymorphic)\). |
| `anyenum` | Indicates that a function accepts any enum data type \(see[Section 37.2.5](https://www.postgresql.org/docs/10/static/extend-type-system.html#extend-types-polymorphic)and[Section 8.7](https://www.postgresql.org/docs/10/static/datatype-enum.html)\). |
| `anyrange` | Indicates that a function accepts any range data type \(see[Section 37.2.5](https://www.postgresql.org/docs/10/static/extend-type-system.html#extend-types-polymorphic)and[Section 8.17](https://www.postgresql.org/docs/10/static/rangetypes.html)\). |
| `cstring` | Indicates that a function accepts or returns a null-terminated C string. |
| `internal` | Indicates that a function accepts or returns a server-internal data type. |
| `language_handler` | A procedural language call handler is declared to return`language_handler`. |
| `fdw_handler` | A foreign-data wrapper handler is declared to return`fdw_handler`. |
| `index_am_handler` | An index access method handler is declared to return`index_am_handler`. |
| `tsm_handler` | A tablesample method handler is declared to return`tsm_handler`. |
| `record` | Identifies a function taking or returning an unspecified row type. |
| `trigger` | A trigger function is declared to return`trigger.` |
| `event_trigger` | An event trigger function is declared to return`event_trigger.` |
| `pg_ddl_command` | Identifies a representation of DDL commands that is available to event triggers. |
| `void` | Indicates that a function returns no value. |
| `unknown` | Identifies a not-yet-resolved type, e.g. of an undecorated string literal. |
| `opaque` | An obsolete type name that formerly served many of the above purposes. |

  


Functions coded in C \(whether built-in or dynamically loaded\) can be declared to accept or return any of these pseudo data types. It is up to the function author to ensure that the function will behave safely when a pseudo-type is used as an argument type.

Functions coded in procedural languages can use pseudo-types only as allowed by their implementation languages. At present most procedural languages forbid use of a pseudo-type as an argument type, and allow only`void`and`record`as a result type \(plus`trigger`or`event_trigger`when the function is used as a trigger or event trigger\). Some also support polymorphic functions using the types`anyelement`,`anyarray`,`anynonarray`,`anyenum`, and`anyrange`.

The`internal`pseudo-type is used to declare functions that are meant only to be called internally by the database system, and not by direct invocation in anSQLquery. If a function has at least one`internal`-type argument then it cannot be called fromSQL. To preserve the type safety of this restriction it is important to follow this coding rule: do not create any function that is declared to return`internal`unless it has at least one`internal`argument.

---



[^1]:  [PostgreSQL: Documentation: 10: 8.20. Pseudo-Types](https://www.postgresql.org/docs/10/static/datatype-pseudo.html)

