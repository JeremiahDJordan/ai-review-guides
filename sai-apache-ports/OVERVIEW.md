# Why this work: what DataStax main-5.0's SAI was missing

*Read it as a web page: <https://jeremiahdjordan.github.io/ai-review-guides/sai-apache-ports/overview.html>*

An audit compared every Apache `cassandra-5.0` and `trunk` SAI commit with DataStax main-5.0 by behaviour, and filed
every bug it found. A series of 107 commits restores the missing Apache fixes that apply to main-5.0 and fixes
DataStax bugs in the same code. The [review guide](https://jeremiahdjordan.github.io/ai-review-guides/sai-apache-ports/) says how to review it; the [audit report](https://jeremiahdjordan.github.io/ai-review-guides/sai-apache-ports/audit-report.html)
has the details.

The 220 Apache SAI commits: 34 restored by the series, 120 already present or equivalent, 57 not applicable, 4 documented deviations, 5 deferred.

## What was missing

| Area | Before the series | Fixed by |
|---|---|---|
| Upgrades and imports from Apache Cassandra 5.0 | SAI files written by Apache Cassandra 5.0 were read as DSE 6.8 files: `CREATE INDEX` on a vector column and imports into a table with a vector index failed, and other indexes were rebuilt from the data | [`5e60c1c499a`](https://github.com/JeremiahDJordan/cassandra/commit/5e60c1c499afecd8de0fbd3eddb16f72e7b7891f), a reader of Apache's format |
| Clusters with Apache Cassandra 5.0 nodes | Filters that only DataStax nodes understand were sent to Apache nodes, which read `OR` as `AND` and map entry ranges as equalities: wrong results | [`59001b61e92`](https://github.com/JeremiahDJordan/cassandra/commit/59001b61e92050ec884b5ecadf05ea2e13f0f330), which refuses such queries while they can reach those nodes |
| Writes split across replicas | An SAI `AND` missed a row that matches only once the replicas' responses are merged (CASSANDRA-19018) | [`017d9e35c59`](https://github.com/JeremiahDJordan/cassandra/commit/017d9e35c5935b7096e6dfca6819a2ba3887a546), with non-strict replica filtering enabled |
| Index entries on writes | Racing writes to one partition of a skip-list memtable could drop a row from its index; a write a trie memtable failed to apply still reached the index; updates and deletions left stale or missing entries in memtable indexes | [`0235c5d3984`](https://github.com/JeremiahDJordan/cassandra/commit/0235c5d398482723b2fc97271205ce0b97a9f7c7), [`ce4eebbd4f1`](https://github.com/JeremiahDJordan/cassandra/commit/ce4eebbd4f15a594cf9af35ddcaec14b27497c25), [`325279bbb41`](https://github.com/JeremiahDJordan/cassandra/commit/325279bbb41a3b8ea02a0fe287f64d7b9404a01e), [`f6ac88c6c17`](https://github.com/JeremiahDJordan/cassandra/commit/f6ac88c6c17f689069a4369eb8e59cddffdf8d5a) and others |
| When an index is queryable | A new index could adopt the files of a dropped index of the same name; indexes could answer without SSTables added by streams, imports or compaction | [`995d2963296`](https://github.com/JeremiahDJordan/cassandra/commit/995d2963296bcb2c098a5632090d4282ef0080bf), [`c8e58e04874`](https://github.com/JeremiahDJordan/cassandra/commit/c8e58e0487474b5616fde9136b4875e3e8243f5b), [`af4e52a0c11`](https://github.com/JeremiahDJordan/cassandra/commit/af4e52a0c119fbecc91d715185c8263b2b98e89a), [`041930d6280`](https://github.com/JeremiahDJordan/cassandra/commit/041930d6280fc8516c6af3c814e88743bd82b040) and others |
| Static rows and `OR` | An `OR` of a static and a regular column was evaluated as `AND`; indexes on partition key components missed static-only partitions after a build or compaction; replica filtering protection could miss rows when replicas disagreed on the static row | [`a8c4b2529ec`](https://github.com/JeremiahDJordan/cassandra/commit/a8c4b2529ec9085b9d5e0f0e9a2df419af31f491), [`925eff0c75d`](https://github.com/JeremiahDJordan/cassandra/commit/925eff0c75dfabdbbbdd39f1c0f61b5d2f42f4f7), [`d00fa3410ea`](https://github.com/JeremiahDJordan/cassandra/commit/d00fa3410eaa1c1a9d17b85e7b5eb12648f9462f) and others |
| Value types | On DESC `inet`, `varint` and `decimal` clustering columns, ranges missed rows, `inet` and `varint` index flushes failed, and reads and builds of `decimal` indexes failed (CASSANDRA-20100); the term size guardrails were never enforced, so oversized terms were left out of the index (CASSANDRA-19493) | [`5d22616e371`](https://github.com/JeremiahDJordan/cassandra/commit/5d22616e371db107087034833c341473452591ab), [`b6c0e11a1ab`](https://github.com/JeremiahDJordan/cassandra/commit/b6c0e11a1abd20f5da5317e8f0f333a6ccc171a8) |
| Query and index support | `LIKE` in filtering queries and SAI on `blob` columns, which Apache Cassandra has on trunk, were missing; reads that need an index a replica doesn't know yet failed with an internal error | [`331dd77ba02`](https://github.com/JeremiahDJordan/cassandra/commit/331dd77ba029ff8154ca46fdf102e48dee2fb685), [`744cd7cf376`](https://github.com/JeremiahDJordan/cassandra/commit/744cd7cf3767d0fce48c014b317984b7688456b1), [`817347ca894`](https://github.com/JeremiahDJordan/cassandra/commit/817347ca894d56d8dc9f8608e76a93047dd4359b) |

## In numbers

- 183 new issues filed on the [fork](https://github.com/JeremiahDJordan/cassandra/issues), 32 of them blocker or
  critical, with 209 pull requests against the fork's copies of the released DataStax and Apache branches.
- The series: 107 commits, about 64,000 added lines, three quarters of them tests. The full unit and in-JVM dtest
  suites pass on the published branch.
