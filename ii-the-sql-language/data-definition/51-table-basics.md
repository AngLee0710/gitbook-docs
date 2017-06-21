# 5.1. 認識表格[^1]

A table in a relational database is much like a table on paper: It consists of rows and columns. The number and order of the columns is fixed, and each column has a name. The number of rows is variable — it reflects how much data is stored at a given moment. SQL does not make any guarantees about the order of the rows in a table. When a table is read, the rows will appear in an unspecified order, unless sorting is explicitly requested. This is covered in[Chapter 7](https://www.postgresql.org/docs/10/static/queries.html). Furthermore, SQL does not assign unique identifiers to rows, so it is possible to have several completely identical rows in a table. This is a consequence of the mathematical model that underlies SQL but is usually not desirable. Later in this chapter we will see how to deal with this issue.

Each column has a data type. The data type constrains the set of possible values that can be assigned to a column and assigns semantics to the data stored in the column so that it can be used for computations. For instance, a column declared to be of a numerical type will not accept arbitrary text strings, and the data stored in such a column can be used for mathematical computations. By contrast, a column declared to be of a character string type will accept almost any kind of data but it does not lend itself to mathematical calculations, although other operations such as string concatenation are available.

PostgreSQLincludes a sizable set of built-in data types that fit many applications. Users can also define their own data types. Most built-in data types have obvious names and semantics, so we defer a detailed explanation to[Chapter 8](https://www.postgresql.org/docs/10/static/datatype.html). Some of the frequently used data types are`integer`for whole numbers,`numeric`for possibly fractional numbers,`text`for character strings,`date`for dates,`time`for time-of-day values, and`timestamp`for values containing both date and time.



To create a table, you use the aptly named[CREATE TABLE](https://www.postgresql.org/docs/10/static/sql-createtable.html)command. In this command you specify at least a name for the new table, the names of the columns and the data type of each column. For example:

```
CREATE TABLE my_first_table (
    first_column text,
    second_column integer
);

```

This creates a table named`my_first_table`with two columns. The first column is named`first_column`and has a data type of`text`; the second column has the name`second_column`and the type`integer`. The table and column names follow the identifier syntax explained in[Section 4.1.1](https://www.postgresql.org/docs/10/static/sql-syntax-lexical.html#sql-syntax-identifiers). The type names are usually also identifiers, but there are some exceptions. Note that the column list is comma-separated and surrounded by parentheses.

Of course, the previous example was heavily contrived. Normally, you would give names to your tables and columns that convey what kind of data they store. So let's look at a more realistic example:

```
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric
);

```

\(The`numeric`type can store fractional components, as would be typical of monetary amounts.\)

### Tip

When you create many interrelated tables it is wise to choose a consistent naming pattern for the tables and columns. For instance, there is a choice of using singular or plural nouns for table names, both of which are favored by some theorist or other.

There is a limit on how many columns a table can contain. Depending on the column types, it is between 250 and 1600. However, defining a table with anywhere near this many columns is highly unusual and often a questionable design.



If you no longer need a table, you can remove it using the[DROP TABLE](https://www.postgresql.org/docs/10/static/sql-droptable.html)command. For example:

```
DROP TABLE my_first_table;
DROP TABLE products;

```

Attempting to drop a table that does not exist is an error. Nevertheless, it is common in SQL script files to unconditionally try to drop each table before creating it, ignoring any error messages, so that the script works whether or not the table exists. \(If you like, you can use the`DROP TABLE IF EXISTS`variant to avoid the error messages, but this is not standard SQL.\)

If you need to modify a table that already exists, see[Section 5.5](https://www.postgresql.org/docs/10/static/ddl-alter.html)later in this chapter.

With the tools discussed so far you can create fully functional tables. The remainder of this chapter is concerned with adding features to the table definition to ensure data integrity, security, or convenience. If you are eager to fill your tables with data now you can skip ahead to[Chapter 6](https://www.postgresql.org/docs/10/static/dml.html)and read the rest of this chapter later.

---

[^1]: [PostgreSQL: Documentation: 10: 5.1. Table Basics](https://www.postgresql.org/docs/10/static/ddl-basics.html)

