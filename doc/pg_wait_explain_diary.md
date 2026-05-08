# EXPLAIN WAITS Patch Diary

Branch-local research notes for `r314tive/pg-wait-explain-mvp`.
This file records non-obvious observations, assumptions, and test scaffolding.
It is not intended to be submitted upstream as-is.

## 2026-05-07

### Wait event identity

The patch must not hardcode native wait event type/name lists.  Runtime
accounting stores `wait_event_info`; display resolves type/name through
`pgstat_get_wait_event_type()` and `pgstat_get_wait_event()`.  This keeps the
feature aligned with core-generated wait event metadata and also preserves
dynamic custom wait event names.

### Query-level vs per-node accounting

Query-level `EXPLAIN (ANALYZE, WAITS)` accounting records each completed wait
once per backend or parallel worker and then aggregates workers into the
leader.  This is the non-duplicated total for the query.

Per-node accounting follows the active executor instrumentation stack.  A wait
is added to every active plan node, so the per-node view is inclusive like
`EXPLAIN ANALYZE` node timing.  Therefore per-node wait times are not additive:
their sum can exceed the query-level wait total.

### Critical-section allocation

Regression testing through a Bitmap Index Scan exposed that an I/O wait can end
inside a critical section.  The wait usage accumulator can need to grow on that
first observed event, so its entries live in a dedicated memory context marked
with `MemoryContextAllowInCriticalSection()`.  This is required for correctness;
it is not an event-specific exception.

### Bitmap regression scaffolding

The Bitmap Index Scan regression case uses test-only scaffolding:

- `enable_seqscan = off` and `enable_indexscan = off` force a deterministic
  Bitmap Heap Scan / Bitmap Index Scan plan shape.
- The `STABLE` PL/pgSQL wrapper around `pg_sleep()` lets the expression be used
  as an index runtime key while preventing SQL inlining from moving the wait
  outside the Bitmap Index Scan boundary.

These choices are regression-test constraints only.  The runtime feature does
not depend on those planner settings, function volatility, or a hardcoded wait
event name.

### Current nested collection boundary

Nested top-level wait collection is still treated as part of the outer
collection.  The stack introduced here is the active plan-node stack for
per-node attribution, not an independent stack of query-level collection
contexts.

## 2026-05-09

### Attribution point

Per-node wait attribution uses the active plan-node stack captured at wait
start, not whatever stack happens to be active at wait end.  This makes the
measured interval's ownership explicit.  In normal PostgreSQL wait reporting
the start and end calls bracket a blocking operation without executor stack
movement, but using the start stack is the stricter model and avoids depending
on that practical property.

### Accumulator lookup

`WaitEventUsage` uses a sorted vector keyed by `wait_event_info`.  Completed
waits use binary search to find the entry; only first observation of a distinct
event requires insertion and possible array growth.  This avoids a linear scan
on the hot wait-end path without introducing a hash table into code that can
run inside critical sections.  The remaining insertion cost is proportional to
the number of distinct wait events already seen by that query or plan node.

### User-facing documentation

The `EXPLAIN` reference documentation promises three key semantics for
`WAITS`: the top-level wait summary is the non-duplicated statement total,
parallel worker waits are included in that total, and per-node wait events are
inclusive like `EXPLAIN ANALYZE` node timing.  It also documents that wait
timing is collected even with `TIMING OFF`, because `TIMING` controls plan-node
execution timing rather than wait-event interval accounting.
