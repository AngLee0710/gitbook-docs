# 8.1. 數字型別[^1]

數字型別由兩位數，四位數和八位數整數，四位元組和八位元組的浮點數以及可調式精確度的小數組成。表格 8.2 列出了可用的類型。

**Table 8.2. Numeric Types**

| Name | Storage Size | Description | Range |
| :--- | :--- | :--- | :--- |
| `smallint` | 2 bytes | small-range integer | -32768 to +32767 |
| `integer` | 4 bytes | typical choice for integer | -2147483648 to +2147483647 |
| `bigint` | 8 bytes | large-range integer | -9223372036854775808 to +9223372036854775807 |
| `decimal` | variable | user-specified precision, exact | up to 131072 digits before the decimal point; up to 16383 digits after the decimal point |
| `numeric` | variable | user-specified precision, exact | up to 131072 digits before the decimal point; up to 16383 digits after the decimal point |
| `real` | 4 bytes | variable-precision, inexact | 6 decimal digits precision |
| `double precision` | 8 bytes | variable-precision, inexact | 15 decimal digits precision |
| `smallserial` | 2 bytes | small autoincrementing integer | 1 to 32767 |
| `serial` | 4 bytes | autoincrementing integer | 1 to 2147483647 |
| `bigserial` | 8 bytes | large autoincrementing integer | 1 to 9223372036854775807 |

[4.1.2 節](/ii-the-sql-language/sql-syntax/41-lexical-structure.md)描述了數字型別常數的語法。 數字型別有一整套相應的算術運算元和函數。有關更多訊息，請參閱第 9 章。 以下各節將詳細介紹這些型別。

### 8.1.1. Integer Types（整數型別）

smallint、integer 和 bigint 型別儲存整數，即不包含小數部分的各種範圍的數字。嘗試儲存在允許的範圍之外的數值將會導致錯誤。

「integer」型別是常見的選擇，因為它提供了數值範圍、儲存空間及效能之間的最佳平衡。「smallint」 列別通常只在磁碟空間不足的情況下使用。「bigint」 型別被設計用於整數型別的範圍不足時。

SQL僅指定整數型別 integer（或 int）、smallint 和 bigint。 型別名稱 int2、int4 和 int8 則是延伸型別，也有一些其他 SQL 資料庫系統使用。

### 8.1.2. Arbitrary Precision Numbers

數字型別可以儲存很多位數的數字。特別建議使用在要求正確性的地方，像是儲存貨幣金額或其他數量。使用數值的計算在可能需要的情況下得到確切的結果，例如 加法、減法、乘法。但是，與整數型別或下一節中介紹的浮點型別相比，對數值的計算速度非常緩慢。

我們使用下面的術語：數字的「scale」是小數點右邊的小數部分，也就是小數的位數。數字的「precision」是整數中有效位數的總數，即小數點兩邊的位數總合。所以 23.5141 的 precision 是 6，scale 是 4。整數可以被認為是 scale 為 0。

可以配置數字欄位的最大 precision 和最大 scale。要宣告數字型別的欄位，請使用以下語法：

```
NUMERIC(precision, scale)
```

precision 必須是正值，scale 為零或正值。或是：

```
NUMERIC(precision)
```

選擇 0 為 scale。這樣使用：

```
NUMERIC
```

沒有任何 precision 或 scale 的話，就會建立一個欄位，其欄位中可以儲存任何 precision 和 scale 的數字值，直到達到 precision 的極限。這種型別的欄位不會將輸入值強制轉為任何特定的 scale，其中具有聲明比例的數字欄位會將輸入值強制為該 scale。 （SQL 標準需要預設 scale 為 0，即強制為整數精度，我們發現這樣做有點無用。如果你擔心可移植性，請務必明確指定 precision 和 scale。

> ### 注意
>
> 在型別宣告中明確指定時允許的最大 precision 為 1000；沒有指定 precision 的NUMERIC 為 Table 8.2 中所述的限制。

If the scale of a value to be stored is greater than the declared scale of the column, the system will round the value to the specified number of fractional digits. Then, if the number of digits to the left of the decimal point exceeds the declared precision minus the declared scale, an error is raised.

Numeric values are physically stored without any extra leading or trailing zeroes. Thus, the declared precision and scale of a column are maximums, not fixed allocations. \(In this sense the`numeric`type is more akin to`varchar(n`\)than to`char(n`\).\) The actual storage requirement is two bytes for each group of four decimal digits, plus three to eight bytes overhead.

In addition to ordinary numeric values, the`numeric`type allows the special value`NaN`, meaning“not-a-number”. Any operation on`NaN`yields another`NaN`. When writing this value as a constant in an SQL command, you must put quotes around it, for example`UPDATE table SET x = 'NaN'`. On input, the string`NaN`is recognized in a case-insensitive manner.

### Note

In most implementations of the“not-a-number”concept,`NaN`is not considered equal to any other numeric value \(including`NaN`\). In order to allow`numeric`values to be sorted and used in tree-based indexes,PostgreSQLtreats`NaN`values as equal, and greater than all non-`NaN`values.

The types`decimal`and`numeric`are equivalent. Both types are part of theSQLstandard.

When rounding values, the`numeric`type rounds ties away from zero, while \(on most machines\) the`real`and`double precision`types round ties to the nearest even number. For example:

```
SELECT x,
  round(x::numeric) AS num_round,
  round(x::double precision) AS dbl_round
FROM generate_series(-3.5, 3.5, 1) as x;
  x   | num_round | dbl_round
------+-----------+-----------
 -3.5 |        -4 |        -4
 -2.5 |        -3 |        -2
 -1.5 |        -2 |        -2
 -0.5 |        -1 |        -0
  0.5 |         1 |         0
  1.5 |         2 |         2
  2.5 |         3 |         2
  3.5 |         4 |         4
(8 rows)
```

### 8.1.3. Floating-Point Types

The data types`real`and`double precision`are inexact, variable-precision numeric types. In practice, these types are usually implementations ofIEEEStandard 754 for Binary Floating-Point Arithmetic \(single and double precision, respectively\), to the extent that the underlying processor, operating system, and compiler support it.

Inexact means that some values cannot be converted exactly to the internal format and are stored as approximations, so that storing and retrieving a value might show slight discrepancies. Managing these errors and how they propagate through calculations is the subject of an entire branch of mathematics and computer science and will not be discussed here, except for the following points:

* If you require exact storage and calculations \(such as for monetary amounts\), use the`numeric`type instead.

* If you want to do complicated calculations with these types for anything important, especially if you rely on certain behavior in boundary cases \(infinity, underflow\), you should evaluate the implementation carefully.

* Comparing two floating-point values for equality might not always work as expected.

On most platforms, the`real`type has a range of at least 1E-37 to 1E+37 with a precision of at least 6 decimal digits. The`double precision`type typically has a range of around 1E-307 to 1E+308 with a precision of at least 15 digits. Values that are too large or too small will cause an error. Rounding might take place if the precision of an input number is too high. Numbers too close to zero that are not representable as distinct from zero will cause an underflow error.

### Note

The[extra\_float\_digits](https://www.postgresql.org/docs/10/static/runtime-config-client.html#guc-extra-float-digits)setting controls the number of extra significant digits included when a floating point value is converted to text for output. With the default value of`0`, the output is the same on every platform supported by PostgreSQL. Increasing it will produce output that more accurately represents the stored value, but may be unportable.

In addition to ordinary numeric values, the floating-point types have several special values:

`Infinity`  
`-Infinity`  
`NaN`

These represent the IEEE 754 special values“infinity”,“negative infinity”, and“not-a-number”, respectively. \(On a machine whose floating-point arithmetic does not follow IEEE 754, these values will probably not work as expected.\) When writing these values as constants in an SQL command, you must put quotes around them, for example`UPDATE table SET x = '-Infinity'`. On input, these strings are recognized in a case-insensitive manner.

### Note

IEEE754 specifies that`NaN`should not compare equal to any other floating-point value \(including`NaN`\). In order to allow floating-point values to be sorted and used in tree-based indexes,PostgreSQLtreats`NaN`values as equal, and greater than all non-`NaN`values.

PostgreSQLalso supports the SQL-standard notations`float`and`float(p`\)for specifying inexact numeric types. Here,`p`_\_specifies the minimum acceptable precision in\_binary\_digits.PostgreSQLaccepts_`float(1)`_to_`float(24)`_as selecting the_`real`_type, while_`float(25)`_to_`float(53)`_select_`double precision`_. Values of_`p`\_outside the allowed range draw an error.`float`with no precision specified is taken to mean`double precision`.

### Note

The assumption that`real`and`double precision`have exactly 24 and 53 bits in the mantissa respectively is correct for IEEE-standard floating point implementations. On non-IEEE platforms it might be off a little, but for simplicity the same ranges of\_`p`\_are used on all platforms.

### 8.1.4. Serial Types

The data types`smallserial`,`serial`and`bigserial`are not true types, but merely a notational convenience for creating unique identifier columns \(similar to the`AUTO_INCREMENT`property supported by some other databases\). In the current implementation, specifying:

```
CREATE TABLE 
tablename
 (

colname
 SERIAL
);
```

is equivalent to specifying:

```
CREATE SEQUENCE 
tablename
_
colname
_seq;
CREATE TABLE 
tablename
 (

colname
 integer NOT NULL DEFAULT nextval('
tablename
_
colname
_seq')
);
ALTER SEQUENCE 
tablename
_
colname
_seq OWNED BY 
tablename
.
colname
;
```

Thus, we have created an integer column and arranged for its default values to be assigned from a sequence generator. A`NOT NULL`constraint is applied to ensure that a null value cannot be inserted. \(In most cases you would also want to attach a`UNIQUE`or`PRIMARY KEY`constraint to prevent duplicate values from being inserted by accident, but this is not automatic.\) Lastly, the sequence is marked as“owned by”the column, so that it will be dropped if the column or table is dropped.

### Note

Because`smallserial`,`serial`and`bigserial`are implemented using sequences, there may be "holes" or gaps in the sequence of values which appears in the column, even if no rows are ever deleted. A value allocated from the sequence is still "used up" even if a row containing that value is never successfully inserted into the table column. This may happen, for example, if the inserting transaction rolls back. See`nextval()`in[Section 9.16](https://www.postgresql.org/docs/10/static/functions-sequence.html)for details.

To insert the next value of the sequence into the`serial`column, specify that the`serial`column should be assigned its default value. This can be done either by excluding the column from the list of columns in the`INSERT`statement, or through the use of the`DEFAULT`key word.

The type names`serial`and`serial4`are equivalent: both create`integer`columns. The type names`bigserial`and`serial8`work the same way, except that they create a`bigint`column.`bigserial`should be used if you anticipate the use of more than 231identifiers over the lifetime of the table. The type names`smallserial`and`serial2`also work the same way, except that they create a`smallint`column.

The sequence created for a`serial`column is automatically dropped when the owning column is dropped. You can drop the sequence without dropping the column, but this will force removal of the column default expression.

---

[^1]: [PostgreSQL: Documentation: 10: 8.1. Numeric Types](https://www.postgresql.org/docs/10/static/datatype-numeric.html)

