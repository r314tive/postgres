EXPLAIN ANALYZE wait events
==========================

This is my PostgreSQL development fork. My current work is
`EXPLAIN (ANALYZE, WAITS)`, tracked in
[CommitFest 6753](https://commitfest.postgresql.org/patch/6753/).

The [v3 branch](https://github.com/r314tive/postgres/tree/r314tive/pg-wait-explain-rfc-v3)
contains the latest patch series posted to pgsql-hackers, on 28 May 2026.
It reports completed wait intervals for a statement and its plan nodes,
including waits from parallel workers. Per-node totals are inclusive, and
the combined wait time of parallel processes can exceed query elapsed time.
The default branch keeps the upstream source; the WAITS code is on the
linked branch.

The current prototype times each reported wait interval. The next step is
to compare sampling approaches, their overhead and the accuracy of plan-node
attribution. Sampling code is not implemented yet.

The design discussion is on
[pgsql-hackers](https://www.postgresql.org/message-id/flat/cover.1778280923.git.tanswis42%40gmail.com).
Earlier benchmark scripts and results are in
[pg-wait-explain-bench](https://github.com/r314tive/pg-wait-explain-bench).
I plan to use
[postgres-experiment-workbench](https://github.com/r314tive/postgres-experiment-workbench)
for repeated workload comparisons, alongside focused tests of the collector.

---

PostgreSQL Database Management System
=====================================

This directory contains the source code distribution of the PostgreSQL
database management system.

PostgreSQL is an advanced object-relational database management system
that supports an extended subset of the SQL standard, including
transactions, foreign keys, subqueries, triggers, user-defined types
and functions.  This distribution also contains C language bindings.

Copyright and license information can be found in the file COPYRIGHT.

General documentation about this version of PostgreSQL can be found at
<https://www.postgresql.org/docs/devel/>.  In particular, information
about building PostgreSQL from the source code can be found at
<https://www.postgresql.org/docs/devel/installation.html>.

The latest version of this software, and related software, may be
obtained at <https://www.postgresql.org/download/>.  For more information
look at our web site located at <https://www.postgresql.org/>.
