# 28. 監控資料庫活動

A database administrator frequently wonders, “What is the system doing right now?” This chapter discusses how to find that out.

Several tools are available for monitoring database activity and analyzing performance. Most of this chapter is devoted to describing PostgreSQL's statistics collector, but one should not neglect regular Unix monitoring programs such as `ps`, `top`, `iostat`, and `vmstat`. Also, once one has identified a poorly-performing query, further investigation might be needed using PostgreSQL's [EXPLAIN](https://www.postgresql.org/docs/10/static/sql-explain.html) command. [Section 14.1](https://www.postgresql.org/docs/10/static/using-explain.html) discusses `EXPLAIN` and other methods for understanding the behavior of an individual query.

