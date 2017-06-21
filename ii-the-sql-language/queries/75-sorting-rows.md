# 7.5. 資料排序[^1]

After a query has produced an output table \(after the select list has been processed\) it can optionally be sorted. If sorting is not chosen, the rows will be returned in an unspecified order. The actual order in that case will depend on the scan and join plan types and the order on disk, but it must not be relied on. A particular output ordering can only be guaranteed if the sort step is explicitly chosen.

The`ORDER BY`clause specifies the sort order:

```
SELECT 
select_list

    FROM 
table_expression

    ORDER BY 
sort_expression1
 [
ASC | DESC
] [
NULLS { FIRST | LAST }
]
             [
, 
sort_expression2
 [
ASC | DESC
] [
NULLS { FIRST | LAST }
] ...
]

```

The sort expression\(s\) can be any expression that would be valid in the query's select list. An example is:

```
SELECT a, b FROM table1 ORDER BY a + b, c;

```

When more than one expression is specified, the later values are used to sort rows that are equal according to the earlier values. Each expression can be followed by an optional`ASC`or`DESC`keyword to set the sort direction to ascending or descending.`ASC`order is the default. Ascending order puts smaller values first, where“smaller”is defined in terms of the`<`operator. Similarly, descending order is determined with the`>`operator.[\[5\]](https://www.postgresql.org/docs/10/static/queries-order.html#ftn.idm46249858218224)

The`NULLS FIRST`and`NULLS LAST`options can be used to determine whether nulls appear before or after non-null values in the sort ordering. By default, null values sort as if larger than any non-null value; that is,`NULLS FIRST`is the default for`DESC`order, and`NULLS LAST`otherwise.

Note that the ordering options are considered independently for each sort column. For example`ORDER BY x, y DESC`means`ORDER BY x ASC, y DESC`, which is not the same as`ORDER BY x DESC, y DESC`.

A_`sort_expression`_can also be the column label or number of an output column, as in:

```
SELECT a + b AS sum, c FROM table1 ORDER BY sum;
SELECT a, max(b) FROM table1 GROUP BY a ORDER BY 1;

```

both of which sort by the first output column. Note that an output column name has to stand alone, that is, it cannot be used in an expression — for example, this is_not_correct:

```
SELECT a + b AS sum, c FROM table1 ORDER BY sum + c;          -- wrong

```

This restriction is made to reduce ambiguity. There is still ambiguity if an`ORDER BY`item is a simple name that could match either an output column name or a column from the table expression. The output column is used in such cases. This would only cause confusion if you use`AS`to rename an output column to match some other table column's name.

`ORDER BY`can be applied to the result of a`UNION`,`INTERSECT`, or`EXCEPT`combination, but in this case it is only permitted to sort by output column names or numbers, not by expressions.

  


---

[\[5\]](https://www.postgresql.org/docs/10/static/queries-order.html#idm46249858218224)Actually,PostgreSQLuses the_default B-tree operator class_for the expression's data type to determine the sort ordering for`ASC`and`DESC`. Conventionally, data types will be set up so that the`<`and`>`operators correspond to this sort ordering, but a user-defined data type's designer could choose to do something different.

---



[^1]: [PostgreSQL: Documentation: 10: 7.5. Sorting Rows](https://www.postgresql.org/docs/10/static/queries-order.html)

