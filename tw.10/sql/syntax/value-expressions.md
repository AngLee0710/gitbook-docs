---
description: 版本：10
---

# 4.2. 參數表示式

參數表示式用在許多不同的方面，像是 SELECT 指令中的回傳列表；在 INSERT 或 UPDATE 指令中指定欄位的新值；又或是在一些命令中，指出搜尋的條件等。參數表示式的結果，有時候會被稱作 scalar，以有別於表格表示式（就是一個表格）的結果。參數表示式也可以稱作 scalar expressions（賦值表示式），甚或簡化為 expressions （表示式）。表示式的語法容許其值為各種運算的單一結果，如數學、邏輯、集合、或其他運算。

參數表示式可以是下列的其中一種形態：

* 常數或文字內容
* 欄位的引用
* 函數參數的引用，在函數裡或預備指令（prepared statement）中
* 子參數表示式
* 欄位選擇表示式
* 運算子宣告
* 函數呼叫
* 彙總表示式
* 窗函數呼叫
* 型別轉換
* 校對轉換（collation expression）
* 賦值子查詢（scalar subquery）
* 陣列建構式
* 列建構式
* 其他被括號括住的參數表示式（用於群組子表示式和強制調整運算優先權）

除了這個列表之外，還有一些建構式也會應用到表示式，但並沒有特別定義語法規則。一般來說，他們會包含函數或運算子的操作，在[第 9 章](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/functions-and-operators.md)中會有適當的說明。其中有一個例子便是 IS NULL 字句。

我們已經在 [4.1.2 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/sql-syntax/41-lexical-structure.md)中討論過常數了，所以接下來就從常數以下的項目繼續說明。

## 4.2.1. 欄位引用

要引要一個欄位的話，請使用下列的形式：

```text
correlation.columnname
```

「correlation」（所屬名稱）是其所屬表格的名稱（也可能需要包含結構名），或是表格的別名（在 FROM 子句中所定義的）。所屬名稱和分隔用的句點是可以省略的，如果欄位名稱在目前查詢中的所有表格中是唯一的話。（[參閱第 7 章](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/queries.md)）

## 4.2.2. 函數參數引用

函數參數的引用，用來指定一個不在該 SQL 指令中的值。參數是使用在 SQL 函數定義或預備查詢之中。有一些用戶端函式庫也支援將資料數值與 SQL 指令分離，在這種情境下，參數就會用來指向外部的資料數值。參數引用的形式如下：

```text
$number
```

舉個例子，有一個函數 dept 的宣告如下：

```text
CREATE FUNCTION dept(text) RETURNS dept
    AS $$ SELECT * FROM dept WHERE name = $1 $$
    LANGUAGE SQL;
```

這裡的 $1 指的是函數被呼叫時的第 1 個輸入參數：

## 4.2.3. 子參數表示式（Subscripts）

如果表示式要產生陣列的結果的話，指定陣列中某個元素，請使用：

```text
expression[subscript]
```

或是要取得陣列中多個相隣的元素，請使用：

```text
expression[lower_subscript:upper_subscript]
```

每一個「subscript」本身都是一個表示式，必須要產生一個整數值。

一般來說，陣列表示式必須被括號起來，但如果該表示式只是一個欄位或參數的引用的話，那麼括號可以省略。然後，多個子參數表示式可以連在一起使用，當你需要陣列表達多維度的概念時。舉例如下：

```text
mytable.arraycolumn[4]
mytable.two_d_column[17][34]
$1[10:42]
(arrayfunction(a,b))[42]
```

在最後一個例子中，括號是必須的。關於陣列，在 [8.15 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/data-types/815-arrays.md)有更多說明。

## 4.2.4. 欄位選擇

如果一個表示式產生了複合性的型別（列型別），那麼要指定其中的某個欄位時，請使用：

```text
expression.fieldname
```

一般來說，列的表示式必須被括號起來，但如果該表示式只是一個欄位或參數的引用的話，那麼括號可以省略。舉例如下：

```text
mytable.mycolumn
$1.somecolumn
(rowfunction(a,b)).col3
```

（然而，有限制的欄位引用，實際上就是一種欄位選擇語法的特列。）有一種重要的特例是從某個複合型別的表格欄位中取其子欄位的值：

```text
(compositecol).somefield
(mytable.compositecol).somefield
```

在這裡，括號是必要的，以表示 compositecol 是一個欄位名稱，但不是表格名稱。而在第二個例子中，mytable 是表格名稱，而非結構名稱。

你可以取得複合資料的所有欄位值，使用「.\*」：

```text
(compositecol).*
```

這個記號在不同的地方有不同的用法，請參閱 [8.16.5 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/data-types/816-composite-types.md)的說明。

## 4.2.5. 運算子宣告（Operator Invocations）

有三種用來進行運算子宣告的語法：

| `expression operator expression`\(雙元中置運算子\) |
| :--- |
| `operator expression`\(單元前置運算子\) |
| `expression operator`\(單元後置運算子\) |

運算子記號的語法規則依 [4.1.3 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/sql-syntax/41-lexical-structure.md)的說明，或是關鍵字 AND、OR、和 NOT，又或是如下形式的限定運算子名稱：

```text
OPERATOR(schema.operatorname)
```

哪些特定的運算子的使用與運算方式，端看系統與使用者如何定義。在[第 9 章](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/functions-and-operators.md)中會說明內建的運算子詳情。

## 4.2.6. 函數呼叫

函數呼叫的語法是，函數的名稱（可能還會加上結構名）接著一連串用括號括起來的參數列表：

```text
function_name ([expression [, expression ... ]] )
```

舉個例子，下面的函數呼叫可以計算 2 的平方根：

```text
sqrt(2)
```

內建函數在[第 9 章](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/functions-and-operators.md)說明，其他的函數可由使用者自訂。

參數可以是選擇性的附加名稱，請參閱 [4.3 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/sql-syntax/43-calling-functions.md)的內容。

## 注意

函數如果只有一個參數，而又是複合型別的話，就稱作使用了欄位選擇語法；反過來說，欄位選擇語法也可以寫成函數的形式。這是因為 col\(table\) 和 table.col 是可以互換的。這並非標準 SQL，但 PostgreSQL 支援了，因為這使得函數的使用可以模擬「計算欄位」（computed fields）。更多資訊請參閱 [8.16.5 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/data-types/816-composite-types.md)。

## 4.2.7. 彙總表示式

彙總表示式用在查詢時，過濾資料進行彙總函數計算的應用。彙總函數壓縮了大量資料輸入成為一個單一的輸出值，例如加總或平均數。彙總表示式的語法可以是下列其中之一：

```text
aggregate_name (expression [ , ... ] [ order_by_clause ] ) [ FILTER ( WHERE filter_clause ) ]

aggregate_name (ALL expression [ , ... ] [ order_by_clause ] ) [ FILTER ( WHERE filter_clause ) ]

aggregate_name (DISTINCT expression [ , ... ] [ order_by_clause ] ) [ FILTER ( WHERE filter_clause ) ]

aggregate_name ( * ) [ FILTER ( WHERE filter_clause ) ]

aggregate_name ( [ expression [ , ... ] ] ) WITHIN GROUP ( order_by_clause ) [ FILTER ( WHERE filter_clause ) ]
```

這裡的 agregate\_name 是預先就定義好的（可能還需要加上結構名稱），表示式可以是任何的函數形態，但不能包含彙總函數或窗函數。而 order\_by\_clause 和 filter\_clause 後續進行說明。

第一種形式的彙總表示式用於每次輸入一列的情況；第二種形式和第一種相同，當 ALL 是預設的時候；第三種形式彙總不重覆的資料（或在多種表示式的時候，取不重覆的集合）；第四種形式也是每次輸入一列，但沒有限定輸入條件，通常是用於 count\(\*\)；最後一種形式用於有次序的彙總函數，稍後說明。

大多數的彙總函數會忽略空值，所以如果表示式計算的結果是空值的話，就會忽略不計。這樣的假設除非有特別設定，對所有內建的函數都是如此。

舉例來說，count\(\*\) 計算輸入列的個數，而 count\(f1\) 是計算輸入列中 f1 欄位非空值的個數，因為 count 會忽略空值；然而，count\(distinct f1\) 則是計算 f1 欄位不重覆又非空值的個數。

通常彙總函數在處理輸入資料時，都是未排序過的。在大多數的情況沒有關係，例如：min 最小值的計算，與其輸入的次序沒有關係。然而，還是有些彙總函數的結果，與其處理次序是有關連的，例如：array\_agg 和 string\_agg。ORDER BY 字句就可以達到此效果，其與一般查詢語法 ORDER BY 的用法相同，詳細說明在 7.5 節，除非該表示式無法輸出成欄位名稱或數字。舉例如下：

```text
SELECT array_agg(a ORDER BY b DESC) FROM table;
```

操作到多參數的彙總函數時，注意 ORDER BY 會處理過所有的彙總參數，例如：

```text
SELECT string_agg(a, ',' ORDER BY a) FROM table;
```

但不能這樣寫：

```text
SELECT string_agg(a ORDER BY a, ',') FROM table;  -- incorrect
```

這在語法上沒有不合法，但這表示一個單參數的彙總函數，使用了兩個排序的關鍵值（第二個完全沒用，因為它是常數）。

如果 DISTINCT 被加到 ORDER BY 子句裡的話，那麼所有的 ORDER BY 表示式都必須符合彙總函數的參數，也就是說，你不能使用不在 DISTINCT 列表中的表示式來排序。

## 注意

在彙總函數中使用 DISTINCT 和 ORDER BY，都是 PostgreSQL 的延伸。

把 ORDER BY 放進彙總函數的參數列表中，就如同到目前為止的描述，用於排序輸入值，進行一般性的處理或統計彙總，而排序是選擇性的。有另一種類型的彙總函數稱作有次序彙總，它們就必須要有 ORDER BY 子句，通常就是因為這些函數的計算結果，只會對某些特定次序的資料產生效果。典型的有次序彙總例子，包含排名和累計百分比計算。對於有次序彙總計算，將 ORDER BY 字句寫進 WITHIN GROUP \(...\) 中，如同上述最後一個語法例子。在 ORDER BY 子句中的表示式會處理每一筆輸入資料，如同一般的彚總函數，然後將其依子句中的表示式計算並排序，最後再依序轉送給彙總函數處理。（這和非處理 WITHIN GROUP 中的 ORDER BY 不同，它們不會再轉送給彙總函數。）如果有在 WITHIN GROUP 之前的表示式的話，稱作直接參數，會和有 ORDER BY 的參數有區分。不像一般的彙總參數，直接參數只會被處理一次，而不是每一筆都一次。這意思是只有在 GROUP BY 中，這些變數才會被彙總處理。這樣的限制就如同直接參數不在彙總表示式之中一樣。直接參數一般用於累計分配，只有在每一次彙整完的值才有意義。直接參數可以是空值，在這個例子中，使用的是 \(\)，而非 \(\*\)。（PostgreSQL 兩種寫法都可以接受，但標準 SQL 只接受前者。）

有次序彙總查詢如下：

```text
SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY income) FROM households;

 percentile_cont
-----------------
           50489
```

這裡包含了 50% 的累計，或是中間數累計，來源是表格 households 的 income 欄位。其中，0.5 是直接參數，它不影響百分累計彙整計算過程。

如果使用了 FILTER，那就只有符合 FILTER 子句條件的資料會被彙總處理，其他的資料都會被忽略掉。舉例來說：

```text
SELECT
    count(*) AS unfiltered,
    count(*) FILTER (WHERE i < 5) AS filtered
FROM generate_series(1,10) AS s(i);

 unfiltered | filtered
------------+----------
         10 |        4
(1 row)
```

預先內建的彙總函數將在 [9.20 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/functions-and-operators/920-aggregate-functions.md)中介紹，其他彙總函數可以由使用者自行設計。

彙總表示式只可以用於結果列表或 SELECT 中的 HAVING 子句。在其他子句中是被禁止的，像是 WHERE，因為這些子句邏輯上都是在彙總處理前就得處理資料。

當彙總表示式使用在子查詢（參閱 [4.2.11 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/sql-syntax/42-value-expressions.md)及 [9.22 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/functions-and-operators/922-subquery-expressions.md)）中時，彙總計算就會一般性地處理子查詢中的資料。但如果該彙總計算的參數用到了外層的變數時，就會產生例外情況：彙整計算是屬於最接近的外層查詢，並且只處理該層的查詢資料。這個彙總表示式對整體而言，只是一個子查詢的引用，它會被視為一個常數的結果，限制它只會出現在 HAVING 子句的運算層次而已。

## 4.2.8. 窗函數呼叫

窗函數呼叫指的是使用類似彙總函數的使用方式，只是僅用於查詢中部份列的選擇上。和非窗函數不同的是，這並不會只輸出為單一列—每一列都仍然分開輸出。然而，窗函數也是處理了所有該列所屬群組的其他列（PARTITION BY），依其窗函數所定義的範圍。窗函數呼叫的方式可以是下列其中之一：

```text
function_name ([expression [, expression ... ]]) [ FILTER ( WHERE filter_clause ) ] OVER window_name
function_name ([expression [, expression ... ]]) [ FILTER ( WHERE filter_clause ) ] OVER ( indow_definition )
function_name ( * ) [ FILTER ( WHERE filter_clause ) ] OVER window_name
function_name ( * ) [ FILTER ( WHERE filter_clause ) ] OVER ( indow_definition )
```

定義「窗」，請使用下列語法：

```text
[ existing_window_name ][ PARTITION BY expression [, ...] ]
[ ORDER BY expression [ ASC | DESC | USING operator ] [ NULLS { FIRST | LAST } ] [, ...] ]
[ frame_clause ]
```

選擇性的 frame\_clause 語法如下：

```text
{ RANGE | ROWS } frame_start
{ RANGE | ROWS } BETWEEN frame_start AND frame_end
```

frame\_start 及 frame\_end 的語法如下：

```text
UNBOUNDED PRECEDING
value PRECEDING 
CURRENT ROW
value FOLLOWING 
UNBOUNDED FOLLOWING
```

在這裡的表示式（expression），除了不能再包含窗函數之外，無其他特別限制。

window\_name 是一個定義在 WINDOW 子句中的命名。另一方面，一個完整的窗也可以是被括號括起來，使用和 WINDOW 子句相同語法的定義。詳見 [SELECT 語法](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands/select.md)頁面。值得探討的是，OVER wname 並不完全等同於 OVER \(wname ...\)；後者隱含著複製及修改窗的定義，而如果包含 frame 子句的話，就會被拒絕執行。

PARTITION BY 子句將查詢分組成為不同的分區，它們將會分別地被窗函數所處理。PARTITION BY 的行為和查詢語句中的 GROUP BY 很類似，除了它的表示式就只是表示式，而且不能產出欄位名稱或編號。沒有 PARTITION BY 的話，所有的列都會被當作一個分組進行彙總。ORDER BY 子句決定窗函數的處理次序，它也和查詢語句中的 ORDER BY 很類似，但它不能使用輸出的欄位或編號。如果沒有 ORDER BY 的話，就無法保證彙總處理的次序了。

frame\_clause 指的是構成該窗的列，再進一步以「窗框」拆分，是目前分區的子集合。對窗函數而言，運算會以窗框的範圍取代整合分區。窗框的指定可以是 RANGE 或 ROW 兩種模式。不論哪種模式，都 frame\_start 執行到 frame\_end，但如果 frame\_end 省略了，預設就是到目前的列（CURRENT ROW）。

UNBOUNDED PRECEDING 的窗框始於該分區的第一列，同樣地，UNBOUNDED FOLLOWING 意指窗框結束於分區的最後一列。

在 RANGE 模式裡，如果 frame\_start 設定為 CURRENT ROW 的話，表示窗框始於目前列同序的那一列（使用 ORDER BY 時，排序相同的那一列），同理，frame\_end 設定為 CURRENT ROW 時，表示窗框止於排序相同的列。而在 ROWS 模式時，CURRENT ROW 指的就是自己。

PRECEDING 和 FOLLOWING 兩個設定值，目前只能用在 ROWS 模式。它們指的是窗框的起迄於指定的一個值，表示目前列之前後多少列。而所謂的值，必須是整數表示式而不包含任何變數、彙總函數、或窗函數。其值也不能是空值或負值，但可以為零，表示只處理目前列。

預設的窗框設定是 RANGE UNBOUNDED PRECEDING，和 RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW 是一樣的。加上 ORDER BY 的話，這可以讓窗框起於和目前列並列的列；沒有 ORDER BY 的話，所有的列都會在分區裡，因為如此就無法判定次序，表示大家都一樣。

frame\_start 的限制是不能使用 UNBOUNDED FOLLOWING，而 frame\_end 不能使用 UNBOUNDED PRECEDING。frame\_end 的設定也不能先於 frame\_start—舉例來說，RANGE BETWEEN CURRENT ROW，使用 PRECEDING 就不可以。

如果有使用到 FILTER 的話，就只有符合 FILTER 條件式的列會被窗函數處理，其餘的列都會被忽略。只有彙總式的窗函數可以使用 FILTER 子句。

內建的窗函數會在 [9.57 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/functions-and-operators/95-binary-string-functions-and-operators.md)中說明，使用者也可以自行設計窗函數。任何內建或自訂的一般函數或統計函數，都可以當作窗函數來使用。（有序集合和假定集合的彙總數，目前不能當作窗函數來使用。）

「\*」語法的使用，用來把無參數的彙總函數當作窗函數來使用，例如：count\(\*\) OVER \(PARTITION BY x ORDER BY y\)。「\*」通常不會用於專門的窗函數上，專門的窗函數不允許參數裡有用到 DISTINCT 或 ORDER BY 的語法。

窗函數呼叫只限於 SELECT 回傳列表，及 ORDER BY 子句中。

更多窗函數的說明請參閱 [3.5 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/advanced-features/35-window-functions.md)、[9.21 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/functions-and-operators/921-window-functions.md)、及 [7.2.5 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/queries/72-table-expressions.md)。

## 4.2.9. 型別轉換

型別轉換指定從一種資料型別轉換為另一種資料型別。PostgreSQL 接受兩種用於型別轉換的等效語法：

```text
CAST ( expression AS type )
expression::type
```

CAST 語法符合 SQL 標準；帶「::」的語法是 PostgreSQL 既有的用法。

當強制轉換應用於已知型別的值表示式時，它表示執行時型別轉換。只有定義了合適的型別轉換操作，操作才能成功。請注意，這與使用帶常數的強制轉換略有不同，如 [4.1.2.7 節](lexical.md#4-1-2-7-qi-ta-xing-chang)所示。應用於未經修飾的字串文字的強制轉換表示將型別初始分配給文字常數，因此對於任何型別（如果字串文字的內容都是資料型別的可接受輸入語法）都會成功。

如果對於值表示式必須產生的型別沒有歧義（例如，當它被分配給資料表欄位），通常可以省略顯式的型別轉換；系統將在這種情況下自動套用型別轉換。但是，只有在系統目錄中標記為「可以隱式套用」的強制轉換才會執行自動強制轉換。其他強制轉換必須使用顯式強制轉換語法來使用。此限制旨在防止系統默默地套用令人意外的轉換。

也可以使用函數式語法來指定型別轉換：

```text
typename ( expression )
```

但是，這僅適用於名稱也可以作為函數名稱使用的型別。例如，雙精度不能用這種方式，但等價的 float8 可以。而且，由於語法衝突，名稱間隔，時間和時間戳記只能使用雙引號才能用於這種方式。因此，使用類似功能的轉換語法會導致不一致，因此可能應該避免。

#### 注意

函數式語法實際上只是一個函數呼叫。當兩個標準轉換語法之一用於執行轉換時，它將在內部呼叫已註冊的函數來執行轉換。按照慣例，這些轉換函數與它們的輸出類型具有相同的名稱，因此「函數式語法」只不過是直接呼叫底層的轉換函數。顯然，這不是一個可移植式應用程序應該依賴的東西。有關更多詳情，請參閱 [CREATE CAST](../../reference/sql-commands/create-cast.md)。

## 4.2.10. Collation Expressions

The`COLLATE`clause overrides the collation of an expression. It is appended to the expression it applies to:

```text
expr
 COLLATE 
collation
```

where\_`collation`\_is a possibly schema-qualified identifier. The`COLLATE`clause binds tighter than operators; parentheses can be used when necessary.

If no collation is explicitly specified, the database system either derives a collation from the columns involved in the expression, or it defaults to the default collation of the database if no column is involved in the expression.

The two common uses of the`COLLATE`clause are overriding the sort order in an`ORDER BY`clause, for example:

```text
SELECT a, b, c FROM tbl WHERE ... ORDER BY a COLLATE "C";
```

and overriding the collation of a function or operator call that has locale-sensitive results, for example:

```text
SELECT * FROM tbl WHERE a 
>
 'foo' COLLATE "C";
```

Note that in the latter case the`COLLATE`clause is attached to an input argument of the operator we wish to affect. It doesn't matter which argument of the operator or function call the`COLLATE`clause is attached to, because the collation that is applied by the operator or function is derived by considering all arguments, and an explicit`COLLATE`clause will override the collations of all other arguments. \(Attaching non-matching`COLLATE`clauses to more than one argument, however, is an error. For more details see[Section 23.2](https://www.postgresql.org/docs/10/static/collation.html).\) Thus, this gives the same result as the previous example:

```text
SELECT * FROM tbl WHERE a COLLATE "C" 
>
 'foo';
```

But this is an error:

```text
SELECT * FROM tbl WHERE (a 
>
 'foo') COLLATE "C";
```

because it attempts to apply a collation to the result of the`>`operator, which is of the non-collatable data type`boolean`.

## 4.2.11. Scalar 子查詢

Scalar 子查詢指的是括號中的普通 SELECT 查詢，但它只回傳一個資料列的一個欄位。（有關撰寫查詢的訊息，請參閱[第 7 章](../7.-zi-liao-cha-xun/)。）執行 SELECT 查詢並在周圍的值表示式中使用單個回傳的值。使用回傳多於一個資料列或多於一個欄位的查詢作為 scalar 子查詢是錯誤的。（但是，如果在特定執行過程中子查詢不回傳任何資料列，則不會出現錯誤；Scalar 結果將視為空）。子查詢可以引用周圍查詢中的變數，該變數在任何一次運算期間都將用作常數的子查詢。有關子查詢的其他表示式，另請參閱[第 9.22 節](../functions/9.22.-zi-cha-xun.md)。

例如，以下是每個州中最大的城市人口數量：

```text
SELECT name, (SELECT max(pop) FROM cities WHERE cities.state = states.name)
    FROM states;
```

## 4.2.12. Array Constructors

An array constructor is an expression that builds an array value using values for its member elements. A simple array constructor consists of the key word`ARRAY`, a left square bracket`[`, a list of expressions \(separated by commas\) for the array element values, and finally a right square bracket`]`. For example:

```text
SELECT ARRAY[1,2,3+4];
  array
---------
 {1,2,7}
(1 row)
```

By default, the array element type is the common type of the member expressions, determined using the same rules as for`UNION`or`CASE`constructs \(see[Section 10.5](https://www.postgresql.org/docs/10/static/typeconv-union-case.html)\). You can override this by explicitly casting the array constructor to the desired type, for example:

```text
SELECT ARRAY[1,2,22.7]::integer[];
  array
----------
 {1,2,23}
(1 row)
```

This has the same effect as casting each expression to the array element type individually. For more on casting, see[Section 4.2.9](https://www.postgresql.org/docs/10/static/sql-expressions.html#sql-syntax-type-casts).

Multidimensional array values can be built by nesting array constructors. In the inner constructors, the key word`ARRAY`can be omitted. For example, these produce the same result:

```text
SELECT ARRAY[ARRAY[1,2], ARRAY[3,4]];
     array
---------------
 &#123;{1,2},{3,4}&#125;
(1 row)

SELECT ARRAY[[1,2],[3,4]];
     array
---------------
 &#123;{1,2},{3,4}&#125;
(1 row)
```

Since multidimensional arrays must be rectangular, inner constructors at the same level must produce sub-arrays of identical dimensions. Any cast applied to the outer`ARRAY`constructor propagates automatically to all the inner constructors.

Multidimensional array constructor elements can be anything yielding an array of the proper kind, not only a sub-`ARRAY`construct. For example:

```text
CREATE TABLE arr(f1 int[], f2 int[]);

INSERT INTO arr VALUES (ARRAY[[1,2],[3,4]], ARRAY[[5,6],[7,8]]);

SELECT ARRAY[f1, f2, '&#123;{9,10},{11,12}&#125;'::int[]] FROM arr;
                     array
------------------------------------------------
 {&#123;{1,2},{3,4}},&#123;{5,6},{7,8}},&#123;{9,10},{11,12}&#125;}
(1 row)
```

You can construct an empty array, but since it's impossible to have an array with no type, you must explicitly cast your empty array to the desired type. For example:

```text
SELECT ARRAY[]::integer[];
 array
-------
 {}
(1 row)
```

It is also possible to construct an array from the results of a subquery. In this form, the array constructor is written with the key word`ARRAY`followed by a parenthesized \(not bracketed\) subquery. For example:

```text
SELECT ARRAY(SELECT oid FROM pg_proc WHERE proname LIKE 'bytea%');
                                 array
-----------------------------------------------------------------------
 {2011,1954,1948,1952,1951,1244,1950,2005,1949,1953,2006,31,2412,2413}
(1 row)

SELECT ARRAY(SELECT ARRAY[i, i*2] FROM generate_series(1,5) AS a(i));
              array
----------------------------------
 &#123;{1,2},{2,4},{3,6},{4,8},{5,10}&#125;
(1 row)
```

The subquery must return a single column. If the subquery's output column is of a non-array type, the resulting one-dimensional array will have an element for each row in the subquery result, with an element type matching that of the subquery's output column. If the subquery's output column is of an array type, the result will be an array of the same type but one higher dimension; in this case all the subquery rows must yield arrays of identical dimensionality, else the result would not be rectangular.

The subscripts of an array value built with`ARRAY`always begin with one. For more information about arrays, see[Section 8.15](https://www.postgresql.org/docs/10/static/arrays.html).

## 4.2.13. Row Constructors

A row constructor is an expression that builds a row value \(also called a composite value\) using values for its member fields. A row constructor consists of the key word`ROW`, a left parenthesis, zero or more expressions \(separated by commas\) for the row field values, and finally a right parenthesis. For example:

```text
SELECT ROW(1,2.5,'this is a test');
```

The key word`ROW`is optional when there is more than one expression in the list.

A row constructor can include the syntax`rowvalue.*`, which will be expanded to a list of the elements of the row value, just as occurs when the`.*`syntax is used at the top level of a`SELECT`list \(see[Section 8.16.5](https://www.postgresql.org/docs/10/static/rowtypes.html#rowtypes-usage)\). For example, if table`t`has columns`f1`and`f2`, these are the same:

```text
SELECT ROW(t.*, 42) FROM t;
SELECT ROW(t.f1, t.f2, 42) FROM t;
```

## Note

BeforePostgreSQL8.2, the`.*`syntax was not expanded in row constructors, so that writing`ROW(t.*, 42)`created a two-field row whose first field was another row value. The new behavior is usually more useful. If you need the old behavior of nested row values, write the inner row value without`.*`, for instance`ROW(t, 42)`.

By default, the value created by a`ROW`expression is of an anonymous record type. If necessary, it can be cast to a named composite type — either the row type of a table, or a composite type created with`CREATE TYPE AS`. An explicit cast might be needed to avoid ambiguity. For example:

```text
CREATE TABLE mytable(f1 int, f2 float, f3 text);

CREATE FUNCTION getf1(mytable) RETURNS int AS 'SELECT $1.f1' LANGUAGE SQL;

-- No cast needed since only one getf1() exists
SELECT getf1(ROW(1,2.5,'this is a test'));
 getf1
-------
     1
(1 row)

CREATE TYPE myrowtype AS (f1 int, f2 text, f3 numeric);

CREATE FUNCTION getf1(myrowtype) RETURNS int AS 'SELECT $1.f1' LANGUAGE SQL;

-- Now we need a cast to indicate which function to call:
SELECT getf1(ROW(1,2.5,'this is a test'));
ERROR:  function getf1(record) is not unique

SELECT getf1(ROW(1,2.5,'this is a test')::mytable);
 getf1
-------
     1
(1 row)

SELECT getf1(CAST(ROW(11,'this is a test',2.5) AS myrowtype));
 getf1
-------
    11
(1 row)
```

Row constructors can be used to build composite values to be stored in a composite-type table column, or to be passed to a function that accepts a composite parameter. Also, it is possible to compare two row values or test a row with`IS NULL`or`IS NOT NULL`, for example:

```text
SELECT ROW(1,2.5,'this is a test') = ROW(1, 3, 'not the same');

SELECT ROW(table.*) IS NULL FROM table;  -- detect all-null rows
```

For more detail see[Section 9.23](https://www.postgresql.org/docs/10/static/functions-comparisons.html). Row constructors can also be used in connection with subqueries, as discussed in[Section 9.22](https://www.postgresql.org/docs/10/static/functions-subquery.html).

## 4.2.14. 表示式運算規則

並沒有定義子表示式的運算順序。特別是，運算子或函數的輸入不一定是從左到右或以任何其他固定順序進行運算。

進一步來說，如果一個表示式的結果可以透過只運算它的某些部分來得到，那麼其他子表示式可能根本就不會被運算。 例如，如果有人寫了：

```text
SELECT true OR somefunc();
```

那麼 somefunc\(\) 將（可能）根本不會被呼叫。如果有人寫了：

```text
SELECT somefunc() OR true;
```

請注意，這與在某些程語言中發現的布林運算是從左到右的「短路」不同。

因此，將具有副作用的函數用作複雜表示式的一部分是不明智的。在 WHERE 和 HAVING 子句中依賴副作用或運算順序是特別危險的，因為這些子句作為製定執行計劃的一部分經常式會被重新運算。這些子句中的布林表示式（AND / OR / NOT 組合）可以按照布林代數法則的任何方式重新組織。

如果必須強制執行某部份的運算指令，則可以使用 CASE 結構（請參閱[第 9.17 節](../functions/functions-conditional.md)）。例如，這是試圖避免在 WHERE 子句中除以零不可信任的方式：

```text
SELECT ... WHERE x 
>
 0 AND y/x 
>
 1.5;
```

但這樣是安全的：

```text
SELECT ... WHERE CASE WHEN x 
>
 0 THEN y/x 
>
 1.5 ELSE false END;
```

以這種方式使用的 CASE 構造將放棄最佳化嘗試，因此只能在必要時進行。（在這個特定的例子中，透過改寫為 y&gt; 1.5 \* x 來避免這個問題會更好。）

然而，CASE 對於這些問題並不是萬能的。上述技術的一個局限是它不能阻止對常數子表示式的預先評估。如[第 37.6 節](../../server-programming/extending-sql/function-volatility-categories.md)所述，標記為 IMMUTABLE 的函數和運算子可以在查詢計劃時進行運算，而不是在執行時進行運算。因此，例如：

```text
SELECT CASE WHEN x 
>
 0 THEN x ELSE 1/0 END FROM tab;
```

由於查詢規劃試圖簡化常數子表示式，因此即使資料表中的每一個資料列都具有 x&gt; 0，以至於在執行時永遠不會走到 ELSE，也可能導致除以零的例外情況。

雖然這個特殊的例子看起來很愚蠢，但是在函數中執行的查詢中可能會出現不明顯涉及常數的情況，因為函數參數和局部變數的值可以作為常數插入到查詢中以用於查詢規劃。例如，在 PL/pgSQL 函數中，使用 IF-THEN-ELSE 語句來保護有風險的運算要比將它嵌套在 CASE 表示式中要安全得多。

同一種類型的另一個限制是，CASE 無法阻止運算其中包含的彙總表示式，因為需要在 SELECT 資料列表或 HAVING 子句中的其他表示式之前計算彙總表示式。例如，下面的查詢可能會導致一個除以零例外情況，儘管似乎已經受到保護：

```text
SELECT CASE WHEN min(employees) > 0
            THEN avg(expenses / employees)
       END
    FROM departments;
```

min\(\) 和 avg\(\) 彙總運算是在所有輸入的資料列上同時計算的，因此如果任何員工的資料等於零，則在有任何測試 min\(\) 結果的機會之前，發生除以零的錯誤。相反，使用 WHERE 或 FILTER 子句來防止有問題的輸入資料列，將可以在彙總函數之前來預防這種情況發生。

