# 2.9. 刪除資料[^1]

把整列資料從表格中移除，就使用 DELETE 這個指令。假設你對於 Hayward 這個城市的天氣不再感興趣了，那麼你可以執行下列指令，來刪除表格中的這些資料：

```
DELETE FROM weather WHERE city = 'Hayward';
```

所有關於 Hayward 的資料都被刪除了。

```
SELECT * FROM weather;
```

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      41 |      55 |    0 | 1994-11-29
(2 rows)
```

這個指令有一個應該要特別注意的情況：

```
DELETE FROM tablename;
```

沒有任何限制的條件，DELETE 將會**刪去所有該表格中的資料**，使成為空的表格。資料庫系統並不會在這個動作執行前和你確認！

---

[^1]: [PostgreSQL: Documentation: 10: 2.9. Deletions](https://www.postgresql.org/docs/10/static/tutorial-delete.html)

