# Summary

* [簡介](README.md)
* [前言](preface.md)
  * [1. 什麼是PostgreSQL？](what-is-postgresql.md)
  * [2. PostgreSQL沿革](a-brief-history-of-postgresql.md)
  * [3. 慣例](conventions.md)
  * [4. 其他參考資訊](further-information.md)
  * [5. 問題回報指南](bug-reporting-guidelines.md)
* [I. 新手教學](i-tutorial.md)
  * [1. 入門指南](getting-started.md)
    * [1.1. 安裝](getting-started/11-installation.md)
    * [1.2. 基礎架構](getting-started/12-architectural-fundamentals.md)
    * [1.3. 建立一個資料庫](getting-started/13-creating-a-database.md)
    * [1.4. 存取一個資料庫](getting-started/14-accessing-a-database.md)
  * [2. SQL查詢語言](the-sql-language.md)
    * [2.1. 簡介](the-sql-language/21-introduction.md)
    * [2.2. 概念](the-sql-language/22-concepts.md)
    * [2.3. 創建一個新的表格](the-sql-language/23-creating-a-new-table.md)
    * [2.4. 列是表格的組成單位](the-sql-language/24-populating-a-table-with-rows.md)
    * [2.5. 表格的查詢](the-sql-language/25-querying-a-table.md)
    * [2.6. 交叉查詢](the-sql-language/26-joins-between-tables.md)
    * [2.7. 彙總查詢](the-sql-language/27-aggregate-functions.md)
    * [2.8. 更新資料](the-sql-language/28-updates.md)
    * [2.9. 刪除資料](the-sql-language/29-deletions.md)
  * [3. 先進功能](advanced-features.md)
    * [3.1. 簡介](advanced-features/31-introduction.md)
    * [3.2. Views](advanced-features/32-views.md)
    * [3.3. Foreign Keys](advanced-features/33-foreign-keys.md)
    * [3.4. 交易安全](advanced-features/34-transactions.md)
    * [3.5. Window Functions](advanced-features/35-window-functions.md)
    * [3.6. 繼承](advanced-features/36-inheritance.md)
    * [3.7. 結論](advanced-features/37-conclusion.md)
* [II. SQL查詢語言](ii-the-sql-language.md)
  * [4. SQL語法](ii-the-sql-language/sql-syntax.md)
    * [4.1. 語法結構](ii-the-sql-language/sql-syntax/41-lexical-structure.md)
    * [4.2. 參數表示式](ii-the-sql-language/sql-syntax/42-value-expressions.md)
    * [4.3. 函式呼叫](ii-the-sql-language/sql-syntax/43-calling-functions.md)
  * [5. 定義資料結構](ii-the-sql-language/data-definition.md)
    * [5.1. 認識表格](ii-the-sql-language/data-definition/51-table-basics.md)
    * [5.2. 預設值](ii-the-sql-language/data-definition/52-default-values.md)
    * [5.3. 限制條件](ii-the-sql-language/data-definition/53-constraints.md)
    * [5.4. 系統欄位](ii-the-sql-language/data-definition/54-system-columns.md)
    * [5.5. 表格變更](ii-the-sql-language/data-definition/55-modifying-tables.md)
    * [5.6. 權限](ii-the-sql-language/data-definition/56-privileges.md)
    * [5.7. 安全原則](ii-the-sql-language/data-definition/57-row-security-policies.md)
    * [5.8. Schemas](ii-the-sql-language/data-definition/58-schemas.md)
    * [5.9. 繼承](ii-the-sql-language/data-definition/59-inheritance.md)
    * [5.10. 分割表格](ii-the-sql-language/data-definition/510-table-partitioning.md)
    * [5.11. 外部資料](ii-the-sql-language/data-definition/511-foreign-data.md)
    * [5.12. 其他資料庫物件](ii-the-sql-language/data-definition/512-other-database-objects.md)
    * [5.13. 相依性追蹤](ii-the-sql-language/data-definition/513-dependency-tracking.md)
  * [6. 資料處理](ii-the-sql-language/data-manipulation.md)
    * [6.1. 新增資料](ii-the-sql-language/data-manipulation/61-inserting-data.md)
    * [6.2. 更新資料](ii-the-sql-language/data-manipulation/62-updating-data.md)
    * [6.3. 刪除資料](ii-the-sql-language/data-manipulation/63-deleting-data.md)
    * [6.4. 修改並回傳資料](ii-the-sql-language/data-manipulation/64-returning-data-from-modified-rows.md)
  * [7. 資料查詢](ii-the-sql-language/queries.md)
    * [7.1. 概觀](ii-the-sql-language/queries/71-overview.md)
    * [7.2. 表格表示式](ii-the-sql-language/queries/72-table-expressions.md)
    * [7.3. 取得資料列表](ii-the-sql-language/queries/73-select-lists.md)
    * [7.4. 合併查詢結果](ii-the-sql-language/queries/74-combining-queries.md)
    * [7.5. 資料排序](ii-the-sql-language/queries/75-sorting-rows.md)
    * [7.6. 指定資料範圍](ii-the-sql-language/queries/76-limit-and-offset.md)
    * [7.7. 列舉資料](ii-the-sql-language/queries/77-values-lists.md)
    * [7.8. 遞迴查詢](ii-the-sql-language/queries/78-with-queries-common-table-expressions.md)
  * [8. 資料型別](ii-the-sql-language/data-types.md)
    * [8.1. 數字型別](ii-the-sql-language/data-types/81-numeric-types.md)
    * [8.2. 貨幣型別](ii-the-sql-language/data-types/82-monetary-types.md)
    * [8.3. 文字型別](ii-the-sql-language/data-types/83-character-types.md)
    * [8.4. 位元組型別](ii-the-sql-language/data-types/84-binary-data-types.md)
    * [8.5. 日期時間型別](ii-the-sql-language/data-types/85-datetime-types.md)
    * [8.6. 布林型別](ii-the-sql-language/data-types/86-boolean-type.md)
    * [8.7. 列舉型別](ii-the-sql-language/data-types/87-enumerated-types.md)
    * [8.8. 地理資訊型別](ii-the-sql-language/data-types/88-geometric-types.md)
    * [8.9. 網路資訊型別](ii-the-sql-language/data-types/89-network-address-types.md)
    * [8.10. 位元字串型別](ii-the-sql-language/data-types/810-bit-string-types.md)
    * [8.11. 全文檢索型別](ii-the-sql-language/data-types/811-text-search-types.md)
    * [8.12. UUID型別](ii-the-sql-language/data-types/812-uuid-type.md)
    * [8.13. XML型別](ii-the-sql-language/data-types/813-xml-type.md)
    * [8.14. JSON型別](ii-the-sql-language/data-types/814-json-types.md)
    * [8.15. 陣列](ii-the-sql-language/data-types/815-arrays.md)
    * [8.16. 複合型別](ii-the-sql-language/data-types/816-composite-types.md)
    * [8.17. 範圍型別](ii-the-sql-language/data-types/817-range-types.md)
    * [8.18. 指標型別](ii-the-sql-language/data-types/818-object-identifier-types.md)
    * [8.19. pg\_lsn型別](ii-the-sql-language/data-types/819-pglsn-type.md)
    * [8.20. 概念型別](ii-the-sql-language/data-types/820-pseudo-types.md)
  * [9. 函式及運算子](ii-the-sql-language/functions-and-operators.md)
    * [9.1. 邏輯運算子](ii-the-sql-language/functions-and-operators/91-logical-operators.md)
    * [9.2. 比較函式及運算子](ii-the-sql-language/functions-and-operators/92-comparison-functions-and-operators.md)
    * [9.3. 數學函式及運算子](ii-the-sql-language/functions-and-operators/93-mathematical-functions-and-operators.md)
    * [9.4. 字串函式及運算子](ii-the-sql-language/functions-and-operators/94-string-functions-and-operators.md)
    * [9.5. 位元字串函式及運算子](ii-the-sql-language/functions-and-operators/95-binary-string-functions-and-operators.md)
    * [9.6. 二元字串函式及運算子](ii-the-sql-language/functions-and-operators/96-bit-string-functions-and-operators.md)
    * [9.7. 樣式比對](ii-the-sql-language/functions-and-operators/97-pattern-matching.md)
    * [9.8. 型別轉換函式](ii-the-sql-language/functions-and-operators/98-data-type-formatting-functions.md)
    * [9.9 日期時間函式及運算子](ii-the-sql-language/functions-and-operators/99-datetime-functions-and-operators.md)
    * [9.10. 列舉型別函式](ii-the-sql-language/functions-and-operators/910-enum-support-functions.md)
    * [9.11. 地理資訊函式及運算子](ii-the-sql-language/functions-and-operators/911-geometric-functions-and-operators.md)
    * [9.12. 網路位址函式及運算子](ii-the-sql-language/functions-and-operators/912-network-address-functions-and-operators.md)
    * [9.13. 文字檢索函式及運算子](ii-the-sql-language/functions-and-operators/913-text-search-functions-and-operators.md)
    * [9.14. XML函式](ii-the-sql-language/functions-and-operators/914-xml-functions.md)
    * [9.15. JSON函式及運算子](ii-the-sql-language/functions-and-operators/915-json-functions-and-operators.md)
    * [9.16. 序列函式](ii-the-sql-language/functions-and-operators/916-sequence-manipulation-functions.md)
    * [9.17. 條件表示式](ii-the-sql-language/functions-and-operators/917-conditional-expressions.md)
    * [9.18. 陣列函式及運算子](ii-the-sql-language/functions-and-operators/918-array-functions-and-operators.md)
    * [9.19. 範圍函式及運算子](ii-the-sql-language/functions-and-operators/919-range-functions-and-operators.md)
    * [9.20. 彙總函式](ii-the-sql-language/functions-and-operators/920-aggregate-functions.md)
    * [9.21. Window函式](ii-the-sql-language/functions-and-operators/921-window-functions.md)
    * [9.22. 子查詢](ii-the-sql-language/functions-and-operators/922-subquery-expressions.md)
    * [9.23. Row and Array Comparisons](ii-the-sql-language/functions-and-operators/923-row-and-array-comparisons.md)
    * 9.24. Set Returning Functions
    * 9.25. System Information Functions
    * 9.26. System Administration Functions
    * 9.27. Trigger Functions
    * 9.28. Event Trigger Functions
  * Type Conversion
    * 10.1. Overview
    * 10.2. Operators
    * 10.3. Functions
    * 10.4. Value Storage
    * 10.5. UNION, CASE, and Related Constructs
    * 10.6. SELECT Output Columns
  * Indexes
    * 11.1. Introduction
    * 11.2. Index Types
    * 11.3. Multicolumn Indexes
    * 11.4. Indexes and ORDER BY
    * 11.5. Combining Multiple Indexes
    * 11.6. Unique Indexes
    * 11.7. Indexes on Expressions
    * 11.8. Partial Indexes
    * 11.9. Operator Classes and Operator Families
    * 11.10. Indexes and Collations
    * 11.11. Index-Only Scans
    * 11.12. Examining Index Usage
  * Full Text Search
    * 12.1. Introduction
    * 12.2. Tables and Indexes
    * 12.3. Controlling Text Search
    * 12.4. Additional Features
    * 12.5. Parsers
    * 12.6. Dictionaries
    * 12.7. Configuration Example
    * 12.8. Testing and Debugging Text Search
    * 12.9. GIN and GiST Index Types
    * 12.10. psql Support
    * 12.11. Limitations
  * Concurrency Control
    * 13.1. Introduction
    * 13.2. Transaction Isolation
    * 13.3. Explicit Locking
    * 13.4. Data Consistency Checks at the Application Level
    * 13.5. Caveats
    * 13.6. Locking and Indexes
  * Performance Tips
    * 14.1. Using EXPLAIN
    * 14.2. Statistics Used by the Planner
    * 14.3. Controlling the Planner with Explicit JOIN Clauses
    * 14.4. Populating a Database
    * 14.5. Non-Durable Settings
  * Parallel Query
    * 15.1. How Parallel Query Works
    * 15.2. When Can Parallel Query Be Used?
    * 15.3. Parallel Plans
    * 15.4. Parallel Safety
* [III. Server Administration](iii-server-administration.md)
* [IV. Client Interfaces](iv-client-interfaces.md)
* [V. Server Programming](v-server-programming.md)
* [VI. Reference](vi-reference.md)
  * I. SQL Commands
  * II. PostgreSQL Client Applications
  * III. PostgreSQL Server Applications
* [VII. Internals](vii-internals.md)
* [VIII. 附錄](viii-appendixes.md)
  * [A. PostgreSQL錯誤代碼](viii-appendixes/postgresql-error-codes.md)
  * [B. 日期時間格式支援](viii-appendixes/datetime-support.md)
    * [B.1. 日期時間解譯流程](viii-appendixes/datetime-support/b1-datetime-input-interpretation.md)
    * [B.2. 日期時間慣用字](viii-appendixes/datetime-support/b2-datetime-key-words.md)
    * [B.3. 日期時間設定檔](viii-appendixes/datetime-support/b3-datetime-configuration-files.md)
    * [B.4. 日期時間的沿革](viii-appendixes/datetime-support/b4-history-of-units.md)
  * SQL Key Words
  * SQL Conformance
  * Release Notes
  * Additional Supplied Modules
  * Additional Supplied Programs
  * External Projects
  * The Source Code Repository
  * [J. 文件取得](viii-appendixes/documentation.md)
  * [K. 縮寫字](viii-appendixes/acronyms.md)
* [參考書目](bibliography.md)

