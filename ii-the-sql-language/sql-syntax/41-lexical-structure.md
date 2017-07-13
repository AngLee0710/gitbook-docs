# 4.1. 語法結構[^1]

SQL 語法包含一連串的命令，命令是由一系列的指示記號所組合而成，以分號結尾。最後如果是串流輸入，也會結束一個命令。指示的合法性是由特別的命令語法所定義的。

指示記號可能是關鍵字、識別項、引號識別項、文字、或一個特別的字元符號。指示一般來說是以空白分隔（空白符號、定位符號、換行符號），但如果不會混淆的話，也不一定需要。（一般只出現在特殊字元用來調整了其他指示的型別）

舉個例子，下面就是一個合法（符合語法）的 SQL 輸入：

```
SELECT * FROM MY_TABLE;
UPDATE MY_TABLE SET A = 5;
INSERT INTO MY_TABLE VALUES (3, 'hi there');
```

這個序列包含了 3 個命令，每行一個（然而這不是一定的，同一行可以超過一個命令，而一個命令也可以分解為多行使用）。

順帶一提的是，註解也是 SQL 輸入的一部份，但不屬於任何指示記號，他們等同於空白字元。

SQL 語法並不是很嚴格要求什麼樣的指示記號來識別命令，或是哪些是運算子或參數。通常最前面的指示記號是命令的名稱，以上面的例子來說，我們通常會說是一個「SELECT」、一個「UPDATE」、以及一個「INSERT」命令。但對於 UPDATE 命令而言，有一個 SET 指示記號出現在某個地方是必要的；同樣地，INSERT 也需要有 VALUES 來搭配。精確的語法規則都在[第 6 部份](/vi-reference.md)中的章節進行說明。

### 4.1.1. 識別項（Identifier）和關鍵字 （Keyword）

在上面的例子中的 SELECT、UPDATE、或是 VALUES，都是屬於關鍵字的範圍。所謂關鍵字，意即在 SQL 語言中，其具有固定的意義。像指示記號 MY\_TABLE 則是屬於識別項。它識別表格的名稱，欄位名稱，或是其他的資料庫物件，端看命令如何看待該識別項。然而，有時候它們會被簡稱為「名稱」。關鍵字和識別項的文法結構是相同的，意即不看整個命令的話，是無法辨別到底是識別項還是關鍵字的。完整的關鍵字列表，收錄在附件 C 當中。

SQL 識別項與關鍵字必須以英文字母開頭（a - z，也可以是附加符號和非拉丁字母，中文沒問題）或是底線（\_）。剩餘的字元可以是字母、底線、數字（0 - 9）、或錢字號（$）。注意錢字號，在標準 SQL 語法中是不允許使用的，所以可能會降低一些應用程式的可攜性。標準 SQL 也沒有定義包含數字或是以底線起迄的關鍵字，所以識別項這樣的形式定義是安全的，不會和標準未來的修訂相衝突。

資料庫系統不能使用長度超過 NAMEDATALEN -1 的識別項；太長的名稱仍然可以在命令中被輸入，但會被截斷。預設上，NAMEDATALEN 的設定是 64，所以最長的識別項名稱長度是 63 位元組。如果這個限制會造成困擾的話，你也可以調整 NAMEDATALEN 的編譯值，它的設定在 src/include/pg\_config\_manual.h 檔案中。

關鍵字和無引號識別項都是不分大小寫的，所以：

```
UPDATE MY_TABLE SET A = 5;
```

等同於：

```
uPDaTE my_TabLE SeT a = 5;
```

有一種寫法很常使用，就是把關鍵字用大寫表示，而識別項名稱使用小寫，例如：

```
UPDATE my_table SET a = 5;
```

第二種要介紹的識別項是，受限制的識別項，或是引號識別項。它的形式就是以雙引號括住的任何字串。受限制的識別項，就一定是識別項，不會是關鍵字。所以，「"select"」就會被識別為名稱為「select」的表格或欄位，而無引號的 select 就會被視為是關鍵字，也可能會產生解譯錯誤，如果剛好用在可能是表格或欄位名稱的位置上的話。使用引號識別項的例子如下：

```
UPDATE "my_table" SET "a" = 5;
```

引號識別項可以包含任何字元，除了字元碼為 0 的字元以外。（要包含雙引號字元的話，請使用連續兩個雙引號。）這可以用來建立原來不能使用的表格或欄位名稱，甚至是包含空白或＂&＂。但長度的限制仍然要遵守。

還有一種變形的引號識別項，允許包含跳脫的形式來表現萬國碼（unicode）。這種變形會以「U&」開頭（U大小寫皆可）緊接在前面的雙引號的前面，不能有任何空白在它們之間，例如：U&"foo"。（注意，這可能會和運算子的 & 產生混淆，但可以在運算子的 & 前後都加上空白來避免這個問題。）在雙引號內，萬國碼字元以跳脫的形式表現，也就是以倒斜線再接 4 位數的 16 進位碼，或倒斜線接一個加號再串一組 6 位數的 16 進位碼。例如，識別項 "data" 可以寫成這樣：

```
U&"d\0061t\+000061"
```

下面是稍微不簡明的例子是，俄文的＂slon＂（大象），以希伯萊文字母表現：

The following less trivial example writes the Russian word“slon”\(elephant\) in Cyrillic letters:

```
U&"\0441\043B\043E\043D"
```

如果希望以不同的跳脫字元來代替倒斜線的話，那麼可以雙引號結束後使用 UESCAPE 子句來指定，舉例來說：

```
U&"d!0061t!+000061" UESCAPE '!'
```

跳脫字元可以是任何的單一字元，除了 16 進位數字的字元、單引號、雙引號、或空白以外。注意指定的跳脫字元是以單引號括住，而不是雙引號。

內容要使用到跳脫字元的話，就重覆輸入 2 次。

萬國碼的跳脫語法，只能使用 UTF8 的編碼。如果有用到其他的編碼的話，只有在 ASCII 範圍（最大為 \007F）可以使用。4 位數及 6 位數的形式，可以組合配對用來指定 UTF-16 中，大於 U+FFFF 的字元，雖然 6 位數的形式單獨就可以解決這個問題（組合配對並不會直接被儲存起來，他們會被編碼成 UTF-8 再儲存。）

把識別項用引號括起來也可以用來保持它的大小寫狀態，沒有括起來的話，都會被轉成小寫字母。舉例來說，對 PostgreSQL 而言，FOO、foo、"foo"，三者都是一樣的，但 "Foo" 和 "FOO" 就彼此及前面三者都視為不同。（在 PostgreSQL 中，把未引號括起的名稱轉成小寫，並不是 SQL 的標準。SQL 標準反而是都轉成大寫。所以在 SQL 標準中，foo 應該是等同於 "FOO" 而不同於 "foo"。如果你要增加語法的可攜性的話，建議最好都使用引號括起特別的名稱，或者都不要使用引號。）

### 4.1.2. 常數

PostgreSQL 中有三種隱含型別的常數：字串、位元字串、和數值。常數也可以強制型別，有助於更精確的表達，也可以讓系統處理更有效率。接下來就開始進行相關的說明。

#### 4.1.2.1. 字串常數

在 SQL 中，所謂的字串常數，指的是用單引號括住的任意字元串列，例如：'This is a string'。如果在字串常數內需要有單引號的話就使用連續兩個單引號，例如：'Dianne''s horse'。注意這不是雙引號，是兩個單引號。

兩個字串常數如果只用空白及至少一個換行符號所分隔的話，那個它們會被連在一起，和寫成一個字串是一樣的。舉例來說：

```
SELECT 'foo'
'bar';
```

等同於：

```
SELECT 'foobar';
```

但如果是這樣：

```
SELECT 'foo'      'bar';
```

語法上就不正確了。（這是來自於 SQL 奇怪的常規，PostgreSQL 單純只是遵循。）

#### 4.1.2.2. C 語言樣式的跳脫字串常數

PostgreSQL 也支援跳脫字串常數，這些是 SQL 標準的延伸。跳脫字串常數使用的是字母 E （大小寫皆可），緊接著單引號所組成，例如：E'foo'。（如果字串有超過一行的話，也只要在第一個單引號前有 E 就可以了。）在跳脫字串當中，使用倒斜線開頭，就可以使用 C 語言式的倒斜線跳脫字串，通常是一個倒斜線再接一個字元，對應到一個特殊位元組的值，如 Table 4.1 所示。

**Table 4.1. 倒斜線跳腳字串（Backslash Escape Sequence）**

| **倒斜線跳腳字串** | 字元意義 |
| :--- | :--- |
| `\b` | backspace |
| `\f` | form feed |
| `\n` | newline |
| `\r` | carriage return |
| `\t` | tab |
| `\o`,`\oo`,`\ooo`\(`o`= 0 - 7\) | octal byte value |
| `\xh`,`\xhh`\(`h`= 0 - 9, A - F\) | hexadecimal byte value |
| `\uxxxx`,`\Uxxxxxxxx`\(`x`= 0 - 9, A - F\) | 16 or 32-bit hexadecimal Unicode character value |

Any other character following a backslash is taken literally. Thus, to include a backslash character, write two backslashes \(`\\`\). Also, a single quote can be included in an escape string by writing`\'`, in addition to the normal way of`''`.

It is your responsibility that the byte sequences you create, especially when using the octal or hexadecimal escapes, compose valid characters in the server character set encoding. When the server encoding is UTF-8, then the Unicode escapes or the alternative Unicode escape syntax, explained in[Section 4.1.2.3](https://www.postgresql.org/docs/10/static/sql-syntax-lexical.html#sql-syntax-strings-uescape), should be used instead. \(The alternative would be doing the UTF-8 encoding by hand and writing out the bytes, which would be very cumbersome.\)

The Unicode escape syntax works fully only when the server encoding is`UTF8`. When other server encodings are used, only code points in the ASCII range \(up to`\u007F`\) can be specified. Both the 4-digit and the 8-digit form can be used to specify UTF-16 surrogate pairs to compose characters with code points larger than U+FFFF, although the availability of the 8-digit form technically makes this unnecessary. \(When surrogate pairs are used when the server encoding is`UTF8`, they are first combined into a single code point that is then encoded in UTF-8.\)

### Caution

If the configuration parameter[standard\_conforming\_strings](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html#guc-standard-conforming-strings)is`off`, thenPostgreSQLrecognizes backslash escapes in both regular and escape string constants. However, as ofPostgreSQL9.1, the default is`on`, meaning that backslash escapes are recognized only in escape string constants. This behavior is more standards-compliant, but might break applications which rely on the historical behavior, where backslash escapes were always recognized. As a workaround, you can set this parameter to`off`, but it is better to migrate away from using backslash escapes. If you need to use a backslash escape to represent a special character, write the string constant with an`E`.

In addition to`standard_conforming_strings`, the configuration parameters[escape\_string\_warning](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html#guc-escape-string-warning)and[backslash\_quote](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html#guc-backslash-quote)govern treatment of backslashes in string constants.

The character with the code zero cannot be in a string constant.

#### 4.1.2.3. String Constants with Unicode Escapes

PostgreSQLalso supports another type of escape syntax for strings that allows specifying arbitrary Unicode characters by code point. A Unicode escape string constant starts with`U&`\(upper or lower case letter U followed by ampersand\) immediately before the opening quote, without any spaces in between, for example`U&'foo'`. \(Note that this creates an ambiguity with the operator`&`. Use spaces around the operator to avoid this problem.\) Inside the quotes, Unicode characters can be specified in escaped form by writing a backslash followed by the four-digit hexadecimal code point number or alternatively a backslash followed by a plus sign followed by a six-digit hexadecimal code point number. For example, the string`'data'`could be written as

```
U
&
'd\0061t\+000061'
```

The following less trivial example writes the Russian word“slon”\(elephant\) in Cyrillic letters:

```
U
&
'\0441\043B\043E\043D'
```

If a different escape character than backslash is desired, it can be specified using the`UESCAPE`clause after the string, for example:

```
U
&
'd!0061t!+000061' UESCAPE '!'
```

The escape character can be any single character other than a hexadecimal digit, the plus sign, a single quote, a double quote, or a whitespace character.

The Unicode escape syntax works only when the server encoding is`UTF8`. When other server encodings are used, only code points in the ASCII range \(up to`\007F`\) can be specified. Both the 4-digit and the 6-digit form can be used to specify UTF-16 surrogate pairs to compose characters with code points larger than U+FFFF, although the availability of the 6-digit form technically makes this unnecessary. \(When surrogate pairs are used when the server encoding is`UTF8`, they are first combined into a single code point that is then encoded in UTF-8.\)

Also, the Unicode escape syntax for string constants only works when the configuration parameter[standard\_conforming\_strings](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html#guc-standard-conforming-strings)is turned on. This is because otherwise this syntax could confuse clients that parse the SQL statements to the point that it could lead to SQL injections and similar security issues. If the parameter is set to off, this syntax will be rejected with an error message.

To include the escape character in the string literally, write it twice.

#### 4.1.2.4. Dollar-quoted String Constants

While the standard syntax for specifying string constants is usually convenient, it can be difficult to understand when the desired string contains many single quotes or backslashes, since each of those must be doubled. To allow more readable queries in such situations,PostgreSQLprovides another way, called“dollar quoting”, to write string constants. A dollar-quoted string constant consists of a dollar sign \(`$`\), an optional“tag”of zero or more characters, another dollar sign, an arbitrary sequence of characters that makes up the string content, a dollar sign, the same tag that began this dollar quote, and a dollar sign. For example, here are two different ways to specify the string“Dianne's horse”using dollar quoting:

```
$$Dianne's horse$$
$SomeTag$Dianne's horse$SomeTag$
```

Notice that inside the dollar-quoted string, single quotes can be used without needing to be escaped. Indeed, no characters inside a dollar-quoted string are ever escaped: the string content is always written literally. Backslashes are not special, and neither are dollar signs, unless they are part of a sequence matching the opening tag.

It is possible to nest dollar-quoted string constants by choosing different tags at each nesting level. This is most commonly used in writing function definitions. For example:

```
$function$
BEGIN
    RETURN ($1 ~ $q$[\t\r\n\v\\]$q$);
END;
$function$
```

Here, the sequence`$q$[\t\r\n\v\\]$q$`represents a dollar-quoted literal string`[\t\r\n\v\\]`, which will be recognized when the function body is executed byPostgreSQL. But since the sequence does not match the outer dollar quoting delimiter`$function$`, it is just some more characters within the constant so far as the outer string is concerned.

The tag, if any, of a dollar-quoted string follows the same rules as an unquoted identifier, except that it cannot contain a dollar sign. Tags are case sensitive, so`$tag$String content$tag$`is correct, but`$TAG$String content$tag$`is not.

A dollar-quoted string that follows a keyword or identifier must be separated from it by whitespace; otherwise the dollar quoting delimiter would be taken as part of the preceding identifier.

Dollar quoting is not part of the SQL standard, but it is often a more convenient way to write complicated string literals than the standard-compliant single quote syntax. It is particularly useful when representing string constants inside other constants, as is often needed in procedural function definitions. With single-quote syntax, each backslash in the above example would have to be written as four backslashes, which would be reduced to two backslashes in parsing the original string constant, and then to one when the inner string constant is re-parsed during function execution.

#### 4.1.2.5. Bit-string Constants

Bit-string constants look like regular string constants with a`B`\(upper or lower case\) immediately before the opening quote \(no intervening whitespace\), e.g.,`B'1001'`. The only characters allowed within bit-string constants are`0`and`1`.

Alternatively, bit-string constants can be specified in hexadecimal notation, using a leading`X`\(upper or lower case\), e.g.,`X'1FF'`. This notation is equivalent to a bit-string constant with four binary digits for each hexadecimal digit.

Both forms of bit-string constant can be continued across lines in the same way as regular string constants. Dollar quoting cannot be used in a bit-string constant.

#### 4.1.2.6. Numeric Constants

Numeric constants are accepted in these general forms:

```
digits
digits
.[
digits
][
e[
+-
]
digits
]
[
digits
].
digits
[
e[
+-
]
digits
]

digits
e[
+-
]
digits
```

where\_`digits`\_is one or more decimal digits \(0 through 9\). At least one digit must be before or after the decimal point, if one is used. At least one digit must follow the exponent marker \(`e`\), if one is present. There cannot be any spaces or other characters embedded in the constant. Note that any leading plus or minus sign is not actually considered part of the constant; it is an operator applied to the constant.

These are some examples of valid numeric constants:

42  
3.5  
4.  
.001  
5e2  
1.925e-3

A numeric constant that contains neither a decimal point nor an exponent is initially presumed to be type`integer`if its value fits in type`integer`\(32 bits\); otherwise it is presumed to be type`bigint`if its value fits in type`bigint`\(64 bits\); otherwise it is taken to be type`numeric`. Constants that contain decimal points and/or exponents are always initially presumed to be type`numeric`.

The initially assigned data type of a numeric constant is just a starting point for the type resolution algorithms. In most cases the constant will be automatically coerced to the most appropriate type depending on context. When necessary, you can force a numeric value to be interpreted as a specific data type by casting it.For example, you can force a numeric value to be treated as type`real`\(`float4`\) by writing:

```
REAL '1.23'  -- string style
1.23::REAL   -- PostgreSQL (historical) style
```

These are actually just special cases of the general casting notations discussed next.

#### 4.1.2.7. Constants of Other Types

A constant of an\_arbitrary\_type can be entered using any one of the following notations:

```
type
 '
string
'
'
string
'::
type

CAST ( '
string
' AS 
type
 )
```

The string constant's text is passed to the input conversion routine for the type called`type`. The result is a constant of the indicated type. The explicit type cast can be omitted if there is no ambiguity as to the type the constant must be \(for example, when it is assigned directly to a table column\), in which case it is automatically coerced.

The string constant can be written using either regular SQL notation or dollar-quoting.

It is also possible to specify a type coercion using a function-like syntax:

```
typename
 ( '
string
' )
```

but not all type names can be used in this way; see[Section 4.2.9](https://www.postgresql.org/docs/10/static/sql-expressions.html#sql-syntax-type-casts)for details.

The`::`,`CAST()`, and function-call syntaxes can also be used to specify run-time type conversions of arbitrary expressions, as discussed in[Section 4.2.9](https://www.postgresql.org/docs/10/static/sql-expressions.html#sql-syntax-type-casts). To avoid syntactic ambiguity, the`type`'`string`'syntax can only be used to specify the type of a simple literal constant. Another restriction on the`type`'`string`'syntax is that it does not work for array types; use`::`or`CAST()`to specify the type of an array constant.

The`CAST()`syntax conforms to SQL. The`type`'`string`'syntax is a generalization of the standard: SQL specifies this syntax only for a few data types, butPostgreSQLallows it for all types. The syntax with`::`is historicalPostgreSQLusage, as is the function-call syntax.

### 4.1.3. Operators

An operator name is a sequence of up to`NAMEDATALEN`-1 \(63 by default\) characters from the following list:

* * \* / &lt;&gt; = ~ ! @ \# % ^ & \| \` ?

There are a few restrictions on operator names, however:

* `--`and`/*`cannot appear anywhere in an operator name, since they will be taken as the start of a comment.

* A multiple-character operator name cannot end in`+`or`-`, unless the name also contains at least one of these characters:

  ~ ! @ \# % ^ & \| \` ?

  For example,`@-`is an allowed operator name, but`*-`is not. This restriction allowsPostgreSQLto parse SQL-compliant queries without requiring spaces between tokens.

When working with non-SQL-standard operator names, you will usually need to separate adjacent operators with spaces to avoid ambiguity. For example, if you have defined a left unary operator named`@`, you cannot write`X*@Y`; you must write`X* @Y`to ensure thatPostgreSQLreads it as two operator names not one.

### 4.1.4. Special Characters

Some characters that are not alphanumeric have a special meaning that is different from being an operator. Details on the usage can be found at the location where the respective syntax element is described. This section only exists to advise the existence and summarize the purposes of these characters.

* A dollar sign \(`$`\) followed by digits is used to represent a positional parameter in the body of a function definition or a prepared statement. In other contexts the dollar sign can be part of an identifier or a dollar-quoted string constant.

* Parentheses \(`()`\) have their usual meaning to group expressions and enforce precedence. In some cases parentheses are required as part of the fixed syntax of a particular SQL command.

* Brackets \(`[]`\) are used to select the elements of an array. See[Section 8.15](https://www.postgresql.org/docs/10/static/arrays.html)for more information on arrays.

* Commas \(`,`\) are used in some syntactical constructs to separate the elements of a list.

* The semicolon \(`;`\) terminates an SQL command. It cannot appear anywhere within a command, except within a string constant or quoted identifier.

* The colon \(`:`\) is used to select“slices”from arrays. \(See[Section 8.15](https://www.postgresql.org/docs/10/static/arrays.html).\) In certain SQL dialects \(such as Embedded SQL\), the colon is used to prefix variable names.

* The asterisk \(`*`\) is used in some contexts to denote all the fields of a table row or composite value. It also has a special meaning when used as the argument of an aggregate function, namely that the aggregate does not require any explicit parameter.

* The period \(`.`\) is used in numeric constants, and to separate schema, table, and column names.

### 4.1.5. Comments

A comment is a sequence of characters beginning with double dashes and extending to the end of the line, e.g.:

```
-- This is a standard SQL comment
```

Alternatively, C-style block comments can be used:

```
/* multiline comment
 * with nesting: /* nested block comment */
 */
```

where the comment begins with`/*`and extends to the matching occurrence of`*/`. These block comments nest, as specified in the SQL standard but unlike C, so that one can comment out larger blocks of code that might contain existing block comments.

A comment is removed from the input stream before further syntax analysis and is effectively replaced by whitespace.

### 4.1.6. Operator Precedence

[Table 4.2](https://www.postgresql.org/docs/10/static/sql-syntax-lexical.html#sql-precedence-table)shows the precedence and associativity of the operators inPostgreSQL. Most operators have the same precedence and are left-associative. The precedence and associativity of the operators is hard-wired into the parser.

You will sometimes need to add parentheses when using combinations of binary and unary operators. For instance:

```
SELECT 5 ! - 6;
```

will be parsed as:

```
SELECT 5 ! (- 6);
```

because the parser has no idea — until it is too late — that`!`is defined as a postfix operator, not an infix one. To get the desired behavior in this case, you must write:

```
SELECT (5 !) - 6;
```

This is the price one pays for extensibility.

**Table 4.2. Operator Precedence \(highest to lowest\)**

| Operator/Element | Associativity | Description |
| :--- | :--- | :--- |
| `.` | left | table/column name separator |
| `::` | left | PostgreSQL-style typecast |
| `[]` | left | array element selection |
| `+-` | right | unary plus, unary minus |
| `^` | left | exponentiation |
| `*/%` | left | multiplication, division, modulo |
| `+-` | left | addition, subtraction |
| \(any other operator\) | left | all other native and user-defined operators |
| `BETWEENINLIKEILIKESIMILAR` |  | range containment, set membership, string matching |
| `<>=<=>=<>` |  | comparison operators |
| `ISISNULLNOTNULL` |  | `IS TRUE`,`IS FALSE`,`IS NULL`,`IS DISTINCT FROM`, etc |
| `NOT` | right | logical negation |
| `AND` | left | logical conjunction |
| `OR` | left | logical disjunction |

Note that the operator precedence rules also apply to user-defined operators that have the same names as the built-in operators mentioned above. For example, if you define a“+”operator for some custom data type it will have the same precedence as the built-in“+”operator, no matter what yours does.

When a schema-qualified operator name is used in the`OPERATOR`syntax, as for example in:

```
SELECT 3 OPERATOR(pg_catalog.+) 4;
```

the`OPERATOR`construct is taken to have the default precedence shown in[Table 4.2](https://www.postgresql.org/docs/10/static/sql-syntax-lexical.html#sql-precedence-table)for“any other operator”. This is true no matter which specific operator appears inside`OPERATOR()`.

### Note

PostgreSQLversions before 9.5 used slightly different operator precedence rules. In particular,`<=>=`and`<>`used to be treated as generic operators;`IS`tests used to have higher priority; and`NOT BETWEEN`and related constructs acted inconsistently, being taken in some cases as having the precedence of`NOT`rather than`BETWEEN`. These rules were changed for better compliance with the SQL standard and to reduce confusion from inconsistent treatment of logically equivalent constructs. In most cases, these changes will result in no behavioral change, or perhaps in“no such operator”failures which can be resolved by adding parentheses. However there are corner cases in which a query might change behavior without any parsing error being reported. If you are concerned about whether these changes have silently broken something, you can test your application with the configuration parameter[operator\_precedence\_warning](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html#guc-operator-precedence-warning)turned on to see if any warnings are logged.

---

[^1]: [PostgreSQL: Documentation: 10: 4.1. Lexical Structure](https://www.postgresql.org/docs/10/static/sql-syntax-lexical.html)

