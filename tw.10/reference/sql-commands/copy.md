---
description: 版本：10
---

# COPY

COPY — 在檔案和資料表之間複製資料

### 語法

```text
COPY table_name [ ( column_name [, ...] ) ]
    FROM { 'filename' | PROGRAM 'command' | STDIN }
    [ [ WITH ] ( option [, ...] ) ]

COPY { table_name [ ( column_name [, ...] ) ] | ( query ) }
    TO { 'filename' | PROGRAM 'command' | STDOUT }
    [ [ WITH ] ( option [, ...] ) ]

where option can be one of:

    FORMAT format_name
    OIDS [ boolean ]
    FREEZE [ boolean ]
    DELIMITER 'delimiter_character'
    NULL 'null_string'
    HEADER [ boolean ]
    QUOTE 'quote_character'
    ESCAPE 'escape_character'
    FORCE_QUOTE { ( column_name [, ...] ) | * }
    FORCE_NOT_NULL ( column_name [, ...] )
    FORCE_NULL ( column_name [, ...] )
    ENCODING 'encoding_name'
```

### 說明

COPY 在 PostgreSQL 資料表和標準檔案系統的檔案之間移動資料。COPY TO 將資料表的內容複製到檔案，而 COPY FROM 將資料從檔案複製到資料表（將資料附加到資料表中）。COPY TO 還可以複製 SELECT 查詢的結果。

如果指定了欄位列表，則 COPY 將僅將指定欄位中的資料複製到檔案或從檔案複製。如果資料表中有任何欄位不在欄位列表中，則 COPY FROM 將插入這些欄位的預設值。

帶有檔案名稱的 COPY 指示 PostgreSQL 伺服器直接讀取或寫入檔案。PostgreSQL 使用者必須可以存取該檔案（伺服器執行的作業系統使用者 ID），並且必須從伺服器的角度指定名稱。使用 PROGRAM 時，伺服器執行給定的命令並從程序的標準輸出讀取，或寫入程序的標準輸入。必須從伺服器的角度使用該命令，並且該命令可由 PostgreSQL 作業系統使用者執行。指定 STDIN 或 STDOUT 時，資料透過用戶端和伺服器之間的連線傳輸。

### 參數

_`table_name`_

現有資料表的名稱（可選擇性加上綱要）。

_`column_name`_

要複製欄位的選擇性列表。如果未指定欄位列表，則將複製資料表的所有欄位。

_`query`_

[SELECT](select.md)，[VALUES](values.md)，[INSERT](insert.md)，[UPDATE](update.md) 或 [DELETE](delete.md) 指令，其結果將被複製。請注意，查詢周圍需要括號。

對於 INSERT，UPDATE 和 DELETE 查詢，必須提供 RETURNING 子句，並且目標關連不能具有條件規則，也不能具有 ALSO 規則，也不能具有延伸為多個語句的 INSTEAD 規則。

_`filename`_

輸入或輸出檔案的路徑名稱。輸入檔案名稱可以是絕對路徑或相對路徑，但輸出檔案名稱必須是絕對路徑。Windows 使用者可能需要使用 E''字串並將路徑名稱中所使用的任何倒斜線加倍。

`PROGRAM`

要執行的命令。在 COPY FROM 中，從命令的標準輸出讀取輸入；在 COPY TO 中，輸出給寫入命令的標準輸入。

請注意，該命令由 shell 呼叫，因此如果您需要將任何參數傳遞給來自不受信任來源的 shell 命令，則必須小心去除或轉義可能對 shell 具有特殊含義的任何特殊字串。出於安全原因，最好使用固定的命令字串，或者至少避免在其中傳遞任何使用者輸入參數。

`STDIN`

指定輸入來自用戶端應用程式。

`STDOUT`

指定輸出轉到用戶端應用程序式。

_`boolean`_

指定是應打開還是關閉所選選項。您可以寫入TRUE，ON 或 1 以啟用該選項，使用 FALSE，OFF 或 0 來停用它。布林值也可以省略，在這種情況下假定為 TRUE

`FORMAT`

選擇要讀取或寫入的資料格式：text，csv（逗號分隔值）或二進位。預設為 text。

`OIDS`

指定複製每個資料列的 OID。 （如果為沒有 OID 的資料表指定 OIDS，或者在複製查詢的情況下，會引發錯誤。）

`FREEZE`

請求複製已經凍結的資料列，就像在執行 VACUUM FREEZE 命令之後一樣。這是初始資料載入的效能選項。只有在目前子事務中建立或清空正在載入的資料表時，才會凍結資料列，而沒有使用中游標，並且此事務不保留舊的快照。

請注意，所有其他連線在成功載入後將立即能夠看到資料。這違反了 MVCC 可見性的正常規則，使用者應該知道這可能的潛在問題。

`DELIMITER`

指定用於分隔檔案每行內欄位的字元。預設值為 text 格式的 tab 字元，CSV 格式的逗號。這必須是一個單位元組字元。採用二進位格式時不允許使用此選項。

`NULL`

指定表示空值的字串。 預設值為 text 格式的 N（倒斜線-N）和 CSV 格式的未加引號的空字串。對於不希望將空值與空字串區分開的情況，即使是 text 格式，也可能更喜歡空字串。採用二進位格式時不允許使用此選項。

**注意**  
使用 COPY FROM 時，與該字串匹配的任何資料項都將儲存為空值，因此您應確保使用與 COPY TO 相同的字串。

`HEADER`

指定該檔案包含標題列，其中包含檔案中每個欄位的名稱。在輸出時，第一行包含資料表中的欄位名稱；在輸入時，第一行將被忽略。僅在採用 CSV 格式時才允許此選項。

`QUOTE`

指定引用資料值時要使用的引用字元。預設為雙引號。這必須是一個單位元組字元。僅在採用 CSV 格式時才允許此選項。

`ESCAPE`

指定應在與 QUOTE 值匹配的資料字元之前出現的字元。預設值與 QUOTE 值相同（因此，如果引號字元出現在資料中，則引號字元加倍）。這必須是一個單位元組字元。僅在使用 CSV 格式時才允許此選項。

`FORCE_QUOTE`

強制引用用於每個指定欄位中的所有非 NULL 值。從不引用 NULL 輸出。如果指定 \*，則將在所有欄位中引用非 NULL 值。此選項僅在 COPY TO 中允許，並且僅在使用 CSV 格式時允許。

`FORCE_NOT_NULL`

不要將指定欄位的值與空字串匹配。在 null 字串為空的預設情況下，這意味著空值將被讀取為零長度字串而不是空值，即使它們未被引用也是如此。此選項僅在 COPY FROM 中允許，並且僅能用在 CSV 格式時。

`FORCE_NULL`

將指定欄位的值與空字串匹配，即使它已被引用，如果找到匹配項，則將值設定為 NULL。在 null 字串為空的預設情況下，這會將帶引號的空字串轉換為 NULL。此選項僅在 COPY FROM 中允許，並且僅能用在 CSV 格式。

`ENCODING`

指定文件在 encoding\_name 中編碼。如果省略此選項，則使用目前用戶端編碼。有關詳細訊息，請參閱下面的註釋。

### 輸出

成功完成後，COPY 命令將回傳命令標記的形式

```text
COPY count
```

計數是複製的資料列數量。

**注意**  
僅當命令不是 COPY ... TO STDOUT 或等效的 psql 元命令 \copy ... to stdout 時，psql 才會輸出此命令標記。這是為了防止命令標記與剛剛輸出的資料混淆。

### Notes

`COPY TO` can only be used with plain tables, not with views. However, you can write `COPY (SELECT * FROM` _`viewname`_\) TO ... to copy the current contents of a view.

`COPY FROM` can be used with plain tables and with views that have `INSTEAD OF INSERT` triggers.

`COPY` only deals with the specific table named; it does not copy data to or from child tables. Thus for example `COPY` _`table`_ TO shows the same data as `SELECT * FROM ONLY` _`table`_. But `COPY (SELECT * FROM` _`table`_\) TO ... can be used to dump all of the data in an inheritance hierarchy.

You must have select privilege on the table whose values are read by `COPY TO`, and insert privilege on the table into which values are inserted by `COPY FROM`. It is sufficient to have column privileges on the column\(s\) listed in the command.

If row-level security is enabled for the table, the relevant `SELECT` policies will apply to `COPY` _`table`_ TO statements. Currently, `COPY FROM` is not supported for tables with row-level security. Use equivalent `INSERT` statements instead.

Files named in a `COPY` command are read or written directly by the server, not by the client application. Therefore, they must reside on or be accessible to the database server machine, not the client. They must be accessible to and readable or writable by the PostgreSQL user \(the user ID the server runs as\), not the client. Similarly, the command specified with `PROGRAM` is executed directly by the server, not by the client application, must be executable by the PostgreSQL user. `COPY` naming a file or command is only allowed to database superusers, since it allows reading or writing any file that the server has privileges to access.

Do not confuse `COPY` with the psql instruction [`\copy`](https://www.postgresql.org/docs/10/static/app-psql.html#APP-PSQL-META-COMMANDS-COPY). `\copy` invokes `COPY FROM STDIN` or `COPY TO STDOUT`, and then fetches/stores the data in a file accessible to the psql client. Thus, file accessibility and access rights depend on the client rather than the server when `\copy` is used.

It is recommended that the file name used in `COPY` always be specified as an absolute path. This is enforced by the server in the case of `COPY TO`, but for `COPY FROM` you do have the option of reading from a file specified by a relative path. The path will be interpreted relative to the working directory of the server process \(normally the cluster's data directory\), not the client's working directory.

Executing a command with `PROGRAM` might be restricted by the operating system's access control mechanisms, such as SELinux.

`COPY FROM` will invoke any triggers and check constraints on the destination table. However, it will not invoke rules.

For identity columns, the `COPY FROM` command will always write the column values provided in the input data, like the `INSERT` option `OVERRIDING SYSTEM VALUE`.

`COPY` input and output is affected by `DateStyle`. To ensure portability to other PostgreSQL installations that might use non-default `DateStyle` settings, `DateStyle` should be set to `ISO` before using `COPY TO`. It is also a good idea to avoid dumping data with `IntervalStyle` set to`sql_standard`, because negative interval values might be misinterpreted by a server that has a different setting for `IntervalStyle`.

Input data is interpreted according to `ENCODING` option or the current client encoding, and output data is encoded in `ENCODING` or the current client encoding, even if the data does not pass through the client but is read from or written to a file directly by the server.

`COPY` stops operation at the first error. This should not lead to problems in the event of a `COPY TO`, but the target table will already have received earlier rows in a `COPY FROM`. These rows will not be visible or accessible, but they still occupy disk space. This might amount to a considerable amount of wasted disk space if the failure happened well into a large copy operation. You might wish to invoke `VACUUM` to recover the wasted space.

`FORCE_NULL` and `FORCE_NOT_NULL` can be used simultaneously on the same column. This results in converting quoted null strings to null values and unquoted null strings to empty strings.

### File Formats

#### Text Format

When the `text` format is used, the data read or written is a text file with one line per table row. Columns in a row are separated by the delimiter character. The column values themselves are strings generated by the output function, or acceptable to the input function, of each attribute's data type. The specified null string is used in place of columns that are null. `COPY FROM` will raise an error if any line of the input file contains more or fewer columns than are expected. If `OIDS` is specified, the OID is read or written as the first column, preceding the user data columns.

End of data can be represented by a single line containing just backslash-period \(`\.`\). An end-of-data marker is not necessary when reading from a file, since the end of file serves perfectly well; it is needed only when copying data to or from client applications using pre-3.0 client protocol.

Backslash characters \(`\`\) can be used in the `COPY` data to quote data characters that might otherwise be taken as row or column delimiters. In particular, the following characters _must_ be preceded by a backslash if they appear as part of a column value: backslash itself, newline, carriage return, and the current delimiter character.

The specified null string is sent by `COPY TO` without adding any backslashes; conversely, `COPY FROM` matches the input against the null string before removing backslashes. Therefore, a null string such as `\N` cannot be confused with the actual data value `\N` \(which would be represented as `\\N`\).

The following special backslash sequences are recognized by `COPY FROM`:

| Sequence | Represents |
| :--- | :--- |
| `\b` | Backspace \(ASCII 8\) |
| `\f` | Form feed \(ASCII 12\) |
| `\n` | Newline \(ASCII 10\) |
| `\r` | Carriage return \(ASCII 13\) |
| `\t` | Tab \(ASCII 9\) |
| `\v` | Vertical tab \(ASCII 11\) |
| `\`_`digits`_ | Backslash followed by one to three octal digits specifies the character with that numeric code |
| `\x`_`digits`_ | Backslash `x` followed by one or two hex digits specifies the character with that numeric code |

Presently, `COPY TO` will never emit an octal or hex-digits backslash sequence, but it does use the other sequences listed above for those control characters.

Any other backslashed character that is not mentioned in the above table will be taken to represent itself. However, beware of adding backslashes unnecessarily, since that might accidentally produce a string matching the end-of-data marker \(`\.`\) or the null string \(`\N` by default\). These strings will be recognized before any other backslash processing is done.

It is strongly recommended that applications generating `COPY` data convert data newlines and carriage returns to the `\n` and `\r` sequences respectively. At present it is possible to represent a data carriage return by a backslash and carriage return, and to represent a data newline by a backslash and newline. However, these representations might not be accepted in future releases. They are also highly vulnerable to corruption if the `COPY` file is transferred across different machines \(for example, from Unix to Windows or vice versa\).

`COPY TO` will terminate each row with a Unix-style newline \(“`\n`”\). Servers running on Microsoft Windows instead output carriage return/newline \(“`\r\n`”\), but only for `COPY` to a server file; for consistency across platforms, `COPY TO STDOUT` always sends “`\n`” regardless of server platform. `COPY FROM` can handle lines ending with newlines, carriage returns, or carriage return/newlines. To reduce the risk of error due to un-backslashed newlines or carriage returns that were meant as data, `COPY FROM` will complain if the line endings in the input are not all alike.

#### CSV Format

This format option is used for importing and exporting the Comma Separated Value \(`CSV`\) file format used by many other programs, such as spreadsheets. Instead of the escaping rules used by PostgreSQL's standard text format, it produces and recognizes the common CSV escaping mechanism.

The values in each record are separated by the `DELIMITER` character. If the value contains the delimiter character, the `QUOTE` character, the `NULL` string, a carriage return, or line feed character, then the whole value is prefixed and suffixed by the `QUOTE` character, and any occurrence within the value of a `QUOTE` character or the `ESCAPE` character is preceded by the escape character. You can also use `FORCE_QUOTE` to force quotes when outputting non-`NULL` values in specific columns.

The `CSV` format has no standard way to distinguish a `NULL` value from an empty string. PostgreSQL's `COPY` handles this by quoting. A `NULL` is output as the `NULL` parameter string and is not quoted, while a non-`NULL` value matching the `NULL` parameter string is quoted. For example, with the default settings, a `NULL` is written as an unquoted empty string, while an empty string data value is written with double quotes \(`""`\). Reading values follows similar rules. You can use `FORCE_NOT_NULL` to prevent `NULL` input comparisons for specific columns. You can also use `FORCE_NULL` to convert quoted null string data values to `NULL`.

Because backslash is not a special character in the `CSV` format, `\.`, the end-of-data marker, could also appear as a data value. To avoid any misinterpretation, a `\.` data value appearing as a lone entry on a line is automatically quoted on output, and on input, if quoted, is not interpreted as the end-of-data marker. If you are loading a file created by another application that has a single unquoted column and might have a value of `\.`, you might need to quote that value in the input file.

#### Note

In `CSV` format, all characters are significant. A quoted value surrounded by white space, or any characters other than `DELIMITER`, will include those characters. This can cause errors if you import data from a system that pads `CSV` lines with white space out to some fixed width. If such a situation arises you might need to preprocess the `CSV` file to remove the trailing white space, before importing the data into PostgreSQL.

#### Note

CSV format will both recognize and produce CSV files with quoted values containing embedded carriage returns and line feeds. Thus the files are not strictly one line per table row like text-format files.

#### Note

Many programs produce strange and occasionally perverse CSV files, so the file format is more a convention than a standard. Thus you might encounter some files that cannot be imported using this mechanism, and `COPY` might produce files that other programs cannot process.

#### Binary Format

The `binary` format option causes all data to be stored/read as binary format rather than as text. It is somewhat faster than the text and `CSV` formats, but a binary-format file is less portable across machine architectures and PostgreSQL versions. Also, the binary format is very data type specific; for example it will not work to output binary data from a `smallint` column and read it into an `integer` column, even though that would work fine in text format.

The `binary` file format consists of a file header, zero or more tuples containing the row data, and a file trailer. Headers and data are in network byte order.

#### Note

PostgreSQL releases before 7.4 used a different binary file format.

**File Header**

The file header consists of 15 bytes of fixed fields, followed by a variable-length header extension area. The fixed fields are:Signature

11-byte sequence `PGCOPY\n\377\r\n\0` — note that the zero byte is a required part of the signature. \(The signature is designed to allow easy identification of files that have been munged by a non-8-bit-clean transfer. This signature will be changed by end-of-line-translation filters, dropped zero bytes, dropped high bits, or parity changes.\)Flags field

32-bit integer bit mask to denote important aspects of the file format. Bits are numbered from 0 \(LSB\) to 31 \(MSB\). Note that this field is stored in network byte order \(most significant byte first\), as are all the integer fields used in the file format. Bits 16-31 are reserved to denote critical file format issues; a reader should abort if it finds an unexpected bit set in this range. Bits 0-15 are reserved to signal backwards-compatible format issues; a reader should simply ignore any unexpected bits set in this range. Currently only one flag bit is defined, and the rest must be zero:Bit 16

if 1, OIDs are included in the data; if 0, notHeader extension area length

32-bit integer, length in bytes of remainder of header, not including self. Currently, this is zero, and the first tuple follows immediately. Future changes to the format might allow additional data to be present in the header. A reader should silently skip over any header extension data it does not know what to do with.

The header extension area is envisioned to contain a sequence of self-identifying chunks. The flags field is not intended to tell readers what is in the extension area. Specific design of header extension contents is left for a later release.

This design allows for both backwards-compatible header additions \(add header extension chunks, or set low-order flag bits\) and non-backwards-compatible changes \(set high-order flag bits to signal such changes, and add supporting data to the extension area if needed\).

**Tuples**

Each tuple begins with a 16-bit integer count of the number of fields in the tuple. \(Presently, all tuples in a table will have the same count, but that might not always be true.\) Then, repeated for each field in the tuple, there is a 32-bit length word followed by that many bytes of field data. \(The length word does not include itself, and can be zero.\) As a special case, -1 indicates a NULL field value. No value bytes follow in the NULL case.

There is no alignment padding or any other extra data between fields.

Presently, all data values in a binary-format file are assumed to be in binary format \(format code one\). It is anticipated that a future extension might add a header field that allows per-column format codes to be specified.

To determine the appropriate binary format for the actual tuple data you should consult the PostgreSQL source, in particular the `*send` and `*recv` functions for each column's data type \(typically these functions are found in the `src/backend/utils/adt/` directory of the source distribution\).

If OIDs are included in the file, the OID field immediately follows the field-count word. It is a normal field except that it's not included in the field-count. In particular it has a length word — this will allow handling of 4-byte vs. 8-byte OIDs without too much pain, and will allow OIDs to be shown as null if that ever proves desirable.

**File Trailer**

The file trailer consists of a 16-bit integer word containing -1. This is easily distinguished from a tuple's field-count word.

A reader should report an error if a field-count word is neither -1 nor the expected number of columns. This provides an extra check against somehow getting out of sync with the data.

### 範例

以下範例使用破折號「\|」作為欄位分隔符把資料表複製到用戶端：

```text
COPY country TO STDOUT (DELIMITER '|');
```

要將檔案中的資料複製到 country 資料表中：

```text
COPY country FROM '/usr1/proj/bray/sql/country_data';
```

要將名稱以「A」開頭的國家複製到檔案中：

```text
COPY (SELECT * FROM country WHERE country_name LIKE 'A%') TO '/usr1/proj/bray/sql/a_list_countries.copy';
```

要複製到壓縮檔案，可以透過外部壓縮程序輸出：

```text
COPY country TO PROGRAM 'gzip > /usr1/proj/bray/sql/country_data.gz';
```

以下是適合從 STDIN 複製到資料表中的資料範例：

```text
AF      AFGHANISTAN
AL      ALBANIA
DZ      ALGERIA
ZM      ZAMBIA
ZW      ZIMBABWE
```

請注意，每行上的空白實際上是 tab 字元。

以下是相同的資料，以二進位格式輸出。在透過 Unix 實用工具 od -c 過濾後顯示資料。該資料表有三個欄位；第一個是 char\(2\) 型別，第二個是 text 型別，第三個是 integer 型別。所有行在第三欄位中都具有空值。

```text
0000000   P   G   C   O   P   Y  \n 377  \r  \n  \0  \0  \0  \0  \0  \0
0000020  \0  \0  \0  \0 003  \0  \0  \0 002   A   F  \0  \0  \0 013   A
0000040   F   G   H   A   N   I   S   T   A   N 377 377 377 377  \0 003
0000060  \0  \0  \0 002   A   L  \0  \0  \0 007   A   L   B   A   N   I
0000100   A 377 377 377 377  \0 003  \0  \0  \0 002   D   Z  \0  \0  \0
0000120 007   A   L   G   E   R   I   A 377 377 377 377  \0 003  \0  \0
0000140  \0 002   Z   M  \0  \0  \0 006   Z   A   M   B   I   A 377 377
0000160 377 377  \0 003  \0  \0  \0 002   Z   W  \0  \0  \0  \b   Z   I
0000200   M   B   A   B   W   E 377 377 377 377 377 377
```

### 相容性

SQL 標準中沒有 COPY 語句。

在 PostgreSQL 版本 9.0 之前使用了以下語法並且仍然支援：

```text
COPY table_name [ ( column_name [, ...] ) ]
    FROM { 'filename' | STDIN }
    [ [ WITH ]
          [ BINARY ]
          [ OIDS ]
          [ DELIMITER [ AS ] 'delimiter' ]
          [ NULL [ AS ] 'null string' ]
          [ CSV [ HEADER ]
                [ QUOTE [ AS ] 'quote' ]
                [ ESCAPE [ AS ] 'escape' ]
                [ FORCE NOT NULL column_name [, ...] ] ] ]

COPY { table_name [ ( column_name [, ...] ) ] | ( query ) }
    TO { 'filename' | STDOUT }
    [ [ WITH ]
          [ BINARY ]
          [ OIDS ]
          [ DELIMITER [ AS ] 'delimiter' ]
          [ NULL [ AS ] 'null string' ]
          [ CSV [ HEADER ]
                [ QUOTE [ AS ] 'quote' ]
                [ ESCAPE [ AS ] 'escape' ]
                [ FORCE QUOTE { column_name [, ...] | * } ] ] ]
```

請注意，在此語法中，BINARY 和 CSV 被視為獨立的關鍵字，而不是 FORMAT 選項的參數。

在 PostgreSQL 版本 7.3 之前使用了以下語法，並且仍然支援：

```text
COPY [ BINARY ] table_name [ WITH OIDS ]
    FROM { 'filename' | STDIN }
    [ [USING] DELIMITERS 'delimiter' ]
    [ WITH NULL AS 'null string' ]

COPY [ BINARY ] table_name [ WITH OIDS ]
    TO { 'filename' | STDOUT }
    [ [USING] DELIMITERS 'delimiter' ]
    [ WITH NULL AS 'null string' ]
```

