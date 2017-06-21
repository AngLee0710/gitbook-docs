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
    * 8.1. Numeric Types
    * 8.2. Monetary Types
    * 8.3. Character Types
    * 8.4. Binary Data Types
    * 8.5. Date/Time Types
    * 8.6. Boolean Type
    * 8.7. Enumerated Types
    * 8.8. Geometric Types
    * 8.9. Network Address Types
    * 8.10. Bit String Types
    * 8.11. Text Search Types
    * 8.12. UUID Type
    * 8.13. XML Type
    * 8.14. JSON Types
    * 8.15. Arrays
    * 8.16. Composite Types
    * 8.17. Range Types
    * 8.18. Object Identifier Types
    * 8.19. pg\_lsn Type
    * 8.20. Pseudo-Types
  * Functions and Operators
    * 9.1. Logical Operators
    * 9.2. Comparison Functions and Operators
    * 9.3. Mathematical Functions and Operators
    * 9.4. String Functions and Operators
    * 9.5. Binary String Functions and Operators
    * 9.6. Bit String Functions and Operators
    * 9.7. Pattern Matching
    * 9.8. Data Type Formatting Functions
    * 9.9. Date/Time Functions and Operators
    * 9.10. Enum Support Functions
    * 9.11. Geometric Functions and Operators
    * 9.12. Network Address Functions and Operators
    * 9.13. Text Search Functions and Operators
    * 9.14. XML Functions
    * 9.15. JSON Functions and Operators
    * 9.16. Sequence Manipulation Functions
    * 9.17. Conditional Expressions
    * 9.18. Array Functions and Operators
    * 9.19. Range Functions and Operators
    * 9.20. Aggregate Functions
    * 9.21. Window Functions
    * 9.22. Subquery Expressions
    * 9.23. Row and Array Comparisons
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

