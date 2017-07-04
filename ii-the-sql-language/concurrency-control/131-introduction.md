# 13.1. 簡介[^1]

PostgreSQLprovides a rich set of tools for developers to manage concurrent access to data. Internally, data consistency is maintained by using a multiversion model \(Multiversion Concurrency Control,MVCC\). This means that each SQL statement sees a snapshot of data \(a_database version_\) as it was some time ago, regardless of the current state of the underlying data. This prevents statements from viewing inconsistent data produced by concurrent transactions performing updates on the same data rows, providing_transaction isolation_for each database session.MVCC, by eschewing the locking methodologies of traditional database systems, minimizes lock contention in order to allow for reasonable performance in multiuser environments.

The main advantage of using theMVCCmodel of concurrency control rather than locking is that inMVCClocks acquired for querying \(reading\) data do not conflict with locks acquired for writing data, and so reading never blocks writing and writing never blocks reading.PostgreSQLmaintains this guarantee even when providing the strictest level of transaction isolation through the use of an innovative_Serializable Snapshot Isolation_\(SSI\) level.

Table- and row-level locking facilities are also available inPostgreSQLfor applications which don't generally need full transaction isolation and prefer to explicitly manage particular points of conflict. However, proper use ofMVCCwill generally provide better performance than locks. In addition, application-defined advisory locks provide a mechanism for acquiring locks that are not tied to a single transaction.

---



[^1]:  [PostgreSQL: Documentation: 10: 13.1. Introduction](https://www.postgresql.org/docs/10/static/mvcc-intro.html)

