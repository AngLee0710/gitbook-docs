# ALTER TABLE

ALTER TABLE — 變更資料表的定義

### 語法

```text
ALTER TABLE [ IF EXISTS ] [ ONLY ] name [ * ]
    action [, ... ]
ALTER TABLE [ IF EXISTS ] [ ONLY ] name [ * ]
    RENAME [ COLUMN ] column_name TO new_column_name
ALTER TABLE [ IF EXISTS ] [ ONLY ] name [ * ]
    RENAME CONSTRAINT constraint_name TO new_constraint_name
ALTER TABLE [ IF EXISTS ] name
    RENAME TO new_name
ALTER TABLE [ IF EXISTS ] name
    SET SCHEMA new_schema
ALTER TABLE ALL IN TABLESPACE name [ OWNED BY role_name [, ... ] ]
    SET TABLESPACE new_tablespace [ NOWAIT ]
ALTER TABLE [ IF EXISTS ] name
    ATTACH PARTITION partition_name FOR VALUES partition_bound_spec
ALTER TABLE [ IF EXISTS ] name
    DETACH PARTITION partition_name

where action is one of:

    ADD [ COLUMN ] [ IF NOT EXISTS ] column_name data_type [ COLLATE collation ] [ column_constraint [ ... ] ]
    DROP [ COLUMN ] [ IF EXISTS ] column_name [ RESTRICT | CASCADE ]
    ALTER [ COLUMN ] column_name [ SET DATA ] TYPE data_type [ COLLATE collation ] [ USING expression ]
    ALTER [ COLUMN ] column_name SET DEFAULT expression
    ALTER [ COLUMN ] column_name DROP DEFAULT
    ALTER [ COLUMN ] column_name { SET | DROP } NOT NULL
    ALTER [ COLUMN ] column_name ADD GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY [ ( sequence_options ) ]
    ALTER [ COLUMN ] column_name { SET GENERATED { ALWAYS | BY DEFAULT } | SET sequence_option | RESTART [ [ WITH ] restart ] } [...]
    ALTER [ COLUMN ] column_name DROP IDENTITY [ IF EXISTS ]
    ALTER [ COLUMN ] column_name SET STATISTICS integer
    ALTER [ COLUMN ] column_name SET ( attribute_option = value [, ... ] )
    ALTER [ COLUMN ] column_name RESET ( attribute_option [, ... ] )
    ALTER [ COLUMN ] column_name SET STORAGE { PLAIN | EXTERNAL | EXTENDED | MAIN }
    ADD table_constraint [ NOT VALID ]
    ADD table_constraint_using_index
    ALTER CONSTRAINT constraint_name [ DEFERRABLE | NOT DEFERRABLE ] [ INITIALLY DEFERRED | INITIALLY IMMEDIATE ]
    VALIDATE CONSTRAINT constraint_name
    DROP CONSTRAINT [ IF EXISTS ]  constraint_name [ RESTRICT | CASCADE ]
    DISABLE TRIGGER [ trigger_name | ALL | USER ]
    ENABLE TRIGGER [ trigger_name | ALL | USER ]
    ENABLE REPLICA TRIGGER trigger_name
    ENABLE ALWAYS TRIGGER trigger_name
    DISABLE RULE rewrite_rule_name
    ENABLE RULE rewrite_rule_name
    ENABLE REPLICA RULE rewrite_rule_name
    ENABLE ALWAYS RULE rewrite_rule_name
    DISABLE ROW LEVEL SECURITY
    ENABLE ROW LEVEL SECURITY
    FORCE ROW LEVEL SECURITY
    NO FORCE ROW LEVEL SECURITY
    CLUSTER ON index_name
    SET WITHOUT CLUSTER
    SET WITH OIDS
    SET WITHOUT OIDS
    SET TABLESPACE new_tablespace
    SET { LOGGED | UNLOGGED }
    SET ( storage_parameter = value [, ... ] )
    RESET ( storage_parameter [, ... ] )
    INHERIT parent_table
    NO INHERIT parent_table
    OF type_name
    NOT OF
    OWNER TO { new_owner | CURRENT_USER | SESSION_USER }
    REPLICA IDENTITY { DEFAULT | USING INDEX index_name | FULL | NOTHING }

and table_constraint_using_index is:

    [ CONSTRAINT constraint_name ]
    { UNIQUE | PRIMARY KEY } USING INDEX index_name
    [ DEFERRABLE | NOT DEFERRABLE ] [ INITIALLY DEFERRED | INITIALLY IMMEDIATE ]
```

### 說明

`ALTER TABLE` 變更現有資料表的定義。有幾個子命令描述如下。請注意，每個子命令所需的鎖定等級可能不同。除非明確指出，否則都是 ACCESS EXCLUSIVE 鎖定。當列出多個子命令時，所有子命令所需的鎖以最嚴格的為準。

`ADD COLUMN [ IF NOT EXISTS ]`

該資料表使用與 [CREATE TABLE](create-table.md) 相同的語法在資料表中增加一個新的欄位。如果 IF NOT EXISTS 被指定，並且欄位已經存在這個名稱，則可以避免引發錯誤。

`DROP COLUMN [ IF EXISTS ]`

從該資料表中刪除一個欄位。涉及該欄位的索引和資料表限制條件也將自動刪除。如果刪除的欄位會導致統計信息僅包含單個欄位的資料的話，那麼引用刪除欄位的多變量統計數據也將被刪除。如果資料表外的任何內容取決於該欄位，例如外部鍵引用或 view，則需要使用 CASCADE。 如果指定 IF EXISTS 但該欄位卻不存在，則不會引發錯誤。通常在這種情況下，會發出提示訊息。

`SET DATA TYPE`

這種語法用於變更一個資料表中欄位的資料型別。涉及該欄位的索引和簡單的資料表限制條件將透過重新分析原始提供的表示式自動轉換為使用新的欄位型別。可選用的 COLLATE 子句指定新欄位的排序規則；如果省略的話，則排序規則是新欄位型別的預設值。可選用的 USING 子句指定如何從舊值計算為新的欄位值；如果省略，則預設轉換與從舊資料類型到新欄位轉換的賦值相同。 如果沒有隱含或賦值從舊型別轉換為新型別，則必須提供 USING 子句。

`SET`/`DROP DEFAULT`

這個語法設定或刪除欄位的預設值。預設值僅適用於其後續的 INSERT 或 UPDATE 指令；它不會變更資料表中已有的資料列。

`SET`/`DROP NOT NULL`

這個語法會變更欄位是否標記為允許空值或拒絕空值。當欄位不應該包含空值時，您就可以使用 SET NOT NULL。

如果此資料表是一個資料表分割區，而在父資料表中標記為 NOT NULL，則不能在欄位上執行 DROP NOT NULL。要從所有分割區中刪除 NOT NULL 約束，請在父資料表上執行 DROP NOT NULL。即使父級沒有 NOT NULL 限制條件，如果需要，這樣的限制條件仍然可以加到單獨的分割區中；也就是說，即使父資料表允許他們，子資料表們也可以不允許使用空值，但是反過來也是如此。

`ADD GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY`  
`SET GENERATED { ALWAYS | BY DEFAULT }`  
`DROP IDENTITY [ IF EXISTS ]`

這個語法會變更欄位是否為標識欄位\(identity column\)或變更現有標識欄位的生成屬性。有關詳細訊息，請參閱 [CREATE TABLE](create-table.md)。

如果指定了 DROP IDENTITY IF EXISTS 而該欄位不是標識欄位，則不會引發錯誤。 在這種情況下，會發布通知。

`SET` _`sequence_option`_  
`RESTART`

這個語法變更現有標識欄下的序列設定。sequence\_option 是 [ALTER SEQUENCE](alter-sequence.md) 支援的選項，像是 INCREMENT BY。

`SET STATISTICS`

此語法為隨後的 [ANALYZE](analyze.md) 操作設定每個欄位的統計目標。目標可以設定在 0 到 10000 範圍內；或者，將其設定為 -1 以恢復為使用系統預設的統計訊息目標（[default\_statistics\_target](../../server-administration/runtime-config/query-planning.md#19-7-4-other-planner-options)）。有關 PostgreSQL 查詢規劃器使用統計訊息的更多資訊，請參閱[第 14.2 節](../../sql/performance-tips/planner-stats.md)。

`SET STATISTICS` 會要求一個 `SHARE UPDATE EXCLUSIVE` 的鎖定。

`SET (` _`attribute_option`_ = _`value`_ \[, ... \] \)  
`RESET (` _`attribute_option`_ \[, ... \] \)

此語法設定或重置每個屬性選項。目前，只有定義的每個屬性選項是 n\_distinct 和 n\_distinct\_inherited，它們會覆蓋後續 [ANALYZE](analyze.md) 操作所做的不同值的估計數量。 n\_distinct 會影響資料表本身的統計訊息，而 n\_distinct\_inherited 會影響為該表及其繼承子資料表所收集的統計訊息。當設定為正值時，ANALYZE 將假定該欄位正好包含指定數量的相異非空值。當設定為負值（必須大於或等於 -1）時，ANALYZE 將假定欄位中相異非空值的數量與表的大小成線性關係；準確的計數是透過將估計的資料表大小乘以給定數字的絕對值來計算。例如，值 -1 意味著欄位中的所有值都是不同的，而值 -0.5 意味著每個值在平均值上會出現兩次。當資料表的大小隨時間變化時這很有用，因為在查詢計劃階段之前，不會執行資料表中行數的乘法運算。指定值 0 以恢復到一般性估計不同值的數量。有關 PostgreSQL 查詢規劃器使用統計資訊的更多訊息，請參閱[第 14.2 節](../../sql/performance-tips/planner-stats.md)。

變更每個屬性選項會要求取得一個 SHARE UPDATE EXCLUSIVE 鎖定。

`SET STORAGE`

此語法設定欄位的儲存模式。 這將控制此欄位是以內建方式保存還是以輔助 TOAST 方式保存，以及是否應該壓縮資料。PLAIN 必須用於固定長度值（如整數），並且是內建的，未壓縮的。MAIN 用於內建可壓縮資料。EXTERNAL 用於外部未壓縮資料，EXTENDED 用於外部壓縮資料。EXTENDED 是非 PLAIN 儲存的大多數資料型別的預設值。 使用 EXTERNAL 將使得對非常大的字串和 bytea 值進行子字串處理的速度更快，從而增加儲存空間。請注意，SET STORAGE 本身並不會改變資料表中的任何內容，它只是設定在將來的資料表更新期間追求的策略。有關更多訊息，請參閱[第 66.2 節](../../internals/database-physical-storage/toast.md)。

`ADD` _`table_constraint`_ \[ NOT VALID \]

此語法用於與 [CREATE TABLE](create-table.md) 相同的語法為資料表加上一個新的限制條件，並可以加上選項 NOT VALID，該選項目前只允許用於外部鍵和 CHECK 限制條件。如果限制條件被標記為 NOT VALID，則跳過用於驗證資料表中的所有資料列滿足限制條件的冗長初始檢查。對於後續的插入或更新，這個檢查仍然會被執行（也就是說，除非在被引用的資料表中存在有匹配的資料，否則在外部鍵的情況下它們將會失敗；並且除非新的資料列匹配指定的檢查，否則它們將會失敗）。但是，資料庫不會假定該限制條件適用於資料表中的所有的資料，直到透過使用 VALIDATE CONSTRAINT 選項進行驗證。

`ADD` _`table_constraint_using_index`_

此語法根據現有的唯一索引向資料表中增加新的 PRIMARY KEY 或 UNIQUE 限制條件。索引中的所有欄位都將包含在限制條件裡。

索引不能有表示式欄位，也不能是部分索引。此外，它必須是具有隱含排序順序的 b-tree 索引。這些限制可確保索引等同於由常態的 ADD PRIMARY KEY 或 ADD UNIQUE 指令建立的索引。

如果指定了 PRIMARY KEY，並且索引的欄位尚未標記為 NOT NULL，那麼此命令將嘗試對每個此類的欄位執行 ALTER COLUMN SET NOT NULL。這需要全資料表掃描來驗證列不包含空值。在所有其他情況下，這是一項快速的操作。

如果提供限制條件名稱，那麼索引將被重新命名以匹配限制條件名稱。否則，限制條件將被命名為與索引相同。

執行此命令後，索引由該限制條件「擁有」，就像索引由一般的 ADD PRIMARY KEY 或 ADD UNIQUE 命令建立的一樣。特別要注意是，刪除限制條件會使索引消失。

#### 注意

在需要增加新的限制條件情況下，使用現有索引加上約束可能會很有幫助。這種情況下需要加上新限制條件需要很長一段時間但不會阻斷資料表更新。為此，請使用 CREATE INDEX CONCURRENTLY 建立索引，然後使用此語法將其作為官方限制條件進行安裝。請參閱後續的例子。

`ALTER CONSTRAINT`

在資料表變更先前建立限制條件屬性。目前只有外部鍵限制條件可以變更。

`VALIDATE CONSTRAINT`

此語法透過掃描資料表來驗證先前設定為 NOT VALID 的外部鍵或檢查限制條件，以確保沒有不滿足限制條件的資料列。如果限制條件已被標記為有效，則不會產生任何行為。

驗證對於大型資料表可能是一個漫長的過程。將驗證與初始設定分離的價值在於，您可以將驗證延遲到不太繁忙的時間處理，或者可以用來給額外的時間來糾正原先存在的錯誤，同時防止出現新的錯誤。另請注意，驗證本身並不妨礙它在執行時對該資料表的正常寫入命令。

驗證僅獲取變更資料表上的 SHARE UPDATE EXCLUSIVE 鎖定。 如果限制條件是外部鍵，那麼在限制條件引用的資料表上也需要 ROW SHARE 鎖定。

`DROP CONSTRAINT [ IF EXISTS ]`

這個語法會在資料表上刪除指定的限制條件。如果指定了 IF EXISTS 並且該限制條件不存在，就不會引發錯誤。在這種情況下，會發布提示通知。

`DISABLE`/`ENABLE [ REPLICA | ALWAYS ] TRIGGER`

這個語法設定這個資料表的觸發器。被禁用的觸發器仍然是系統已知的，只是在發生觸發事件時不會執行而已。對於延遲觸發器，當事件發生時檢查啟用狀態，而不是在實際執行觸發器函數時檢查。可以禁用或啟用由名稱指定的單個觸發器或資料表中的所有觸發器，或者僅禁用使用者的觸發器（此選項不包括內部生成的限制條件觸發器，例如用於實作外部鍵約束或可延遲唯一性和排除限制條件的觸發器）。禁用或啟用內部生成的限制條件觸發器需要超級使用者權限；應該謹慎對待，因為如果不執行觸發器，限制條件的完整性將無法得到保證。觸發器的觸發機制也會受設定變數 [session\_replication\_role](../../server-administration/runtime-config/runtime-config-client.md#session_replication_role-enum) 的影響。當複製角色是「origin」（預設）或「local」時，只需啟用觸發器就會觸發。配置為 ENABLE REPLICA 的觸發器只會在連線處於「replica」模式時觸發，而設定為 ENABLE ALWAYS 的觸發器將觸發，不論目前的複複模式為何。

此指令會取得一個 SHARE ROW EXCLUSIVE 鎖定。

`DISABLE`/`ENABLE [ REPLICA | ALWAYS ] RULE`

這個語法設定屬於資料表的覆寫規則的觸發。禁用的規則仍為系統所知的，但在查詢覆寫期間不適用。其語法義意和禁用/啟用觸發器一樣。對於 ON SELECT 規則，將忽略此設定，即使目前連線處於非預設的複製角色，也始終使用此設定以保持 view 能正常工作。

`DISABLE`/`ENABLE ROW LEVEL SECURITY`

這個語法為資料表控制屬於資料表的資料列級安全原則的適用。 如果啟用且該資料表不存在任何安全原則，則應用預設為拒絕原則。請注意，即使資料列級安全性被禁用，安全原則也可以存在於資料表中 - 在這種情況下，原則將不會被採用，它們將被忽略。 另請參閱 [CREATE POLICY](create-policy.md)。

`NO FORCE`/`FORCE ROW LEVEL SECURITY`

當使用者為資料表的所有者時，這個語法控制屬於資料表的資料列安全原則的使用。如果啟用，則在使用者是資料表所有者時應用資料列級安全原則。 如果禁用（預設），那麼當使用者是資料表所有者時，資料列級安全性將不會被應用。另請參閱 [CREATE POLICY](create-policy.md)。

`CLUSTER ON`

此資料表為將來的 [CLUSTER](cluster.md) 操作選擇預設索引。但它實際上並不會重組資料表。

變更叢集選項將取得 SHARE UPDATE EXCLUSIVE 鎖定。

`SET WITHOUT CLUSTER`

此語法從資料表中刪除最近使用的 CLUSTER 索引設定。這會影響未指定索引的後續叢集操作。

 變更叢集選項將取得 SHARE UPDATE EXCLUSIVE 鎖定。

`SET WITH OIDS`

此語法在資料表中增加了一個 oid 系統欄位（參閱[第 5.4 節](../../sql/ddl/system-columns.md)）。 如果資料表已經有 OID，那就什麼都不做。

請注意，這不等同於 ADD COLUMN oid oid；那只會增加一個正常的欄位，而它碰巧被命名為 oid，而不是系統欄位。

`SET WITHOUT OIDS`

此語法從資料表中移除 oid 系統欄位。這完全等同於 DROP COLUMN oid RESTRICT，只是如果已經沒有 oid 欄位，它不會有動作產生。

`SET TABLESPACE`

此語法將資料表的資料表空間更改為指定的資料表空間，並將與資料表關聯的資料檔案移動到新的資料表空間。資料表中的索引（如果有的話）不會移動；但它們可以通過額外的 SET TABLESPACE 指令單獨移動。資料表空間中目前資料庫中的所有資料表都可以通過使用 ALL IN TABLESPACE 語法來移動，它將鎖定所有要移動的資料表，然後移動每個資料表。這種語法也支持 OWNED BY，它只會移動指定角色擁有的資料表。 如果指定了 NOWAIT 選項，那麼如果無法立即取得所有需要的鎖定，該指令將失敗。請注意，如果需要，系統目錄不會被此指令移動，而是使用 ALTER DATABASE 或 ALTER TABLE 呼叫。information\_schema 關連不被視為系統目錄的一部分，將會被移動。另請參閱 [CREATE TABLESPACE](create-tablespace.md)。

`SET { LOGGED | UNLOGGED }`

此子句會將資料表從無日誌資料表變更為有日誌資料表或反之亦然（請參閱 [UNLOGGED](create-table.md)）。它不能用於臨時資料表。

`SET (` _`storage_parameter`_ = _`value`_ \[, ... \] \)

此子句變更資料表的一個或多個儲存參數。有關可用參數的詳細訊息，請參閱[儲存參數選項](create-table.md#storage-parameters)。請注意，這個指令不會立即修改資料表內容；根據參數，您可能需要重填資料表以獲得所需的效果。這可以透過 [VACUUM FULL](vacuum.md)、[CLUSTER](cluster.md)或強制重填資料表的 ALTER TABLE 形式來完成。對於與規劃器相關的參數，更改將在下次資料表鎖定時生效，因此目前執行的查詢不會受到影響。

SHARE UPDATE EXCLUSIVE 會針對 fillfactor 和 autovacuum 儲存參數以及以下計劃程序的相關參數進行鎖定：effective\_io\_concurrency，parallel\_workers，seq\_page\_cost，random\_page\_cost，n\_distinct 和 n\_distinct\_inherited。

#### 注意

雖然 CREATE TABLE 允許在 WITH（storage\_parameter）語法中指定 OIDS，但 ALTER TABLE 不會將 OIDS 視為儲存參數。而是使用 SET WITH OIDS 和 SET WITHOUT OIDS 語法來變更 OID 狀態。

`RESET (` _`storage_parameter`_ \[, ... \] \)

此語法將一個或多個儲存參數重置為其預設值。 和 SET 一樣，可能需要重新寫入資料來完成更新其效果。

`INHERIT` _`parent_table`_

此子句將目標資料表加到指定的父資料表中成為新的子資料表。然後，針對父資料表的查詢將會包含目標資料表的資料。要作為子資料表加入前，目標資料表必須已經包含與父資料表的所有欄位（它也可以具有其他欄位）。這些欄位必須具有可匹配的資料型別，並且如果它們在父資料表中具有 NOT NULL 限制條件，那麼它們還必須在子資料表中也具有 NOT NULL 限制條件。

對於父資料表的所有 CHECK 限制條件，必須還有相對應的子資料表限制條件，除非父資料表中標記為不可繼承（即使用ALTER TABLE ... ADD CONSTRAINT ... NO INHERIT 所建立的，它們將會被忽略；所有匹配的子資料表限制條件不得標記為不可繼承。目前不用考慮 UNIQUE，PRIMARY KEY 和 FOREIGN KEY，但未來這些可能會改變。

`NO INHERIT` _`parent_table`_

此字句從指定的父資料表的子資料表中刪除目標資料表。針對父資料表的查詢將不再包含從目標資料表中所産生的記錄。

`OF` _`type_name`_

此子句將資料表連接到複合型別，就像 CREATE TABLE OF 已經産生它一樣。該資料表的欄位名稱和型別必須與組合型別的列表完全吻合；oid 系統欄位的存在會有所不同。該資料表不得從任何其他的資料表繼承。這些限制確保了 CREATE TABLE OF 將得到一個等效的資料表定義。

`NOT OF`

此子句將複合型別資料表從它的型別中分離出來。

`OWNER`

該子句將資料表、序列、檢視表、具體化檢視表或外部資料表的擁有者變更為指定的使用者。

`REPLICA IDENTITY`

此子句變更寫入 WAL 的訊息，以識別更新或刪除的資料列。如果正在使用邏輯複製的話，則此子句不起作用。DEFAULT（非系統資料表的預設值）記錄主鍵欄位的舊值（如果有的話）。USING INDEX 記錄指定索引覆蓋欄位的舊值，它必須是唯一的，不能是部分的，也不可是延遲的，並且只能包含標記為 NOT NULL 的欄位。FULL 記錄行中所有欄位的舊值。 沒有記錄關於舊資料列的訊息。（這是系統資料表的預設值。）在任何情況下，都不記錄舊值，除非至少有一個將記錄的欄位在新舊版本的資料列之間不同。

`RENAME`

The `RENAME` forms change the name of a table \(or an index, sequence, view, materialized view, or foreign table\), the name of an individual column in a table, or the name of a constraint of the table. There is no effect on the stored data.

`SET SCHEMA`

This form moves the table into another schema. Associated indexes, constraints, and sequences owned by table columns are moved as well.

`ATTACH PARTITION` _`partition_name`_ FOR VALUES _`partition_bound_spec`_

This form attaches an existing table \(which might itself be partitioned\) as a partition of the target table using the same syntax for _`partition_bound_spec`_ as [CREATE TABLE](https://www.postgresql.org/docs/10/static/sql-createtable.html). The partition bound specification must correspond to the partitioning strategy and partition key of the target table. The table to be attached must have all the same columns as the target table and no more; moreover, the column types must also match. Also, it must have all the `NOT NULL` and `CHECK` constraints of the target table. Currently `UNIQUE`, `PRIMARY KEY`, and `FOREIGN KEY` constraints are not considered. If any of the `CHECK` constraints of the table being attached is marked `NO INHERIT`, the command will fail; such a constraint must be recreated without the `NO INHERIT` clause.

If the new partition is a regular table, a full table scan is performed to check that no existing row in the table violates the partition constraint. It is possible to avoid this scan by adding a valid `CHECK` constraint to the table that would allow only the rows satisfying the desired partition constraint before running this command. It will be determined using such a constraint that the table need not be scanned to validate the partition constraint. This does not work, however, if any of the partition keys is an expression and the partition does not accept `NULL` values. If attaching a list partition that will not accept `NULL` values, also add `NOT NULL` constraint to the partition key column, unless it's an expression.

If the new partition is a foreign table, nothing is done to verify that all the rows in the foreign table obey the partition constraint. \(See the discussion in [CREATE FOREIGN TABLE](https://www.postgresql.org/docs/10/static/sql-createforeigntable.html) about constraints on the foreign table.\)

`DETACH PARTITION` _`partition_name`_

This form detaches specified partition of the target table. The detached partition continues to exist as a standalone table, but no longer has any ties to the table from which it was detached.

All the forms of ALTER TABLE that act on a single table, except `RENAME`, `SET SCHEMA`, `ATTACH PARTITION`, and `DETACH PARTITION` can be combined into a list of multiple alterations to be applied together. For example, it is possible to add several columns and/or alter the type of several columns in a single command. This is particularly useful with large tables, since only one pass over the table need be made.

You must own the table to use `ALTER TABLE`. To change the schema or tablespace of a table, you must also have `CREATE` privilege on the new schema or tablespace. To add the table as a new child of a parent table, you must own the parent table as well. Also, to attach a table as a new partition of the table, you must own the table being attached. To alter the owner, you must also be a direct or indirect member of the new owning role, and that role must have `CREATE` privilege on the table's schema. \(These restrictions enforce that altering the owner doesn't do anything you couldn't do by dropping and recreating the table. However, a superuser can alter ownership of any table anyway.\) To add a column or alter a column type or use the `OF` clause, you must also have `USAGE` privilege on the data type.

### Parameters

`IF EXISTS`

Do not throw an error if the table does not exist. A notice is issued in this case.

_`name`_

The name \(optionally schema-qualified\) of an existing table to alter. If `ONLY` is specified before the table name, only that table is altered. If `ONLY` is not specified, the table and all its descendant tables \(if any\) are altered. Optionally, `*` can be specified after the table name to explicitly indicate that descendant tables are included.

_`column_name`_

Name of a new or existing column.

_`new_column_name`_

New name for an existing column.

_`new_name`_

New name for the table.

_`data_type`_

Data type of the new column, or new data type for an existing column.

_`table_constraint`_

New table constraint for the table.

_`constraint_name`_

Name of a new or existing constraint.

`CASCADE`

Automatically drop objects that depend on the dropped column or constraint \(for example, views referencing the column\), and in turn all objects that depend on those objects \(see [Section 5.13](https://www.postgresql.org/docs/10/static/ddl-depend.html)\).

`RESTRICT`

Refuse to drop the column or constraint if there are any dependent objects. This is the default behavior.

_`trigger_name`_

Name of a single trigger to disable or enable.

`ALL`

Disable or enable all triggers belonging to the table. \(This requires superuser privilege if any of the triggers are internally generated constraint triggers such as those that are used to implement foreign key constraints or deferrable uniqueness and exclusion constraints.\)

`USER`

Disable or enable all triggers belonging to the table except for internally generated constraint triggers such as those that are used to implement foreign key constraints or deferrable uniqueness and exclusion constraints.

_`index_name`_

The name of an existing index.

_`storage_parameter`_

The name of a table storage parameter.

_`value`_

The new value for a table storage parameter. This might be a number or a word depending on the parameter.

_`parent_table`_

A parent table to associate or de-associate with this table.

_`new_owner`_

The user name of the new owner of the table.

_`new_tablespace`_

The name of the tablespace to which the table will be moved.

_`new_schema`_

The name of the schema to which the table will be moved.

_`partition_name`_

The name of the table to attach as a new partition or to detach from this table.

_`partition_bound_spec`_

The partition bound specification for a new partition. Refer to [CREATE TABLE](https://www.postgresql.org/docs/10/static/sql-createtable.html) for more details on the syntax of the same.

### Notes

The key word `COLUMN` is noise and can be omitted.

When a column is added with `ADD COLUMN`, all existing rows in the table are initialized with the column's default value \(NULL if no `DEFAULT` clause is specified\). If there is no `DEFAULT` clause, this is merely a metadata change and does not require any immediate update of the table's data; the added NULL values are supplied on readout, instead.

Adding a column with a `DEFAULT` clause or changing the type of an existing column will require the entire table and its indexes to be rewritten. As an exception when changing the type of an existing column, if the `USING` clause does not change the column contents and the old type is either binary coercible to the new type or an unconstrained domain over the new type, a table rewrite is not needed; but any indexes on the affected columns must still be rebuilt. Adding or removing a system `oid` column also requires rewriting the entire table. Table and/or index rebuilds may take a significant amount of time for a large table; and will temporarily require as much as double the disk space.

Adding a `CHECK` or `NOT NULL` constraint requires scanning the table to verify that existing rows meet the constraint, but does not require a table rewrite.

Similarly, when attaching a new partition it may be scanned to verify that existing rows meet the partition constraint.

The main reason for providing the option to specify multiple changes in a single `ALTER TABLE` is that multiple table scans or rewrites can thereby be combined into a single pass over the table.

The `DROP COLUMN` form does not physically remove the column, but simply makes it invisible to SQL operations. Subsequent insert and update operations in the table will store a null value for the column. Thus, dropping a column is quick but it will not immediately reduce the on-disk size of your table, as the space occupied by the dropped column is not reclaimed. The space will be reclaimed over time as existing rows are updated. \(These statements do not apply when dropping the system `oid` column; that is done with an immediate rewrite.\)

To force immediate reclamation of space occupied by a dropped column, you can execute one of the forms of `ALTER TABLE` that performs a rewrite of the whole table. This results in reconstructing each row with the dropped column replaced by a null value.

The rewriting forms of `ALTER TABLE` are not MVCC-safe. After a table rewrite, the table will appear empty to concurrent transactions, if they are using a snapshot taken before the rewrite occurred. See [Section 13.5](https://www.postgresql.org/docs/10/static/mvcc-caveats.html) for more details.

The `USING` option of `SET DATA TYPE` can actually specify any expression involving the old values of the row; that is, it can refer to other columns as well as the one being converted. This allows very general conversions to be done with the `SET DATA TYPE` syntax. Because of this flexibility, the `USING` expression is not applied to the column's default value \(if any\); the result might not be a constant expression as required for a default. This means that when there is no implicit or assignment cast from old to new type, `SET DATA TYPE` might fail to convert the default even though a `USING` clause is supplied. In such cases, drop the default with `DROP DEFAULT`, perform the `ALTER TYPE`, and then use `SET DEFAULT` to add a suitable new default. Similar considerations apply to indexes and constraints involving the column.

If a table has any descendant tables, it is not permitted to add, rename, or change the type of a column in the parent table without doing same to the descendants. This ensures that the descendants always have columns matching the parent. Similarly, a constraint cannot be renamed in the parent without also renaming it in all descendants, so that constraints also match between the parent and its descendants. Also, because selecting from the parent also selects from its descendants, a constraint on the parent cannot be marked valid unless it is also marked valid for those descendants. In all of these cases, `ALTER TABLE ONLY` will be rejected.

A recursive `DROP COLUMN` operation will remove a descendant table's column only if the descendant does not inherit that column from any other parents and never had an independent definition of the column. A nonrecursive `DROP COLUMN` \(i.e., `ALTER TABLE ONLY ... DROP COLUMN`\) never removes any descendant columns, but instead marks them as independently defined rather than inherited. A nonrecursive `DROP COLUMN` command will fail for a partitioned table, because all partitions of a table must have the same columns as the partitioning root.

The actions for identity columns \(`ADD GENERATED`, `SET` etc., `DROP IDENTITY`\), as well as the actions `TRIGGER`, `CLUSTER`, `OWNER`, and `TABLESPACE` never recurse to descendant tables; that is, they always act as though `ONLY` were specified. Adding a constraint recurses only for `CHECK`constraints that are not marked `NO INHERIT`.

Changing any part of a system catalog table is not permitted.

Refer to [CREATE TABLE](https://www.postgresql.org/docs/10/static/sql-createtable.html) for a further description of valid parameters. [Chapter 5](https://www.postgresql.org/docs/10/static/ddl.html) has further information on inheritance.

### 範例

要將一個 varchar 型別的欄位加到資料表中，請執行以下操作指令：

```text
ALTER TABLE distributors ADD COLUMN address varchar(30);
```

從資料表中刪除一個欄位：

```text
ALTER TABLE distributors DROP COLUMN address RESTRICT;
```

在一個操作指令中變更兩個現有欄位的型別：

```text
ALTER TABLE distributors
    ALTER COLUMN address TYPE varchar(80),
    ALTER COLUMN name TYPE varchar(100);
```

透過 USING 子句將包含 Unix 時間戳記的整數欄位變更為帶有時區的時間戳記：

```text
ALTER TABLE foo
    ALTER COLUMN foo_timestamp SET DATA TYPE timestamp with time zone
    USING
        timestamp with time zone 'epoch' + foo_timestamp * interval '1 second';
```

同樣，當有一個欄位沒有自動轉換為新資料型別的預設表示式時：

```text
ALTER TABLE foo
    ALTER COLUMN foo_timestamp DROP DEFAULT,
    ALTER COLUMN foo_timestamp TYPE timestamp with time zone
    USING
        timestamp with time zone 'epoch' + foo_timestamp * interval '1 second',
    ALTER COLUMN foo_timestamp SET DEFAULT now();
```

重新命名現有的欄位：

```text
ALTER TABLE distributors RENAME COLUMN address TO city;
```

重新命名現有的資料表：

```text
ALTER TABLE distributors RENAME TO suppliers;
```

重新命名現有的限制條件：

```text
ALTER TABLE distributors RENAME CONSTRAINT zipchk TO zip_check;
```

要將欄位加上 not null 的限制條件：

```text
ALTER TABLE distributors ALTER COLUMN street SET NOT NULL;
```

從欄位中刪除 not null 的限制條件：

```text
ALTER TABLE distributors ALTER COLUMN street DROP NOT NULL;
```

為資料表及其所有子資料表加上檢查的限制條件：

```text
ALTER TABLE distributors ADD CONSTRAINT zipchk CHECK (char_length(zipcode) = 5);
```

要僅將要檢查的限制條件加到資料表而不加到其子資料表：

```text
ALTER TABLE distributors ADD CONSTRAINT zipchk CHECK (char_length(zipcode) = 5) NO INHERIT;
```

（檢查用的限制條件並不會被未來的子資料表繼承。）

從資料表及其所有子資料表中移除限制條件：

```text
ALTER TABLE distributors DROP CONSTRAINT zipchk;
```

僅從一個資料表中刪除限制條件：

```text
ALTER TABLE ONLY distributors DROP CONSTRAINT zipchk;
```

（限制條件會保留在所有的子資料表中。）

將外部鍵的限制條件加到到資料表中：

```text
ALTER TABLE distributors ADD CONSTRAINT distfk FOREIGN KEY (address) REFERENCES addresses (address);
```

將外部鍵限制條件以其他工作影響最小的方式加到資料表中：

```text
ALTER TABLE distributors ADD CONSTRAINT distfk FOREIGN KEY (address) REFERENCES addresses (address) NOT VALID;
ALTER TABLE distributors VALIDATE CONSTRAINT distfk;
```

在資料表中加上（多個欄位）唯一性的限制條件：

```text
ALTER TABLE distributors ADD CONSTRAINT dist_id_zipcode_key UNIQUE (dist_id, zipcode);
```

要在資料表中加上一個自動命名的主鍵限制條件，注意的是，一個資料表只能有一個主鍵：

```text
ALTER TABLE distributors ADD PRIMARY KEY (dist_id);
```

將資料表移動到不同的資料表空間：

```text
ALTER TABLE distributors SET TABLESPACE fasttablespace;
```

將資料表移動到不同的 schema：

```text
ALTER TABLE myschema.distributors SET SCHEMA yourschema;
```

在重建索引時重新建立主鍵的限制條件，而不阻擋資料更新：

```text
CREATE UNIQUE INDEX CONCURRENTLY dist_id_temp_idx ON distributors (dist_id);
ALTER TABLE distributors DROP CONSTRAINT distributors_pkey,
    ADD CONSTRAINT distributors_pkey PRIMARY KEY USING INDEX dist_id_temp_idx;
```

將資料表分割區附加到範圍型的分割資料表中：

```text
ALTER TABLE measurement
    ATTACH PARTITION measurement_y2016m07 FOR VALUES FROM ('2016-07-01') TO ('2016-08-01');
```

將資料表分割區附加到列表型的分割資料表中：

```text
ALTER TABLE cities
    ATTACH PARTITION cities_ab FOR VALUES IN ('a', 'b');
```

從分割資料表中分離資料表分割區：

```text
ALTER TABLE measurement
    DETACH PARTITION measurement_y2015m12;
```

### 相容性

ADD（沒有 USING INDEX）、DROP \[COLUMN\]、DROP IDENTITY、RESTART、SET DEFAULT、SET DATA TYPE（沒有 USING）、SET GENERATED 和 SET sequence\_option 的語法是符合 SQL 標準的。其他語法則是 SQL 標準的 PostgreSQL 延伸語法。此外，在單個 ALTER TABLE 指令中進行多個操作的功能也是延伸語法。

ALTER TABLE DROP COLUMN 可用於刪除資料表的單一欄位，而留下一個沒有欄位的資料表。這是 SQL 的延伸，SQL 標準禁止使用無欄位的資料表。

### 參閱

[CREATE TABLE](create-table.md)

