# 19.7. 查詢規畫

#### 19.7.1. Planner Method Configuration

These configuration parameters provide a crude method of influencing the query plans chosen by the query optimizer. If the default plan chosen by the optimizer for a particular query is not optimal, a _temporary_ solution is to use one of these configuration parameters to force the optimizer to choose a different plan. Better ways to improve the quality of the plans chosen by the optimizer include adjusting the planner cost constants \(see [Section 19.7.2](https://www.postgresql.org/docs/10/static/runtime-config-query.html#RUNTIME-CONFIG-QUERY-CONSTANTS)\), running [ANALYZE](https://www.postgresql.org/docs/10/static/sql-analyze.html) manually, increasing the value of the [default\_statistics\_target](https://www.postgresql.org/docs/10/static/runtime-config-query.html#GUC-DEFAULT-STATISTICS-TARGET)configuration parameter, and increasing the amount of statistics collected for specific columns using `ALTER TABLE SET STATISTICS`.`enable_bitmapscan` \(`boolean`\)

Enables or disables the query planner's use of bitmap-scan plan types. The default is `on`.`enable_gathermerge` \(`boolean`\)

Enables or disables the query planner's use of gather merge plan types. The default is `on`.`enable_hashagg` \(`boolean`\)

Enables or disables the query planner's use of hashed aggregation plan types. The default is `on`.`enable_hashjoin` \(`boolean`\)

Enables or disables the query planner's use of hash-join plan types. The default is `on`.`enable_indexscan` \(`boolean`\)

Enables or disables the query planner's use of index-scan plan types. The default is `on`.`enable_indexonlyscan` \(`boolean`\)

Enables or disables the query planner's use of index-only-scan plan types \(see [Section 11.11](https://www.postgresql.org/docs/10/static/indexes-index-only-scans.html)\). The default is `on`.`enable_material` \(`boolean`\)

Enables or disables the query planner's use of materialization. It is impossible to suppress materialization entirely, but turning this variable off prevents the planner from inserting materialize nodes except in cases where it is required for correctness. The default is `on`.`enable_mergejoin` \(`boolean`\)

Enables or disables the query planner's use of merge-join plan types. The default is `on`.`enable_nestloop` \(`boolean`\)

Enables or disables the query planner's use of nested-loop join plans. It is impossible to suppress nested-loop joins entirely, but turning this variable off discourages the planner from using one if there are other methods available. The default is `on`.`enable_seqscan` \(`boolean`\)

Enables or disables the query planner's use of sequential scan plan types. It is impossible to suppress sequential scans entirely, but turning this variable off discourages the planner from using one if there are other methods available. The default is `on`.`enable_sort` \(`boolean`\)

Enables or disables the query planner's use of explicit sort steps. It is impossible to suppress explicit sorts entirely, but turning this variable off discourages the planner from using one if there are other methods available. The default is `on`.`enable_tidscan` \(`boolean`\)

Enables or disables the query planner's use of TID scan plan types. The default is `on`.

#### 19.7.2. Planner Cost Constants

The _cost_ variables described in this section are measured on an arbitrary scale. Only their relative values matter, hence scaling them all up or down by the same factor will result in no change in the planner's choices. By default, these cost variables are based on the cost of sequential page fetches; that is, `seq_page_cost` is conventionally set to `1.0` and the other cost variables are set with reference to that. But you can use a different scale if you prefer, such as actual execution times in milliseconds on a particular machine.

#### Note

Unfortunately, there is no well-defined method for determining ideal values for the cost variables. They are best treated as averages over the entire mix of queries that a particular installation will receive. This means that changing them on the basis of just a few experiments is very risky.`seq_page_cost` \(`floating point`\)

Sets the planner's estimate of the cost of a disk page fetch that is part of a series of sequential fetches. The default is 1.0. This value can be overridden for tables and indexes in a particular tablespace by setting the tablespace parameter of the same name \(see [ALTER TABLESPACE](https://www.postgresql.org/docs/10/static/sql-altertablespace.html)\).`random_page_cost` \(`floating point`\)

Sets the planner's estimate of the cost of a non-sequentially-fetched disk page. The default is 4.0. This value can be overridden for tables and indexes in a particular tablespace by setting the tablespace parameter of the same name \(see [ALTER TABLESPACE](https://www.postgresql.org/docs/10/static/sql-altertablespace.html)\).

Reducing this value relative to `seq_page_cost` will cause the system to prefer index scans; raising it will make index scans look relatively more expensive. You can raise or lower both values together to change the importance of disk I/O costs relative to CPU costs, which are described by the following parameters.

Random access to mechanical disk storage is normally much more expensive than four times sequential access. However, a lower default is used \(4.0\) because the majority of random accesses to disk, such as indexed reads, are assumed to be in cache. The default value can be thought of as modeling random access as 40 times slower than sequential, while expecting 90% of random reads to be cached.

If you believe a 90% cache rate is an incorrect assumption for your workload, you can increase random\_page\_cost to better reflect the true cost of random storage reads. Correspondingly, if your data is likely to be completely in cache, such as when the database is smaller than the total server memory, decreasing random\_page\_cost can be appropriate. Storage that has a low random read cost relative to sequential, e.g. solid-state drives, might also be better modeled with a lower value for random\_page\_cost.

#### Tip

Although the system will let you set `random_page_cost` to less than `seq_page_cost`, it is not physically sensible to do so. However, setting them equal makes sense if the database is entirely cached in RAM, since in that case there is no penalty for touching pages out of sequence. Also, in a heavily-cached database you should lower both values relative to the CPU parameters, since the cost of fetching a page already in RAM is much smaller than it would normally be.`cpu_tuple_cost` \(`floating point`\)

Sets the planner's estimate of the cost of processing each row during a query. The default is 0.01.`cpu_index_tuple_cost` \(`floating point`\)

Sets the planner's estimate of the cost of processing each index entry during an index scan. The default is 0.005.`cpu_operator_cost` \(`floating point`\)

Sets the planner's estimate of the cost of processing each operator or function executed during a query. The default is 0.0025.`parallel_setup_cost` \(`floating point`\)

Sets the planner's estimate of the cost of launching parallel worker processes. The default is 1000.`parallel_tuple_cost` \(`floating point`\)

Sets the planner's estimate of the cost of transferring one tuple from a parallel worker process to another process. The default is 0.1.`min_parallel_table_scan_size` \(`integer`\)

Sets the minimum amount of table data that must be scanned in order for a parallel scan to be considered. For a parallel sequential scan, the amount of table data scanned is always equal to the size of the table, but when indexes are used the amount of table data scanned will normally be less. The default is 8 megabytes \(`8MB`\).`min_parallel_index_scan_size` \(`integer`\)

Sets the minimum amount of index data that must be scanned in order for a parallel scan to be considered. Note that a parallel index scan typically won't touch the entire index; it is the number of pages which the planner believes will actually be touched by the scan which is relevant. The default is 512 kilobytes \(`512kB`\).`effective_cache_size` \(`integer`\)

Sets the planner's assumption about the effective size of the disk cache that is available to a single query. This is factored into estimates of the cost of using an index; a higher value makes it more likely index scans will be used, a lower value makes it more likely sequential scans will be used. When setting this parameter you should consider both PostgreSQL's shared buffers and the portion of the kernel's disk cache that will be used for PostgreSQL data files. Also, take into account the expected number of concurrent queries on different tables, since they will have to share the available space. This parameter has no effect on the size of shared memory allocated by PostgreSQL, nor does it reserve kernel disk cache; it is used only for estimation purposes. The system also does not assume data remains in the disk cache between queries. The default is 4 gigabytes \(`4GB`\).

#### 19.7.3. Genetic Query Optimizer

The genetic query optimizer \(GEQO\) is an algorithm that does query planning using heuristic searching. This reduces planning time for complex queries \(those joining many relations\), at the cost of producing plans that are sometimes inferior to those found by the normal exhaustive-search algorithm. For more information see [Chapter 59](https://www.postgresql.org/docs/10/static/geqo.html).`geqo` \(`boolean`\)

Enables or disables genetic query optimization. This is on by default. It is usually best not to turn it off in production; the `geqo_threshold` variable provides more granular control of GEQO.`geqo_threshold` \(`integer`\)

Use genetic query optimization to plan queries with at least this many `FROM` items involved. \(Note that a `FULL OUTER JOIN` construct counts as only one `FROM` item.\) The default is 12. For simpler queries it is usually best to use the regular, exhaustive-search planner, but for queries with many tables the exhaustive search takes too long, often longer than the penalty of executing a suboptimal plan. Thus, a threshold on the size of the query is a convenient way to manage use of GEQO.`geqo_effort` \(`integer`\)

Controls the trade-off between planning time and query plan quality in GEQO. This variable must be an integer in the range from 1 to 10. The default value is five. Larger values increase the time spent doing query planning, but also increase the likelihood that an efficient query plan will be chosen.

`geqo_effort` doesn't actually do anything directly; it is only used to compute the default values for the other variables that influence GEQO behavior \(described below\). If you prefer, you can set the other parameters by hand instead.`geqo_pool_size` \(`integer`\)

Controls the pool size used by GEQO, that is the number of individuals in the genetic population. It must be at least two, and useful values are typically 100 to 1000. If it is set to zero \(the default setting\) then a suitable value is chosen based on `geqo_effort` and the number of tables in the query.`geqo_generations` \(`integer`\)

Controls the number of generations used by GEQO, that is the number of iterations of the algorithm. It must be at least one, and useful values are in the same range as the pool size. If it is set to zero \(the default setting\) then a suitable value is chosen based on `geqo_pool_size`.`geqo_selection_bias` \(`floating point`\)

Controls the selection bias used by GEQO. The selection bias is the selective pressure within the population. Values can be from 1.50 to 2.00; the latter is the default.`geqo_seed` \(`floating point`\)

Controls the initial value of the random number generator used by GEQO to select random paths through the join order search space. The value can range from zero \(the default\) to one. Varying the value changes the set of join paths explored, and may result in a better or worse best path being found.

#### 19.7.4. Other Planner Options

`default_statistics_target` \(`integer`\)

Sets the default statistics target for table columns without a column-specific target set via `ALTER TABLE SET STATISTICS`. Larger values increase the time needed to do `ANALYZE`, but might improve the quality of the planner's estimates. The default is 100. For more information on the use of statistics by the PostgreSQL query planner, refer to [Section 14.2](https://www.postgresql.org/docs/10/static/planner-stats.html).`constraint_exclusion` \(`enum`\)

Controls the query planner's use of table constraints to optimize queries. The allowed values of `constraint_exclusion` are `on` \(examine constraints for all tables\), `off` \(never examine constraints\), and `partition` \(examine constraints only for inheritance child tables and `UNION ALL` subqueries\). `partition` is the default setting. It is often used with inheritance and partitioned tables to improve performance.

When this parameter allows it for a particular table, the planner compares query conditions with the table's `CHECK` constraints, and omits scanning tables for which the conditions contradict the constraints. For example:

```text
CREATE TABLE parent(key integer, ...);
CREATE TABLE child1000(check (key between 1000 and 1999)) INHERITS(parent);
CREATE TABLE child2000(check (key between 2000 and 2999)) INHERITS(parent);
...
SELECT * FROM parent WHERE key = 2400;
```

With constraint exclusion enabled, this `SELECT` will not scan `child1000` at all, improving performance.

Currently, constraint exclusion is enabled by default only for cases that are often used to implement table partitioning. Turning it on for all tables imposes extra planning overhead that is quite noticeable on simple queries, and most often will yield no benefit for simple queries. If you have no partitioned tables you might prefer to turn it off entirely.

Refer to [Section 5.10.4](https://www.postgresql.org/docs/10/static/ddl-partitioning.html#DDL-PARTITIONING-CONSTRAINT-EXCLUSION) for more information on using constraint exclusion and partitioning.`cursor_tuple_fraction` \(`floating point`\)

Sets the planner's estimate of the fraction of a cursor's rows that will be retrieved. The default is 0.1. Smaller values of this setting bias the planner towards using “fast start” plans for cursors, which will retrieve the first few rows quickly while perhaps taking a long time to fetch all rows. Larger values put more emphasis on the total estimated time. At the maximum setting of 1.0, cursors are planned exactly like regular queries, considering only the total estimated time and not how soon the first rows might be delivered.`from_collapse_limit` \(`integer`\)

The planner will merge sub-queries into upper queries if the resulting `FROM` list would have no more than this many items. Smaller values reduce planning time but might yield inferior query plans. The default is eight. For more information see [Section 14.3](https://www.postgresql.org/docs/10/static/explicit-joins.html).

Setting this value to [geqo\_threshold](https://www.postgresql.org/docs/10/static/runtime-config-query.html#GUC-GEQO-THRESHOLD) or more may trigger use of the GEQO planner, resulting in non-optimal plans. See [Section 19.7.3](https://www.postgresql.org/docs/10/static/runtime-config-query.html#RUNTIME-CONFIG-QUERY-GEQO).`join_collapse_limit` \(`integer`\)

The planner will rewrite explicit `JOIN` constructs \(except `FULL JOIN`s\) into lists of `FROM` items whenever a list of no more than this many items would result. Smaller values reduce planning time but might yield inferior query plans.

By default, this variable is set the same as `from_collapse_limit`, which is appropriate for most uses. Setting it to 1 prevents any reordering of explicit `JOIN`s. Thus, the explicit join order specified in the query will be the actual order in which the relations are joined. Because the query planner does not always choose the optimal join order, advanced users can elect to temporarily set this variable to 1, and then specify the join order they desire explicitly. For more information see [Section 14.3](https://www.postgresql.org/docs/10/static/explicit-joins.html).

Setting this value to [geqo\_threshold](https://www.postgresql.org/docs/10/static/runtime-config-query.html#GUC-GEQO-THRESHOLD) or more may trigger use of the GEQO planner, resulting in non-optimal plans. See [Section 19.7.3](https://www.postgresql.org/docs/10/static/runtime-config-query.html#RUNTIME-CONFIG-QUERY-GEQO).`force_parallel_mode` \(`enum`\)

Allows the use of parallel queries for testing purposes even in cases where no performance benefit is expected. The allowed values of `force_parallel_mode` are `off` \(use parallel mode only when it is expected to improve performance\), `on` \(force parallel query for all queries for which it is thought to be safe\), and `regress` \(like `on`, but with additional behavior changes as explained below\).

More specifically, setting this value to `on` will add a `Gather` node to the top of any query plan for which this appears to be safe, so that the query runs inside of a parallel worker. Even when a parallel worker is not available or cannot be used, operations such as starting a subtransaction that would be prohibited in a parallel query context will be prohibited unless the planner believes that this will cause the query to fail. If failures or unexpected results occur when this option is set, some functions used by the query may need to be marked `PARALLEL UNSAFE` \(or, possibly, `PARALLEL RESTRICTED`\).

Setting this value to `regress` has all of the same effects as setting it to `on` plus some additional effects that are intended to facilitate automated regression testing. Normally, messages from a parallel worker include a context line indicating that, but a setting of `regress`suppresses this line so that the output is the same as in non-parallel execution. Also, the `Gather` nodes added to plans by this setting are hidden in `EXPLAIN` output so that the output matches what would be obtained if this setting were turned `off`.

