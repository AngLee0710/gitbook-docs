# 7.2. 資料表表示式[^1]

資料表表示式計算結果會是一個資料表。 資料表表示式包含一個 FROM 子句，可以選擇性地加上 WHERE、GROUP BY 和 HAVING 子句。普通資料表表示式只是簡單地引用磁碟上的資料表，即所謂的基本資料表，但是更複雜的表示式可以用來以各種方式修改或組合基本資料表。

資料表表示式中選擇性的 WHERE、GROUP BY 和 HAVING 子句指定在 FROM 子句中執行連續轉換程序後產出衍生的資料表。所有這些轉換都會生成一個虛擬資料表，該資料表提供給資料列表的資料列來計算查詢輸出的欄位。

### 7.2.1. The`FROM`Clause

FROM 子句由逗號分隔的引用資料表列表中給予的一個或多個其他資料表進而衍生出一個資料表。

```
FROM table_reference [,table_reference [, ...]]
```

資料表的引用可以是資料表名稱（可能是帶有完整路徑的），或衍生資料表（如子查詢，JOIN 查詢或這些運算的複合組合）。如果在 FROM 子句中列出了多個資料表引用，則這些資料表會自動交叉連結（cross-join）（也就是說，它們資料列的笛卡爾積 Cartesian product 已經形成了，詳見下文）。 FROM 列表的結果是一個中繼的虛擬資料表，然後可以透過 WHERE、GROUP BY 和HAVING 子句進行轉換，最終成為整個資料表表示式的結果。

當一個資料表引用將一個資料表作為資料表繼承結構的父資料表進行命名時，資料表引用不僅產生了該資料表的所有資料列，還產生了其所有後代資料表的資料列，除非該關鍵字只提供資料表名稱。但是，引用只會產生出現在資料表列表中的欄位 - 忽略在子資料表中增加的任何欄位。

除了在資料表名稱之前指定之外，還可以在表名後面寫「\*」以明確表示包含所有之後的資料表。 現在沒有什麼理由再使用這個語法了，因為搜尋子資料表在目前是預設的行為。但是，它提供了與舊版本支援的相容性。

#### 7.2.1.1. Joined Tables

交叉查詢（JOIN）的資料表是根據特定連接類型的規則從其他兩個（實際或衍生）資料表共同衍生而來的資料表。有 inner、outer、 及 cross-joins 可以使用。 交叉查詢資料表的一般語法是：

```
T1 join_type T2 [join_condition]
```

所有類型的交叉查詢可以連接在一起，也可以是巢狀結構： T1 或 T2 都可以是交叉查詢後的資料表。在 JOIN 子句使用可以小括號來控制交叉查詢的次序。在沒有括號的情況下，JOIN 子句就從左到右巢狀運算。

**Join Types**

* Cross join

```
T1 CROSS JOIN T2
```

對於來自 T1 和 T2（即笛卡爾乘積）資料列的每個可能的組合，交叉查詢的資料表將包含由 T1 中的所有欄位組成的資料列，隨後是 T2 中的所有的欄位。如果這些資料表分別具有 N 個資料列和 M 資料列，則交叉查詢後的資料表將具有 N \* M 個資料列。

`FROM T1 CROSS JOIN T2` 相當於 `FROM T1 INNER JOIN T2 ON TRUE`（詳見下文）。 它也相當於 `FROM T1, T2`。

> ### 注意
>
> 當超過兩個資料表出現時，最後描述的等價關係並不完全適用，因為 JOIN 的綁定比只有逗號的意義更緊密。例如「FROM T1 CROSS JOIN T2 INNER JOIN T3 ON condition」與「FROM T1，T2 INNER JOIN T3 ON condition」不一樣，因為條件 condition 會在第一種情況下引用 T1，而不會在第二種情況下引用 T1。

* Qualified joins

```
T1 { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2 ON boolean_expression
T1 { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2 USING (join column list )

T1 NATURAL { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2
```

INNER 和 OUTER 是所有語法上都是可以使用的選項。INNER 是預設的；LEFT、RIGHT 和 FULL 隱含著外部交叉查詢（outer join）的意義。

交叉查詢的條件在 ON 或 USING 子句中指定，或者由NATURAL 隱含指定。交叉查詢條件確定兩個資料表中的哪些欄位被「配對」，詳細說明如下。

支援指定的交叉查詢的類型是：

`INNER JOIN`

對於 T1 的每一資料列 R1，連結在資料表 T2 中每一資料列都有一個滿足與 R1 的交叉查詢條件。

`LEFT OUTER JOIN`

首先，會先執行內部交叉查詢（inner join）。 然後，對於 T1 中不滿足與 T2 中的任何資料列的交叉查詢條件的每一資料列，在 T2 加上空值欄位。因此，交叉查詢最後的資料表中 T1 的每個資料列都至少會出現一次。

`RIGHT OUTER JOIN`

首先，會先執行內部交叉查詢（inner join）。 然後，對於 T2 中不滿足與 T1 中的任何資料列的交叉查詢條件的每一資料列，在 T1 加上空值欄位。也就是左向交叉查詢的反向： 交叉查詢最後的資料表中 T2 的每個資料列都至少會出現一次。

`FULL OUTER JOIN`

首先，會先執行內部交叉查詢（inner join）。 然後，對於 T1 中不滿足與 T2 中的任何資料列的交叉查詢條件的每一資料列，在 T2 加上空值欄位。 同樣地，對於 T2 中不滿足與 T1 中的任何資料列的交叉查詢條件的每一資料列，在 T1 加上空值欄位。

ON 子句是最一般形式的交叉查詢條件：採用與 WHERE 子句中使用的相同形式的布林表示式。 如果 ON 表示式的計算結果為 true，則 T1 和 T2 中的一對資料列形成匹配。

USING 子句是一種簡寫，它允許你利用交叉查詢的兩端對接欄位使用相同名稱的特定情況。它使用逗號分隔的共享欄位名稱列表，並形成包含每個欄位的相等比較的交叉查詢條件。例如，使用USING \(a，b\) 連結 T1 和 T2 產生交叉查詢條件「ON T1.a = T2.a AND T1.b = T2.b」。

此外，JOIN USING 的輸出減少重覆的欄位：不需要輸出兩個連結好的欄位，因為它們一定具有相同的值。雖然 JOIN ON 由 T1 的所有欄位，再接著 T2 的所有欄位，但 JOIN USING 為每個列出的欄位配對（按列出的順序）產生一個輸出欄位，其後是來自 T1 的剩餘欄，隨後是來自 T2 的剩餘欄位。

最後，NATURAL 是 USING 的簡寫形式：它會形成一個 USING 列表，其中包含出現在兩個輸入資料表中的所有的欄位名。和 USING 一樣，這些欄位在輸出資料表中就只會出現一次。如果沒有相同的欄位名稱，NATURAL 可能就像 CROSS JOIN。

### Note

因為只有列出的欄位會被組合，所以 USING 對於交叉查詢關係中的欄位更改是相當安全的。NATURAL 的風險是較大，因為任何資料結構更新都導致新的配對欄位名稱出現，也造成交叉查詢組合新欄位。

把這些東西放在一起來看，假設我們有資料表 t1：

```
 num | name
-----+------
   1 | a
   2 | b
   3 | c
```

資料表`t2`:

```
 num | value
-----+-------
   1 | xxx
   3 | yyy
   5 | zzz
```

那麼我們可以得到以下結果為各種交叉查詢：

```
=> SELECT * FROM t1 CROSS JOIN t2;

 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   1 | a    |   3 | yyy
   1 | a    |   5 | zzz
   2 | b    |   1 | xxx
   2 | b    |   3 | yyy
   2 | b    |   5 | zzz
   3 | c    |   1 | xxx
   3 | c    |   3 | yyy
   3 | c    |   5 | zzz
(9 rows)


=> SELECT * FROM t1 INNER JOIN t2 ON t1.num = t2.num;

 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   3 | c    |   3 | yyy
(2 rows)


=> SELECT * FROM t1 INNER JOIN t2 USING (num);

 num | name | value
-----+------+-------
   1 | a    | xxx
   3 | c    | yyy
(2 rows)


=> SELECT * FROM t1 NATURAL INNER JOIN t2;

 num | name | value
-----+------+-------
   1 | a    | xxx
   3 | c    | yyy
(2 rows)


=> SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num;

 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   2 | b    |     |
   3 | c    |   3 | yyy
(3 rows)


=> SELECT * FROM t1 LEFT JOIN t2 USING (num);

 num | name | value
-----+------+-------
   1 | a    | xxx
   2 | b    |
   3 | c    | yyy
(3 rows)


=> SELECT * FROM t1 RIGHT JOIN t2 ON t1.num = t2.num;

 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   3 | c    |   3 | yyy
     |      |   5 | zzz
(3 rows)


=> SELECT * FROM t1 FULL JOIN t2 ON t1.num = t2.num;

 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   2 | b    |     |
   3 | c    |   3 | yyy
     |      |   5 | zzz
(4 rows)
```

ON 指定的交叉查詢條件也可以包含與交叉查詢無關的條件。 這可以證明對某些查詢有用，但需要仔細考慮。例如：

```
=> SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num AND t2.value = 'xxx';

 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
   2 | b    |     |
   3 | c    |     |
(3 rows)
```

請注意，將限制條件放在 WHERE 子句中會產生不同的結果：

```
=> SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num WHERE t2.value = 'xxx';

 num | name | num | value
-----+------+-----+-------
   1 | a    |   1 | xxx
(1 row)
```

這是因為在 ON 子句中放置的條件會在交叉查詢之前處理，而在交叉查詢之後才會處理在 WHERE 子句中的條件。這在 INNER JOIN 時沒有關係，但 OUTER JOIN 時就很重要。

#### 7.2.1.2. Table and Column Aliases

可以為資料表和複雜的資料表引用指定一個臨時名稱，以便在查詢的其餘部分中用於對該衍生資料表引用。這稱作為資料表別名。

要建立一個資料表別名，請使用：

```
FROM table_reference AS alias
```

或

```
FROM table_reference alias
```

AS 關鍵字是選用的。別名可以是任何識別名稱。

資料表別名的一個典型應用是將短識別名稱分配給長資料表名稱，以保持交叉查詢子句的可讀性。例如：

```
SELECT * FROM some_very_long_table_name s JOIN another_fairly_long_name a ON s.id = a.num;
```

就當下的查詢而言，別名將成為資料表引用的新名稱 — 但不允許在查詢中其他地方使用原始名稱來引用資料表。 因此，像這樣是無效的：

```
SELECT * FROM my_table AS m WHERE my_table.a > 5;    -- wrong
```

資料表別名主要是為了符號方便，但是在將資料表和自己交叉查詢時更有必要使用它們，例如：

```
SELECT * FROM people AS mother JOIN people AS child ON mother.id = child.mother_id;
```

此外，如果資料表引用是子查詢，則別名必要的（請向下參閱[第 7.2.1.3 節](#7213-subqueries)）。

括號可以用來解決歧義。在以下範例中，第一個查詢語句將別名 b 分配給 my\_table 的第二個實例，但第二個語句將別名分配給交叉查詢的結果：

```
SELECT * FROM my_table AS a CROSS JOIN my_table AS b ...
SELECT * FROM (my_table AS a CROSS JOIN my_table) AS b ...
```

另一種形式的資料表別名為資料表的欄位所提供的臨時名稱，就如同資料表本身的別名一樣：

```
FROM table_reference [AS] alias ( column1 [, column2 [, ...］] )
```

如果指定的欄位別名少於實際資料表中的欄位，則其餘列就不會重新命名。此語法對於自我交叉查詢或子查詢時特別有用。

將別名應用於 JOIN 子句的輸出時，別名隱藏了 JOIN 中的原始名稱。 例如：

```
SELECT a.* FROM my_table AS a JOIN your_table AS b ON ...
```

是合法的 SQL，但：

```
SELECT a.* FROM (my_table AS a JOIN your_table AS b ON ...) AS c
```

是不合法的；資料表別名 a 在別名 c 以外是不可見的。

#### 7.2.1.3. 子查詢（Subqueries）

指定衍生資料表的子查詢必須用圓括號括起來，並且必須分配一個資料表表別名（如[第 7.2.1.2 節](#7212-table-and-column-aliases)所述）。 例如：

```
FROM (SELECT * FROM table1) AS alias_name
```

這例子等同於`FROM table1 AS alias_name`。當子查詢涉及分組或彙總時，會出現更多有趣的情況，這些情況不能簡化為普通的交叉查詢。

子查詢也可以是一個 VALUES 列表：

```
FROM (VALUES ('anne', 'smith'), ('bob', 'jones'), ('joe', 'blow'))
     AS names(first, last)
```

再一次，這時候一個資料表別名是必須的。 將別名指定給 VALUES 列表的欄位是可選的，但這會是很好的做法。更多資訊請參閱[第 7.7 節](/ii-the-sql-language/queries/77-values-lists.md)。

#### 7.2.1.4. Table Functions

資料表函數是由基本資料類型（scalar types）或複合資料類型（資料表的資料列）所組成的函數。它們在查詢的 FROM 子句中用作資料表、View 或子查詢。資料表函數回傳的資料列可以與資料表、View 或子查詢的資料列相同的方式包含在 SELECT、JOIN 或 WHERE子句中。

資料表函數也可以使用 ROWS FROM 語法進行組合，結果在平行的欄位中回傳；在這種情況下結果資料列的數量是函數結果的最大數量，較小的結果填充空值來搭配。

```
function_call [WITH ORDINALITY] [[AS] table_alias [(column_alias [, ...])]]
ROWS FROM( function_call [, ...] ) [WITH ORDINALITY] [[AS] table_alias [(column_alias [, ...])]]
```

If the`WITH ORDINALITY`clause is specified, an additional column of type`bigint`will be added to the function result columns. This column numbers the rows of the function result set, starting from 1. \(This is a generalization of the SQL-standard syntax for`UNNEST ... WITH ORDINALITY`.\) By default, the ordinal column is called`ordinality`, but a different column name can be assigned to it using an`AS`clause.

The special table function`UNNEST`may be called with any number of array parameters, and it returns a corresponding number of columns, as if`UNNEST`\([Section 9.18](https://www.postgresql.org/docs/10/static/functions-array.html)\) had been called on each parameter separately and combined using the`ROWS FROM`construct.

```
UNNEST( array_expression [, ...] ) [WITH ORDINALITY] [[AS] table_alias [(column_alias [, ...])]]
```

If no\_`table_alias`\_is specified, the function name is used as the table name; in the case of a`ROWS FROM()`construct, the first function's name is used.

If column aliases are not supplied, then for a function returning a base data type, the column name is also the same as the function name. For a function returning a composite type, the result columns get the names of the individual attributes of the type.

Some examples:

```
CREATE TABLE foo (fooid int, foosubid int, fooname text);

CREATE FUNCTION getfoo(int) RETURNS SETOF foo AS $$
    SELECT * FROM foo WHERE fooid = $1;
$$ LANGUAGE SQL;

SELECT * FROM getfoo(1) AS t1;

SELECT * FROM foo
    WHERE foosubid IN (
                        SELECT foosubid
                        FROM getfoo(foo.fooid) z
                        WHERE z.fooid = foo.fooid
                      );

CREATE VIEW vw_getfoo AS SELECT * FROM getfoo(1);

SELECT * FROM vw_getfoo;
```

In some cases it is useful to define table functions that can return different column sets depending on how they are invoked. To support this, the table function can be declared as returning the pseudo-type`record`. When such a function is used in a query, the expected row structure must be specified in the query itself, so that the system can know how to parse and plan the query. This syntax looks like:

```
function_call [AS] alias (column_definition [, ...])

function_call AS [alias] (column_definition [, ...])
ROWS FROM( ... function_call AS (column_definition [, ...]) [, ...] )
```

When not using the`ROWS FROM()`syntax, the`column_definition`_\_list replaces the column alias list that could otherwise be attached to the_`FROM`_item; the names in the column definitions serve as column aliases. When using the_`ROWS FROM()`_syntax, a_`column_definition`_list can be attached to each member function separately; or if there is only one member function and no_`WITH ORDINALITY`_clause, a_`column_definition`\_list can be written in place of a column alias list following`ROWS FROM()`.

Consider this example:

```
SELECT *
    FROM dblink('dbname=mydb', 'SELECT proname, prosrc FROM pg_proc')
      AS t1(proname name, prosrc text)
    WHERE proname LIKE 'bytea%';
```

The[dblink](https://www.postgresql.org/docs/10/static/contrib-dblink-function.html)function \(part of the[dblink](https://www.postgresql.org/docs/10/static/dblink.html)module\) executes a remote query. It is declared to return`record`since it might be used for any kind of query. The actual column set must be specified in the calling query so that the parser knows, for example, what`*`should expand to.

#### 7.2.1.5. `LATERAL`Subqueries

Subqueries appearing in`FROM`can be preceded by the key word`LATERAL`. This allows them to reference columns provided by preceding`FROM`items. \(Without`LATERAL`, each subquery is evaluated independently and so cannot cross-reference any other`FROM`item.\)

Table functions appearing in`FROM`can also be preceded by the key word`LATERAL`, but for functions the key word is optional; the function's arguments can contain references to columns provided by preceding`FROM`items in any case.

A`LATERAL`item can appear at top level in the`FROM`list, or within a`JOIN`tree. In the latter case it can also refer to any items that are on the left-hand side of a`JOIN`that it is on the right-hand side of.

When a`FROM`item contains`LATERAL`cross-references, evaluation proceeds as follows: for each row of the`FROM`item providing the cross-referenced column\(s\), or set of rows of multiple`FROM`items providing the columns, the`LATERAL`item is evaluated using that row or row set's values of the columns. The resulting row\(s\) are joined as usual with the rows they were computed from. This is repeated for each row or set of rows from the column source table\(s\).

A trivial example of`LATERAL`is

```
SELECT * FROM foo, LATERAL (SELECT * FROM bar WHERE bar.id = foo.bar_id) ss;
```

This is not especially useful since it has exactly the same result as the more conventional

```
SELECT * FROM foo, bar WHERE bar.id = foo.bar_id;
```

`LATERAL`is primarily useful when the cross-referenced column is necessary for computing the row\(s\) to be joined. A common application is providing an argument value for a set-returning function. For example, supposing that`vertices(polygon)`returns the set of vertices of a polygon, we could identify close-together vertices of polygons stored in a table with:

```
SELECT p1.id, p2.id, v1, v2
FROM polygons p1, polygons p2,
     LATERAL vertices(p1.poly) v1,
     LATERAL vertices(p2.poly) v2
WHERE (v1 
<
-
>
 v2) 
<
 10 AND p1.id != p2.id;
```

This query could also be written

```
SELECT p1.id, p2.id, v1, v2
FROM polygons p1 CROSS JOIN LATERAL vertices(p1.poly) v1,
     polygons p2 CROSS JOIN LATERAL vertices(p2.poly) v2
WHERE (v1 
<
-
>
 v2) 
<
 10 AND p1.id != p2.id;
```

or in several other equivalent formulations. \(As already mentioned, the`LATERAL`key word is unnecessary in this example, but we use it for clarity.\)

It is often particularly handy to`LEFT JOIN`to a`LATERAL`subquery, so that source rows will appear in the result even if the`LATERAL`subquery produces no rows for them. For example, if`get_product_names()`returns the names of products made by a manufacturer, but some manufacturers in our table currently produce no products, we could find out which ones those are like this:

```
SELECT m.name
FROM manufacturers m LEFT JOIN LATERAL get_product_names(m.id) pname ON true
WHERE pname IS NULL;
```

### 7.2.2. The`WHERE`Clause

The syntax of the[`WHERE`Clause](https://www.postgresql.org/docs/10/static/sql-select.html#sql-where)is

```
WHERE 
search_condition
```

where\_`search_condition`\_is any value expression \(see[Section 4.2](https://www.postgresql.org/docs/10/static/sql-expressions.html)\) that returns a value of type`boolean`.

After the processing of the`FROM`clause is done, each row of the derived virtual table is checked against the search condition. If the result of the condition is true, the row is kept in the output table, otherwise \(i.e., if the result is false or null\) it is discarded. The search condition typically references at least one column of the table generated in the`FROM`clause; this is not required, but otherwise the`WHERE`clause will be fairly useless.

### Note

The join condition of an inner join can be written either in the`WHERE`clause or in the`JOIN`clause. For example, these table expressions are equivalent:

```
FROM a, b WHERE a.id = b.id AND b.val 
>
 5
```

and:

```
FROM a INNER JOIN b ON (a.id = b.id) WHERE b.val 
>
 5
```

or perhaps even:

```
FROM a NATURAL JOIN b WHERE b.val 
>
 5
```

Which one of these you use is mainly a matter of style. The`JOIN`syntax in the`FROM`clause is probably not as portable to other SQL database management systems, even though it is in the SQL standard. For outer joins there is no choice: they must be done in the`FROM`clause. The`ON`or`USING`clause of an outer join is\_not\_equivalent to a`WHERE`condition, because it results in the addition of rows \(for unmatched input rows\) as well as the removal of rows in the final result.

Here are some examples of`WHERE`clauses:

```
SELECT ... FROM fdt WHERE c1 
>
 5

SELECT ... FROM fdt WHERE c1 IN (1, 2, 3)

SELECT ... FROM fdt WHERE c1 IN (SELECT c1 FROM t2)

SELECT ... FROM fdt WHERE c1 IN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10)

SELECT ... FROM fdt WHERE c1 BETWEEN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10) AND 100

SELECT ... FROM fdt WHERE EXISTS (SELECT c1 FROM t2 WHERE c2 
>
 fdt.c1)
```

`fdt`is the table derived in the`FROM`clause. Rows that do not meet the search condition of the`WHERE`clause are eliminated from`fdt`. Notice the use of scalar subqueries as value expressions. Just like any other query, the subqueries can employ complex table expressions. Notice also how`fdt`is referenced in the subqueries. Qualifying`c1`as`fdt.c1`is only necessary if`c1`is also the name of a column in the derived input table of the subquery. But qualifying the column name adds clarity even when it is not needed. This example shows how the column naming scope of an outer query extends into its inner queries.

### 7.2.3. The`GROUP BY`and`HAVING`Clauses

After passing the`WHERE`filter, the derived input table might be subject to grouping, using the`GROUP BY`clause, and elimination of group rows using the`HAVING`clause.

```
SELECT 
select_list

    FROM ...
    [
WHERE ...
]
    GROUP BY 
grouping_column_reference
 [
, 
grouping_column_reference
]...
```

The[`GROUP BY`Clause](https://www.postgresql.org/docs/10/static/sql-select.html#sql-groupby)is used to group together those rows in a table that have the same values in all the columns listed. The order in which the columns are listed does not matter. The effect is to combine each set of rows having common values into one group row that represents all rows in the group. This is done to eliminate redundancy in the output and/or compute aggregates that apply to these groups. For instance:

```
=
>
SELECT * FROM test1;

 x | y
---+---
 a | 3
 c | 2
 b | 5
 a | 1
(4 rows)


=
>
SELECT x FROM test1 GROUP BY x;

 x
---
 a
 b
 c
(3 rows)
```

In the second query, we could not have written`SELECT * FROM test1 GROUP BY x`, because there is no single value for the column`y`that could be associated with each group. The grouped-by columns can be referenced in the select list since they have a single value in each group.

In general, if a table is grouped, columns that are not listed in`GROUP BY`cannot be referenced except in aggregate expressions. An example with aggregate expressions is:

```
=
>
SELECT x, sum(y) FROM test1 GROUP BY x;

 x | sum
---+-----
 a |   4
 b |   5
 c |   2
(3 rows)
```

Here`sum`is an aggregate function that computes a single value over the entire group. More information about the available aggregate functions can be found in[Section 9.20](https://www.postgresql.org/docs/10/static/functions-aggregate.html).

### Tip

Grouping without aggregate expressions effectively calculates the set of distinct values in a column. This can also be achieved using the`DISTINCT`clause \(see[Section 7.3.3](https://www.postgresql.org/docs/10/static/queries-select-lists.html#queries-distinct)\).

Here is another example: it calculates the total sales for each product \(rather than the total sales of all products\):

```
SELECT product_id, p.name, (sum(s.units) * p.price) AS sales
    FROM products p LEFT JOIN sales s USING (product_id)
    GROUP BY product_id, p.name, p.price;
```

In this example, the columns`product_id`,`p.name`, and`p.price`must be in the`GROUP BY`clause since they are referenced in the query select list \(but see below\). The column`s.units`does not have to be in the`GROUP BY`list since it is only used in an aggregate expression \(`sum(...)`\), which represents the sales of a product. For each product, the query returns a summary row about all sales of the product.

If the products table is set up so that, say,`product_id`is the primary key, then it would be enough to group by`product_id`in the above example, since name and price would be\_functionally dependent\_on the product ID, and so there would be no ambiguity about which name and price value to return for each product ID group.

In strict SQL,`GROUP BY`can only group by columns of the source table butPostgreSQLextends this to also allow`GROUP BY`to group by columns in the select list. Grouping by value expressions instead of simple column names is also allowed.

If a table has been grouped using`GROUP BY`, but only certain groups are of interest, the`HAVING`clause can be used, much like a`WHERE`clause, to eliminate groups from the result. The syntax is:

```
SELECT 
select_list
 FROM ... [
WHERE ...
] GROUP BY ... HAVING 
boolean_expression
```

Expressions in the`HAVING`clause can refer both to grouped expressions and to ungrouped expressions \(which necessarily involve an aggregate function\).

Example:

```
=
>
SELECT x, sum(y) FROM test1 GROUP BY x HAVING sum(y) 
>
 3;

 x | sum
---+-----
 a |   4
 b |   5
(2 rows)


=
>
SELECT x, sum(y) FROM test1 GROUP BY x HAVING x 
<
 'c';

 x | sum
---+-----
 a |   4
 b |   5
(2 rows)
```

Again, a more realistic example:

```
SELECT product_id, p.name, (sum(s.units) * (p.price - p.cost)) AS profit
    FROM products p LEFT JOIN sales s USING (product_id)
    WHERE s.date 
>
 CURRENT_DATE - INTERVAL '4 weeks'
    GROUP BY product_id, p.name, p.price, p.cost
    HAVING sum(p.price * s.units) 
>
 5000;
```

In the example above, the`WHERE`clause is selecting rows by a column that is not grouped \(the expression is only true for sales during the last four weeks\), while the`HAVING`clause restricts the output to groups with total gross sales over 5000. Note that the aggregate expressions do not necessarily need to be the same in all parts of the query.

If a query contains aggregate function calls, but no`GROUP BY`clause, grouping still occurs: the result is a single group row \(or perhaps no rows at all, if the single row is then eliminated by`HAVING`\). The same is true if it contains a`HAVING`clause, even without any aggregate function calls or`GROUP BY`clause.

### 7.2.4. `GROUPING SETS`,`CUBE`, and`ROLLUP`

More complex grouping operations than those described above are possible using the concept of_grouping sets_. The data selected by the`FROM`and`WHERE`clauses is grouped separately by each specified grouping set, aggregates computed for each group just as for simple`GROUP BY`clauses, and then the results returned. For example:

```
=
>
SELECT * FROM items_sold;

 brand | size | sales
-------+------+-------
 Foo   | L    |  10
 Foo   | M    |  20
 Bar   | M    |  15
 Bar   | L    |  5
(4 rows)


=
>
SELECT brand, size, sum(sales) FROM items_sold GROUP BY GROUPING SETS ((brand), (size), ());

 brand | size | sum
-------+------+-----
 Foo   |      |  30
 Bar   |      |  20
       | L    |  15
       | M    |  35
       |      |  50
(5 rows)
```

Each sublist of`GROUPING SETS`may specify zero or more columns or expressions and is interpreted the same way as though it were directly in the`GROUP BY`clause. An empty grouping set means that all rows are aggregated down to a single group \(which is output even if no input rows were present\), as described above for the case of aggregate functions with no`GROUP BY`clause.

References to the grouping columns or expressions are replaced by null values in result rows for grouping sets in which those columns do not appear. To distinguish which grouping a particular output row resulted from, see[Table 9.56](https://www.postgresql.org/docs/10/static/functions-aggregate.html#functions-grouping-table).

A shorthand notation is provided for specifying two common types of grouping set. A clause of the form

```
ROLLUP ( 
e1
, 
e2
, 
e3
, ... )
```

represents the given list of expressions and all prefixes of the list including the empty list; thus it is equivalent to

```
GROUPING SETS (
    ( 
e1
, 
e2
, 
e3
, ... ),
    ...
    ( 
e1
, 
e2
 ),
    ( 
e1
 ),
    ( )
)
```

This is commonly used for analysis over hierarchical data; e.g. total salary by department, division, and company-wide total.

A clause of the form

```
CUBE ( 
e1
, 
e2
, ... )
```

represents the given list and all of its possible subsets \(i.e. the power set\). Thus

```
CUBE ( a, b, c )
```

is equivalent to

```
GROUPING SETS (
    ( a, b, c ),
    ( a, b    ),
    ( a,    c ),
    ( a       ),
    (    b, c ),
    (    b    ),
    (       c ),
    (         )
)
```

The individual elements of a`CUBE`or`ROLLUP`clause may be either individual expressions, or sublists of elements in parentheses. In the latter case, the sublists are treated as single units for the purposes of generating the individual grouping sets. For example:

```
CUBE ( (a, b), (c, d) )
```

is equivalent to

```
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b       ),
    (       c, d ),
    (            )
)
```

and

```
ROLLUP ( a, (b, c), d )
```

is equivalent to

```
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b, c    ),
    ( a          ),
    (            )
)
```

The`CUBE`and`ROLLUP`constructs can be used either directly in the`GROUP BY`clause, or nested inside a`GROUPING SETS`clause. If one`GROUPING SETS`clause is nested inside another, the effect is the same as if all the elements of the inner clause had been written directly in the outer clause.

If multiple grouping items are specified in a single`GROUP BY`clause, then the final list of grouping sets is the cross product of the individual items. For example:

```
GROUP BY a, CUBE (b, c), GROUPING SETS ((d), (e))
```

is equivalent to

```
GROUP BY GROUPING SETS (
    (a, b, c, d), (a, b, c, e),
    (a, b, d),    (a, b, e),
    (a, c, d),    (a, c, e),
    (a, d),       (a, e)
)
```

### Note

The construct`(a, b)`is normally recognized in expressions as a[row constructor](https://www.postgresql.org/docs/10/static/sql-expressions.html#sql-syntax-row-constructors). Within the`GROUP BY`clause, this does not apply at the top levels of expressions, and`(a, b)`is parsed as a list of expressions as described above. If for some reason you\_need\_a row constructor in a grouping expression, use`ROW(a, b)`.

### 7.2.5. Window Function Processing

If the query contains any window functions \(see[Section 3.5](https://www.postgresql.org/docs/10/static/tutorial-window.html),[Section 9.21](https://www.postgresql.org/docs/10/static/functions-window.html)and[Section 4.2.8](https://www.postgresql.org/docs/10/static/sql-expressions.html#syntax-window-functions)\), these functions are evaluated after any grouping, aggregation, and`HAVING`filtering is performed. That is, if the query uses any aggregates,`GROUP BY`, or`HAVING`, then the rows seen by the window functions are the group rows instead of the original table rows from`FROM`/`WHERE`.

When multiple window functions are used, all the window functions having syntactically equivalent`PARTITION BY`and`ORDER BY`clauses in their window definitions are guaranteed to be evaluated in a single pass over the data. Therefore they will see the same sort ordering, even if the`ORDER BY`does not uniquely determine an ordering. However, no guarantees are made about the evaluation of functions having different`PARTITION BY`or`ORDER BY`specifications. \(In such cases a sort step is typically required between the passes of window function evaluations, and the sort is not guaranteed to preserve ordering of rows that its`ORDER BY`sees as equivalent.\)

Currently, window functions always require presorted data, and so the query output will be ordered according to one or another of the window functions'`PARTITION BY`/`ORDER BY`clauses. It is not recommended to rely on this, however. Use an explicit top-level`ORDER BY`clause if you want to be sure the results are sorted in a particular way.

---

[^1]: [PostgreSQL: Documentation: 10: 7.2. Table Expressions](https://www.postgresql.org/docs/10/static/queries-table-expressions.html)

