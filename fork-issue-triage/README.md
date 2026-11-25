# Fork issue triage, 25 November 2026

*Read it as a web page: <https://jeremiahdjordan.github.io/ai-review-guides/fork-issue-triage/>*

A triage of the bug reports on the [JeremiahDJordan/cassandra](https://github.com/JeremiahDJordan/cassandra/issues) fork: what areas they cover, which ones duplicate each other, which ones could be merged, and which ones Apache Cassandra or DataStax had already fixed upstream.

## Snapshot

Issues and pull requests were listed on 25 November 2026 at 20:24 UTC: **394 issues** (393 open, 1 closed) and **466 pull requests** (462 open), up to issue #859. Other people kept filing issues while this triage ran; issues filed after the snapshot are not covered.

After this triage, **388 issues are open**: 128 labelled `apache` (Apache Cassandra code) and 261 labelled `datastax` (DataStax code); [#827](https://github.com/JeremiahDJordan/cassandra/issues/827) carries both labels. 5 issues and 5 pull requests were closed because the fix is already upstream. 457 pull requests are open.

| Severity of the open issues | Issues |
|---|---|
| blocker | 5 |
| critical | 66 |
| high | 82 |
| medium | 98 |
| low | 122 |
| (none) | 15 |

By issue type: 373 Bug, 9 Performance Improvement, 4 Documentation, 2 Feature. Only bugs have a severity label. [#549](https://github.com/JeremiahDJordan/cassandra/issues/549), [#550](https://github.com/JeremiahDJordan/cassandra/issues/550), [#645](https://github.com/JeremiahDJordan/cassandra/issues/645) have no branch label: each describes a problem that exists only with an unmerged fix for another issue (see the merge proposals).

## How the upstream check was done

- Upstream branches fetched on 25 November 2026: Apache `cassandra-4.0` `ff5a1eb859c`, `cassandra-4.1` `14e042c88ed`, `cassandra-5.0` `3e3892ce99a`, `cassandra-6.0` `243566b968a`, `trunk` `25cb50d1f7c`; DataStax `main` `f7a7fc031a3` and `main-4.0` `b1a64c8e5fe`.
- The fork's `apache/base-*` branches are at the Apache tips. `datastax/base-5.0` is 16 commits behind `datastax/main` and `datastax/base-4.0` 4 commits behind `main-4.0`. 11 commits reached `cassandra-5.0` after the commit most Apache 5.0 issues were filed against (`fb7f1b7aeeb`).
- Every open PR's diff was matched by patch ID against the upstream commits since the bases, and applied (forward and reverse) to the upstream tip of its base. No PR matched an upstream commit exactly and none was already applied. Leaving out `CHANGES.txt`, 11 no longer apply cleanly; 3 of those are closed below.
- The upstream commits that touch the same code as an issue were read, and for the likely candidates the PR's tests were run on the upstream tip without the fix (and on the old base where needed to confirm they fail there).

## Bug areas

| Area | Open | Apache | DataStax | Severity |
|---|---|---|---|---|
| [SAI query results](#sai-query-results) | 66 | 14 | 52 | 15 critical, 25 high, 18 medium, 8 low |
| [Index builds and index state](#index-builds-and-index-state) | 42 | 11 | 31 | 5 critical, 17 high, 15 medium, 5 low |
| [Vector search, BM25 and ORDER BY](#vector-search-bm25-and-order-by) | 38 | 12 | 26 | 16 critical, 9 high, 9 medium, 4 low |
| [Replica reconciliation, filtering, paging and GROUP BY](#replica-reconciliation-filtering-paging-and-group-by) | 20 | 9 | 11 | 11 critical, 4 high, 2 medium, 3 (none) |
| [CQL literals, bound collection values, functions and restrictions](#cql-literals-bound-collection-values-functions-and-restrictions) | 35 | 20 | 16 | 3 critical, 7 medium, 25 low |
| [Mixed versions and storage compatibility](#mixed-versions-and-storage-compatibility) | 19 | 1 | 18 | 4 critical, 5 high, 1 medium, 8 low, 1 (none) |
| [SAI resources, metrics, options and documentation](#sai-resources-metrics-options-and-documentation) | 19 | 1 | 18 | 1 high, 1 medium, 9 low, 8 (none) |
| [Trie memtable and byte-comparable values](#trie-memtable-and-byte-comparable-values) | 17 | 1 | 16 | 3 blocker, 5 critical, 3 high, 2 medium, 4 low |
| [Compaction, SSTables and tools](#compaction-sstables-and-tools) | 16 | 4 | 12 | 2 blocker, 2 critical, 2 high, 4 medium, 5 low, 1 (none) |
| [Cluster metadata (TCM) and upgrades to 6.0](#cluster-metadata-tcm-and-upgrades-to-60) | 15 | 15 | 0 | 2 high, 11 medium, 2 low |
| [only_purge_repaired_tombstones](#only_purge_repaired_tombstones) | 12 | 6 | 6 | 6 high, 2 medium, 4 low |
| [Schema changes, dropped columns and user types](#schema-changes-dropped-columns-and-user-types) | 11 | 5 | 6 | 2 critical, 1 high, 7 medium, 1 low |
| [Legacy secondary indexes](#legacy-secondary-indexes) | 8 | 2 | 6 | 3 critical, 5 medium |
| [Auto-repair](#auto-repair) | 6 | 6 | 0 | 4 medium, 2 low |
| [Client protocol and buffers](#client-protocol-and-buffers) | 6 | 2 | 4 | 1 high, 2 medium, 3 low |
| [Hints](#hints) | 4 | 2 | 2 | 1 high, 2 medium, 1 low |
| [Commit log and durability](#commit-log-and-durability) | 4 | 3 | 1 | 2 high, 2 low |
| [Gossip, membership, reads and other](#gossip-membership-reads-and-other) | 14 | 6 | 8 | 3 high, 5 medium, 5 low, 1 (none) |
| [Test harness, build and CI](#test-harness-build-and-ci) | 36 | 8 | 28 | 1 medium, 34 low, 1 (none) |

Each issue is in one area, by what its fix changes. The four SAI and vector search areas hold 165 of the 388 open issues (43%).

### SAI query results

Storage-attached index (SAI) queries that return wrong rows, miss rows or fail, outside vector search: negations (`!=`, `NOT IN`, `NOT CONTAINS`), empty values and empty bounds, analyzers, collections and map entries, `inet`, `decimal` and `varint` ordering, descending clustering columns, values written in another serialization, token collisions, the memtable index under concurrent writes and updates, strict filtering across replicas, and the term size guardrails.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#128](https://github.com/JeremiahDJordan/cassandra/issues/128) | Apache | 5.0, 6.0, trunk | critical | SAI queries on two columns with a limit or paging return no rows at LOCAL_QUORUM in a datacenter with a single replica |
| [#133](https://github.com/JeremiahDJordan/cassandra/issues/133) | Apache | 5.0, 6.0, trunk | critical | Paged partition-restricted index queries without strict filtering return rows of other partitions or page around the ring forever |
| [#198](https://github.com/JeremiahDJordan/cassandra/issues/198) | DataStax | main-4.0, main-5.0 | critical | SAI index on a partition key component misses the partitions that only have a static row after an index build or a compaction, and CREATE INDEX on a ... |
| [#199](https://github.com/JeremiahDJordan/cassandra/issues/199) | DataStax | main-4.0, main-5.0 | critical | A WHERE clause with an OR of a static column and a regular column is evaluated as an AND: pk1 = 3 AND (v = 1 OR st = 2) drops matching rows |
| [#221](https://github.com/JeremiahDJordan/cassandra/issues/221) | DataStax | main-4.0, main-5.0 | critical | Concurrent writes of a row on a skip-list memtable can drop the row from its SAI index |
| [#225](https://github.com/JeremiahDJordan/cassandra/issues/225) | DataStax | main-4.0, main-5.0 | critical | A value that was already expired when the SAI memtable index got it, written again unchanged, leaves the row out of the index until compaction |
| [#239](https://github.com/JeremiahDJordan/cassandra/issues/239) | DataStax | main-4.0, main-5.0 | critical | SAI ignores RowFilter#isStrict: a row whose matching columns are on different replicas is never returned at consistency levels that reconcile |
| [#246](https://github.com/JeremiahDJordan/cassandra/issues/246) | DataStax | main-4.0, main-5.0 | critical | SAI on DESC inet, varint and decimal clustering columns: ranges return no rows, flushes fail, and decimal indexes fail queries and compaction |
| [#249](https://github.com/JeremiahDJordan/cassandra/issues/249) | DataStax | main-4.0, main-5.0 | critical | SAI term size guardrails are never enforced: oversized terms are written but left out of the index, and vectors are lost on compaction |
| [#266](https://github.com/JeremiahDJordan/cassandra/issues/266) | DataStax | main-4.0, main-5.0 | critical | SAI memtable index under-skips into another partition's keys when an index mixes static and regular row keys: skipTo assertion, or a matching row ... |
| [#277](https://github.com/JeremiahDJordan/cassandra/issues/277) | DataStax | main-4.0, main-5.0 | critical | After ALTER TYPE ... ADD, SAI indexes on frozen values of the type return no rows, and ENTRIES indexes fail writes, reads and compaction |
| [#373](https://github.com/JeremiahDJordan/cassandra/issues/373) | DataStax | main-4.0, main-5.0 | critical | SAI: an index caches the descriptor of an early-opened compaction result, and the final SSTable is missing from every index until restart |
| [#426](https://github.com/JeremiahDJordan/cassandra/issues/426) | DataStax | main-5.0 | critical | SAI doesn't return range tombstones for filters on static columns, so replica filtering protection returns deleted rows |
| [#696](https://github.com/JeremiahDJordan/cassandra/issues/696) | DataStax | main-4.0, main-5.0 | critical | SAI filtering of an unindexed map entry fails or misses rows when the key or value doesn't fit in a composite (65535-byte key, value over 65535 bytes) |
| [#707](https://github.com/JeremiahDJordan/cassandra/issues/707) | DataStax | main-4.0, main-5.0 | critical | SAI: a range with != or NOT IN on a literal column filtered in another index's read returns no rows |
| [#226](https://github.com/JeremiahDJordan/cassandra/issues/226) | DataStax | main-4.0, main-5.0 | high | SAI misses rows updated to the empty value in the memtable: v = '' and CONTAINS '' don't find them, also after flush |
| [#227](https://github.com/JeremiahDJordan/cassandra/issues/227) | DataStax | main-4.0, main-5.0 | high | An update in the memtable removes a row from an SAI term the row still holds: a duplicate collection element, a decimal with another scale, a long ... |
| [#242](https://github.com/JeremiahDJordan/cassandra/issues/242) | DataStax | main-4.0, main-5.0 | high | Non-strict SAI reads with all index matches repaired still intersect unindexed expressions strictly and lose rows split across replicas |
| [#247](https://github.com/JeremiahDJordan/cassandra/issues/247) | DataStax | main-4.0, main-5.0 | high | SAI on a DESC clustering column doesn't index the empty value and serves !=, NOT IN and uuid ranges that the ASC index rejects |
| [#251](https://github.com/JeremiahDJordan/cassandra/issues/251) | DataStax | main-4.0, main-5.0 | high | SAI misses equal values written in another serialization in frozen values, map entries and varints (2.5 vs 2.50, 0x0001 vs 0x01) |
| [#252](https://github.com/JeremiahDJordan/cassandra/issues/252) | DataStax | main-4.0, main-5.0 | high | != on SAI map entries and frozen values before on-disk version db misses rows whose bytes sort differently from their values |
| [#253](https://github.com/JeremiahDJordan/cassandra/issues/253) | DataStax | main-4.0, main-5.0 | high | SAI NOT CONTAINS and NOT CONTAINS KEY miss rows whose collection held the value in an expired or overwritten element |
| [#254](https://github.com/JeremiahDJordan/cassandra/issues/254) | DataStax | main-4.0, main-5.0 | high | m[k] != v on an SAI map entries index misses rows without the key and rows with a null or empty map |
| [#255](https://github.com/JeremiahDJordan/cassandra/issues/255) | DataStax | main-4.0, main-5.0 | high | Key scans of SAI negations can skip back into a partition they already returned rows of, looping until the query times out |
| [#256](https://github.com/JeremiahDJordan/cassandra/issues/256) | DataStax | main-4.0, main-5.0 | high | SAI negations on an indexed static collection miss the partitions that only have a static row |
| [#279](https://github.com/JeremiahDJordan/cassandra/issues/279) | DataStax | main-4.0, main-5.0 | high | Analyzed CONTAINS of several words matches words in different elements on the replica, and analyzed NOT CONTAINS excludes the rows holding any of ... |
| [#370](https://github.com/JeremiahDJordan/cassandra/issues/370) | DataStax | main-5.0 | high | SAI: index files written by Apache Cassandra 5.0 are parsed as DSE 6.8 files |
| [#429](https://github.com/JeremiahDJordan/cassandra/issues/429) | DataStax | main-4.0, main-5.0 | high | Analyzed SAI NOT CONTAINS drops the rows holding an element the analyzer makes no term of |
| [#503](https://github.com/JeremiahDJordan/cassandra/issues/503) | Apache | 5.0, 6.0, trunk | high | SAI orders and compares inet values as IPv6, so range and equality queries return other rows than filtering without SAI |
| [#504](https://github.com/JeremiahDJordan/cassandra/issues/504) | DataStax | main-4.0, main-5.0 | high | SAI orders and compares inet values as IPv6, so range and equality queries return other rows than filtering without SAI |
| [#515](https://github.com/JeremiahDJordan/cassandra/issues/515) | Apache | 5.0, trunk | high | SAI: a replica that doesn't know a non-tokenizing index answers = and CONTAINS reads without its options and misses rows |
| [#516](https://github.com/JeremiahDJordan/cassandra/issues/516) | DataStax | main-4.0, main-5.0 | high | SAI: a replica that doesn't know an analyzed collection index answers CONTAINS reads without the analyzer and misses rows |
| [#594](https://github.com/JeremiahDJordan/cassandra/issues/594) | DataStax | main-4.0, main-5.0 | high | SAI on a DESC inet clustering column: the SSTable index's term range is narrowed or inverted, and index builds and compactions can fail |
| [#627](https://github.com/JeremiahDJordan/cassandra/issues/627) | DataStax | main-4.0, main-5.0 | high | SAI: IN with several values on an analyzed column fails, and the index ORs the terms of each value |
| [#725](https://github.com/JeremiahDJordan/cassandra/issues/725) | Apache | 5.0, 6.0, trunk | high | SAI: the restrictions left to filtering on a column with an analyzed index (!=, IN, NOT IN, NOT CONTAINS, NOT CONTAINS KEY) compare raw values, ... |
| [#726](https://github.com/JeremiahDJordan/cassandra/issues/726) | Apache | 5.0, 6.0, trunk | high | SAI filtering of an unindexed map entry fails the read when a row holds an entry whose key or value is over 65535 bytes |
| [#727](https://github.com/JeremiahDJordan/cassandra/issues/727) | Apache | 6.0, trunk | high | Map entry restrictions on a frozen map partition key match every partition (fm[k] = v, fm[k] != v) |
| [#758](https://github.com/JeremiahDJordan/cassandra/issues/758) | DataStax | main-4.0, main-5.0 | high | SAI: != and NOT IN on a column whose analyzed index serves = as a match compare raw values, while = uses the analyzer |
| [#785](https://github.com/JeremiahDJordan/cassandra/issues/785) | Apache | 5.0, 6.0, trunk | high | SAI term size failure threshold isn't applied when the warning threshold is disabled, or when the analyzer makes a term larger than the value: the ... |
| [#798](https://github.com/JeremiahDJordan/cassandra/issues/798) | Apache | 5.0, 6.0, trunk | high | SAI sstable index writer checks the size of an analyzed value instead of its terms: compaction and index rebuild index different rows than the ... |
| [#200](https://github.com/JeremiahDJordan/cassandra/issues/200) | DataStax | main-4.0, main-5.0 | medium | A partition without matching rows is returned as its static content when the row restrictions are inside an OR: pk = 13 AND (v = 1 OR v = 9) |
| [#248](https://github.com/JeremiahDJordan/cassandra/issues/248) | DataStax | main-4.0, main-5.0 | medium | SAI queries with an empty value as a bound on a non-literal index fail the read |
| [#250](https://github.com/JeremiahDJordan/cassandra/issues/250) | DataStax | main-4.0, main-5.0 | medium | Map entries with a key or value of 65535 bytes or more on an SAI ENTRIES index fail the write after it is applied, and break index builds and ... |
| [#275](https://github.com/JeremiahDJordan/cassandra/issues/275) | DataStax | main-4.0, main-5.0 | medium | SAI memtable index misses empty frozen tuple and UDT values for != and NOT IN, and != of an empty value fails the read |
| [#276](https://github.com/JeremiahDJordan/cassandra/issues/276) | DataStax | main-4.0, main-5.0 | medium | SAI version GA rejects distinct partition keys that share a token, and SAI primary key maps resolve such keys by token |
| [#282](https://github.com/JeremiahDJordan/cassandra/issues/282) | DataStax | main-4.0, main-5.0 | medium | NOT CONTAINS KEY on an analyzed map KEYS index fails with NullPointerException for a row without the map |
| [#395](https://github.com/JeremiahDJordan/cassandra/issues/395) | DataStax | main-4.0, main-5.0 | medium | SAI row estimates fail queries on infinite and NaN floats and on very large varints and decimals, and DESC float and double estimates are far off |
| [#430](https://github.com/JeremiahDJordan/cassandra/issues/430) | DataStax | main-4.0, main-5.0 | medium | SAI AND of two indexes fails with "skipTo skipped to an item smaller than the target" on SSTables of SAI versions ga and ec |
| [#465](https://github.com/JeremiahDJordan/cassandra/issues/465) | DataStax | main-4.0, main-5.0 | medium | SAI queries with CONTAINS on an unindexed frozen collection fail with IndexOutOfBoundsException |
| [#469](https://github.com/JeremiahDJordan/cassandra/issues/469) | DataStax | main-4.0, main-5.0 | medium | An analyzed : match of a value without terms matches every row in RowFilter and none through the SAI index |
| [#532](https://github.com/JeremiahDJordan/cassandra/issues/532) | Apache | 5.0, 6.0, trunk | medium | SAI range restrictions with the empty value as a bound fail the read on non-literal indexes |
| [#533](https://github.com/JeremiahDJordan/cassandra/issues/533) | Apache | 5.0, 6.0, trunk | medium | SAI leaves the iterators of earlier segments and SSTable indexes open when a later search of the query fails |
| [#541](https://github.com/JeremiahDJordan/cassandra/issues/541) | Apache | 5.0, 6.0, trunk | medium | SAI returns a wide partition without a partition deletion written during the search, with the deletion applied to its later rows |
| [#542](https://github.com/JeremiahDJordan/cassandra/issues/542) | DataStax | main-4.0, main-5.0 | medium | SAI returns a wide partition without a partition deletion written during the search, with the deletion applied to its later rows |
| [#600](https://github.com/JeremiahDJordan/cassandra/issues/600) | DataStax | main-4.0, main-5.0 | medium | SAI range estimates on decimals with far-apart exponents stall or fail the query |
| [#730](https://github.com/JeremiahDJordan/cassandra/issues/730) | Apache | 6.0, trunk | medium | During an upgrade from 5.0, 6.0 coordinators send SAI reads with != or non-strict IN that 5.0 replicas ignore or reject |
| [#763](https://github.com/JeremiahDJordan/cassandra/issues/763) | DataStax | main-4.0, main-5.0 | medium | SAI: a match of a value the query analyzer makes no terms of returns the rows holding '' when the index analyzer keeps the empty term |
| [#797](https://github.com/JeremiahDJordan/cassandra/issues/797) | Apache | 5.0, 6.0, trunk | medium | SAI term size guardrail isn't applied to a partition key component written by a static-only insert: the write succeeds and the index misses the ... |
| [#79](https://github.com/JeremiahDJordan/cassandra/issues/79) | DataStax | main-4.0, main-5.0 | low | IN on an SAI-indexed partition key column is rejected in key range queries |
| [#183](https://github.com/JeremiahDJordan/cassandra/issues/183) | DataStax | main-4.0, main-5.0 | low | A filtered (c) IN ((x), (y)) with a null element fails the read when an SAI index serves the query |
| [#271](https://github.com/JeremiahDJordan/cassandra/issues/271) | DataStax | main-4.0, main-5.0 | low | SAI range query with a value over 65535 bytes times out instead of failing with InvalidRequestException |
| [#280](https://github.com/JeremiahDJordan/cassandra/issues/280) | DataStax | main-4.0, main-5.0 | low | SAI range on a map entry with a key of 32768 bytes or more fails |
| [#281](https://github.com/JeremiahDJordan/cassandra/issues/281) | DataStax | main-4.0, main-5.0 | low | SAI queries on partition keys that share a token fail or return wrong rows when SAI version aa is the write version |
| [#301](https://github.com/JeremiahDJordan/cassandra/issues/301) | DataStax | main-4.0, main-5.0 | low | With a tokenizing analyzer, NOT CONTAINS '' only returns the rows without the collection |
| [#560](https://github.com/JeremiahDJordan/cassandra/issues/560) | DataStax | main-4.0, main-5.0 | low | SegmentFlushTest leaves ~43 GB of temporary index files behind |
| [#786](https://github.com/JeremiahDJordan/cassandra/issues/786) | DataStax | main-5.0 | low | SAI: a client write whose analyzed term is over the term size failure threshold is accepted, and the term not indexed, when the value as written is ... |

### Index builds and index state

An index that becomes queryable without some SSTables, or loses or reuses index files: index builds that overlap streaming, import, compaction, TRUNCATE, `rebuild_index` or a restart; an index dropped and created again under the same name; build state kept by index name in `SecondaryIndexManager`. Mostly SAI, some shared with legacy indexes.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#134](https://github.com/JeremiahDJordan/cassandra/issues/134) | Apache | 5.0, 6.0, trunk | critical | SAI index created while a stream is received silently misses the rows of the streamed SSTables |
| [#368](https://github.com/JeremiahDJordan/cassandra/issues/368) | DataStax | main-5.0 | critical | SAI: an index created while a stream is received, or before streamed or imported SSTables are added, misses their rows |
| [#369](https://github.com/JeremiahDJordan/cassandra/issues/369) | DataStax | main-4.0, main-5.0 | critical | SAI: a new index adopts the files of a dropped or imported index of the same name and returns wrong results |
| [#583](https://github.com/JeremiahDJordan/cassandra/issues/583) | DataStax | main-4.0 | critical | SAI: a stopped index rebuild reports success and leaves the index answering without the SSTable |
| [#748](https://github.com/JeremiahDJordan/cassandra/issues/748) | Apache | 5.0, 6.0, trunk | critical | SAI: an index created over SSTables that already have per-SSTable SAI files misses rows after a row that expired since the flush or a tombstone-only ... |
| [#161](https://github.com/JeremiahDJordan/cassandra/issues/161) | Apache | 5.0, 6.0, trunk | high | An SAI index created just before streamed or imported SSTables are added misses their rows, and a failed index build leaves those SSTables in the ... |
| [#360](https://github.com/JeremiahDJordan/cassandra/issues/360) | DataStax | main-4.0, main-5.0 | high | SAI: at startup an SSTable whose index components fail to load counts as indexed, so the index is queryable without it |
| [#374](https://github.com/JeremiahDJordan/cassandra/issues/374) | DataStax | main-5.0 | high | SAI: an initial index build becomes queryable without the SSTables restored by a compaction stopped during the build |
| [#375](https://github.com/JeremiahDJordan/cassandra/issues/375) | DataStax | main-4.0, main-5.0 | high | SAI: indexes stay queryable without an SSTable that comes back without its per-SSTable components |
| [#376](https://github.com/JeremiahDJordan/cassandra/issues/376) | DataStax | main-5.0 | high | SAI: nodetool rebuild_index makes its index queryable without an SSTable removed and added again during the rebuild |
| [#379](https://github.com/JeremiahDJordan/cassandra/issues/379) | DataStax | main-4.0, main-5.0 | high | nodetool rebuild_index doesn't flush the memtable when the index is writable, so it reports success while a write the index missed stays missing |
| [#382](https://github.com/JeremiahDJordan/cassandra/issues/382) | DataStax | main-5.0 | high | SAI: CREATE INDEX makes the index queryable without an SSTable reloaded during its build (index summary redistribution) |
| [#513](https://github.com/JeremiahDJordan/cassandra/issues/513) | Apache | 5.0, trunk | high | SAI: a replica whose index of the same name was recreated with other options answers reads with its own options and can miss rows |
| [#514](https://github.com/JeremiahDJordan/cassandra/issues/514) | DataStax | main-4.0, main-5.0 | high | SAI: a replica whose index of the same name was recreated with other options answers reads with its own analyzer and can miss rows |
| [#527](https://github.com/JeremiahDJordan/cassandra/issues/527) | Apache | 4.0, 4.1, 5.0, trunk | high | Secondary indexes: a node that gets an index dropped and created again with other options in one schema change keeps the old index |
| [#528](https://github.com/JeremiahDJordan/cassandra/issues/528) | DataStax | main-4.0, main-5.0 | high | Secondary indexes: a node that gets an index dropped and created again with other options in one schema change keeps the old index |
| [#538](https://github.com/JeremiahDJordan/cassandra/issues/538) | Apache | 5.0, trunk | high | SAI: a new index uses the files of a dropped index of the same name while a query holds them |
| [#585](https://github.com/JeremiahDJordan/cassandra/issues/585) | Apache | 5.0, 6.0, trunk | high | SAI: TRUNCATE makes an index queryable that misses an SSTable the truncate kept |
| [#586](https://github.com/JeremiahDJordan/cassandra/issues/586) | DataStax | main-4.0, main-5.0 | high | SAI: TRUNCATE makes an index queryable that misses an SSTable the truncate kept |
| [#630](https://github.com/JeremiahDJordan/cassandra/issues/630) | DataStax | main-4.0, main-5.0 | high | SAI: an index build waiting for another build's per-SSTable files reports success when that build fails, leaving the index queryable without the ... |
| [#631](https://github.com/JeremiahDJordan/cassandra/issues/631) | DataStax | main-4.0 | high | SAI: two builds of the same index on one SSTable write the same files at once, so queries fail and the index loses its files |
| [#635](https://github.com/JeremiahDJordan/cassandra/issues/635) | DataStax | main-4.0, main-5.0 | high | SAI: an SSTable whose complete index components fail to load is treated as indexed, and the index becomes queryable without it |
| [#6](https://github.com/JeremiahDJordan/cassandra/issues/6) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | medium | DROP INDEX fails with "was already marked compacted" after a legacy index is re-created with the same name during a compaction |
| [#31](https://github.com/JeremiahDJordan/cassandra/issues/31) | DataStax | main-4.0, main-5.0 | medium | DROP INDEX of a legacy index while it is in use can fail the build, write, compaction or read with a NullPointerException in SingletonIndexGroup |
| [#361](https://github.com/JeremiahDJordan/cassandra/issues/361) | DataStax | main-4.0, main-5.0 | medium | SAI: an already built index becomes queryable at restart only after gossip has started |
| [#371](https://github.com/JeremiahDJordan/cassandra/issues/371) | DataStax | main-5.0 | medium | SAI: the index build before nodetool import adds SSTables treats them as live, and a failed build makes the index non-queryable |
| [#372](https://github.com/JeremiahDJordan/cassandra/issues/372) | DataStax | main-4.0, main-5.0 | medium | Secondary indexes: the build of a dropped index changes the build state of a new index of the same name |
| [#378](https://github.com/JeremiahDJordan/cassandra/issues/378) | DataStax | main-4.0, main-5.0 | medium | Secondary indexes: if CompactionManager#submitIndexBuild throws, the indexes stay queryable without the SSTables and count as building forever |
| [#534](https://github.com/JeremiahDJordan/cassandra/issues/534) | Apache | 5.0, trunk | medium | Secondary indexes: the build of a dropped index changes the build state of a new index of the same name |
| [#559](https://github.com/JeremiahDJordan/cassandra/issues/559) | DataStax | main-4.0, main-5.0 | medium | SAI: a full rebuild of a dropped index deletes, when it aborts, the files of a new index of the same name |
| [#564](https://github.com/JeremiahDJordan/cassandra/issues/564) | DataStax | main-4.0, main-5.0 | medium | SAI: with immutable components, a rebuild that fails after writing the per-SSTable components leaves a generation that the next build registers, and ... |
| [#582](https://github.com/JeremiahDJordan/cassandra/issues/582) | DataStax | main-5.0 | medium | SAI: a TRUNCATE during an index's initial build leaves the index non-queryable |
| [#603](https://github.com/JeremiahDJordan/cassandra/issues/603) | DataStax | main-4.0, main-5.0 | medium | Index builds: one failing index type marks the other indexes of the build failed while they are still building |
| [#612](https://github.com/JeremiahDJordan/cassandra/issues/612) | DataStax | main-5.0 | medium | SAI: an index built on an early-opened compaction result is missing from the final SSTable's TOC and components, and whole-SSTable streaming of that ... |
| [#636](https://github.com/JeremiahDJordan/cassandra/issues/636) | DataStax | main-4.0, main-5.0 | medium | SAI: the truncate task can make an index queryable over a flush whose index failed to load meanwhile |
| [#637](https://github.com/JeremiahDJordan/cassandra/issues/637) | DataStax | main-4.0, main-5.0 | medium | SAI: TOC of SSTables written by streaming and other non-compaction operations lists index components that weren't written |
| [#647](https://github.com/JeremiahDJordan/cassandra/issues/647) | DataStax | main-5.0 | medium | SAI: a failed or stopped index build of streamed or imported SSTables makes the index non-queryable, though they are never added |
| [#174](https://github.com/JeremiahDJordan/cassandra/issues/174) | Apache | 5.0, 6.0, trunk | low | An SAI index created while the node is starting misses the rows of SSTables that a stream or an import is adding |
| [#381](https://github.com/JeremiahDJordan/cassandra/issues/381) | DataStax | main-5.0 | low | SAI: a partial build that fails during nodetool rebuild_index is overridden when the rebuild completes, leaving the index queryable without the ... |
| [#535](https://github.com/JeremiahDJordan/cassandra/issues/535) | Apache | 4.0, 4.1, 5.0, trunk | low | SASI: a dropped index stays subscribed to its table's tracker |
| [#613](https://github.com/JeremiahDJordan/cassandra/issues/613) | DataStax | main-4.0, main-5.0 | low | Secondary indexes: a rebuild whose SAI index is dropped before its build starts leaves the other indexes of the rebuild building until restart |
| [#634](https://github.com/JeremiahDJordan/cassandra/issues/634) | DataStax | main-4.0, main-5.0 | low | Secondary indexes: a schema change fails while the build counter of an index that doesn't register itself is above zero |

### Vector search, BM25 and ORDER BY

ANN, BM25 and generic `ORDER BY` queries: rows lost or repeated with static columns, partition and token restrictions, `IN` and negations applied after the top-k, the vector memtable index under concurrent writes and TRUNCATE, vector graph and flush failures, and vector columns of user types.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#80](https://github.com/JeremiahDJordan/cassandra/issues/80) | DataStax | main-4.0, main-5.0 | critical | Concurrent writes can lose VectorMemtableIndex key bounds updates, so ANN queries can miss memtable rows |
| [#137](https://github.com/JeremiahDJordan/cassandra/issues/137) | Apache | 5.0, 6.0, trunk | critical | Concurrent writes of one row can leave the SAI vector memtable index with a wrong, missing or extra vector for it |
| [#178](https://github.com/JeremiahDJordan/cassandra/issues/178) | Apache | 5.0, 6.0, trunk | critical | SAI vector memtable index keeps the old vector of a row when a delete leaves the row with no live data, so ANN queries rank the row by a vector it ... |
| [#202](https://github.com/JeremiahDJordan/cassandra/issues/202) | DataStax | main-4.0, main-5.0 | critical | SAI ORDER BY returns only the first row of each partition for a static vector column, and the first rows instead of the best ones with PER PARTITION ... |
| [#222](https://github.com/JeremiahDJordan/cassandra/issues/222) | DataStax | main-4.0, main-5.0 | critical | Concurrent writes of one row on a skip-list memtable can leave the SAI vector memtable index with a wrong, missing or extra vector for it |
| [#223](https://github.com/JeremiahDJordan/cassandra/issues/223) | DataStax | main-4.0, main-5.0 | critical | A vector memtable index flush fails with "value already present" when a row id is in the postings of two vectors, and keeps failing |
| [#224](https://github.com/JeremiahDJordan/cassandra/issues/224) | DataStax | main-4.0, main-5.0 | critical | Partition-restricted ANN and token sub-range replica reads miss a row that got its vector by an UPDATE while the row is in the memtable |
| [#479](https://github.com/JeremiahDJordan/cassandra/issues/479) | DataStax | main-4.0, main-5.0 | critical | After a TRUNCATE without a flush, a write racing it can leave the truncated memtable's SAI index live: ANN queries return the truncated row or fail |
| [#495](https://github.com/JeremiahDJordan/cassandra/issues/495) | Apache | 5.0, 6.0, trunk | critical | After a TRUNCATE without a flush, a write racing it can leave the truncated memtable's SAI index live, and ANN queries return the truncated row |
| [#742](https://github.com/JeremiahDJordan/cassandra/issues/742) | Apache | 5.0 | critical | SAI vector search with IN on another indexed column applies the IN after the top-k and returns fewer rows than the limit |
| [#744](https://github.com/JeremiahDJordan/cassandra/issues/744) | Apache | 6.0, trunk | critical | SAI vector search with IN on another indexed column returns no rows on cassandra-6.0 and trunk |
| [#746](https://github.com/JeremiahDJordan/cassandra/issues/746) | Apache | 5.0, 6.0, trunk | critical | SAI vector search with an equality restriction on an indexed static column returns no rows |
| [#759](https://github.com/JeremiahDJordan/cassandra/issues/759) | Apache | 5.0, 6.0, trunk | critical | SAI vector search ordered by a static vector column with a restriction on another indexed column returns no rows |
| [#775](https://github.com/JeremiahDJordan/cassandra/issues/775) | Apache | 6.0, trunk | critical | SAI vector search with a negated restriction (!=, NOT IN, NOT CONTAINS, NOT CONTAINS KEY) returns fewer rows than its limit, or none |
| [#787](https://github.com/JeremiahDJordan/cassandra/issues/787) | Apache | 5.0, 6.0, trunk | critical | SAI vector search with a restriction on another indexed column fails with ReadFailureException when the query reads some but not all static columns |
| [#788](https://github.com/JeremiahDJordan/cassandra/issues/788) | Apache | 5.0, 6.0, trunk | critical | SAI vector search ordered by a static vector column skips the partitions that only have static data |
| [#190](https://github.com/JeremiahDJordan/cassandra/issues/190) | Apache | 5.0, 6.0, trunk | high | SAI vector queries never complete after a vector search fails: the SSTable indexes are released twice |
| [#203](https://github.com/JeremiahDJordan/cassandra/issues/203) | DataStax | main-4.0, main-5.0 | high | SAI ORDER BY with a predicate on a static column or a partition key component returns no rows, too few rows, or fails with NullPointerException |
| [#204](https://github.com/JeremiahDJordan/cassandra/issues/204) | DataStax | main-4.0, main-5.0 | high | SAI ANN on a vector in a primary key column misses rows, returns rows twice or returns deleted rows when partitions have static rows |
| [#205](https://github.com/JeremiahDJordan/cassandra/issues/205) | DataStax | main-5.0 | high | SAI ANN with a predicate loses rows of a partition with a static row when the plan filters first |
| [#206](https://github.com/JeremiahDJordan/cassandra/issues/206) | DataStax | main-5.0 | high | SAI ORDER BY with a predicate fails with ClassCastException "RangeTombstoneBoundMarker cannot be cast to Row" when a range deletion covers the row |
| [#209](https://github.com/JeremiahDJordan/cassandra/issues/209) | DataStax | main-4.0, main-5.0 | high | SAI ANN with a partition or token restriction returns no rows from SSTables of versions CA and DB when the first row of the range has no vector |
| [#210](https://github.com/JeremiahDJordan/cassandra/issues/210) | DataStax | main-4.0, main-5.0 | high | Generic ORDER BY on tables with clustering columns returns one row per partition, or fails, when an SAI index has version AA |
| [#452](https://github.com/JeremiahDJordan/cassandra/issues/452) | DataStax | main-4.0, main-5.0 | high | Vector memtable indexes searched with a key range stay on the heap after their memtable is flushed or their table is dropped |
| [#471](https://github.com/JeremiahDJordan/cassandra/issues/471) | DataStax | main-4.0, main-5.0 | high | A table with an SAI vector index can't flush on a node with several data directories |
| [#32](https://github.com/JeremiahDJordan/cassandra/issues/32) | DataStax | main-4.0, main-5.0 | medium | ANN queries can fail with ReadFailureException: PairedSlidingWindowReservoir snapshots can contain nulls |
| [#176](https://github.com/JeremiahDJordan/cassandra/issues/176) | Apache | 5.0, 6.0, trunk | medium | SAI vector query with another predicate or a partition restriction fails with "AssertionError: low must be less than high: 0 >= 0" on an SSTable ... |
| [#207](https://github.com/JeremiahDJordan/cassandra/issues/207) | DataStax | main-4.0, main-5.0 | medium | BM25 and scored ANN queries fail with ClassCastException when a row was deleted and written again |
| [#208](https://github.com/JeremiahDJordan/cassandra/issues/208) | DataStax | main-4.0, main-5.0 | medium | SAI ANN with a partition restriction fails with ArrayIndexOutOfBoundsException in SortingIterator during concurrent writes |
| [#211](https://github.com/JeremiahDJordan/cassandra/issues/211) | DataStax | main-4.0, main-5.0 | medium | SAI vector graph written by a compaction or an index rebuild can have a node that no other node links to, so ANN queries never return its rows |
| [#269](https://github.com/JeremiahDJordan/cassandra/issues/269) | DataStax | main-4.0, main-5.0 | medium | Ordered SAI queries (ANN, BM25, ORDER BY) restricting an unindexed clustering column of a fully restricted partition are rejected |
| [#273](https://github.com/JeremiahDJordan/cassandra/issues/273) | DataStax | main-4.0, main-5.0 | medium | With -Dcassandra.float_only_vectors=false SAI accepts ANN indexes on non-float vectors, whose results are meaningless |
| [#307](https://github.com/JeremiahDJordan/cassandra/issues/307) | DataStax | main-4.0, main-5.0 | medium | ORDER BY on a clustering column and the non_partition_restricted_index_query_enabled guardrail miss index restrictions nested in disjunctions |
| [#467](https://github.com/JeremiahDJordan/cassandra/issues/467) | DataStax | main-4.0, main-5.0 | medium | SAI checksum validation skips the vector graph, and every vector component before version ec |
| [#212](https://github.com/JeremiahDJordan/cassandra/issues/212) | DataStax | main-4.0, main-5.0 | low | SAI ANN with ann_options {'rerank_k': 0} on a vector in a primary key column fails with NullPointerException from SSTables |
| [#359](https://github.com/JeremiahDJordan/cassandra/issues/359) | DataStax | main-4.0, main-5.0 | low | SAI: flushes of a table with a vector index fail in VectorPostings.computeRowIds when UCS splits them into shards |
| [#468](https://github.com/JeremiahDJordan/cassandra/issues/468) | DataStax | main-4.0, main-5.0 | low | A replica that doesn't know an SAI index yet fails ANN, BM25 and : reads with an internal error instead of INDEX_NOT_AVAILABLE |
| [#478](https://github.com/JeremiahDJordan/cassandra/issues/478) | DataStax | main-4.0, main-5.0 | low | On-disk SAI vector graph searchers are never closed, so each keeps a buffer pool buffer when SAI files aren't memory-mapped |

### Replica reconciliation, filtering, paging and GROUP BY

Coordinator-side merging of replica responses: replica filtering protection (RFP) and short read protection (SRP) with static columns, range tombstones and limits; `GROUP BY` with limits and paging; filtering on collections with deleted elements.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#10](https://github.com/JeremiahDJordan/cassandra/issues/10) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | critical | GROUP BY with a limit or paging fails with "open RT bound as its last item", or counts deleted rows, when the limits stop in a range tombstone |
| [#129](https://github.com/JeremiahDJordan/cassandra/issues/129) | Apache | 5.0, 6.0, trunk | critical | Replica filtering protection returns deleted rows for filters on static columns when replicas diverge |
| [#131](https://github.com/JeremiahDJordan/cassandra/issues/131) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | critical | Filtering on a collection matches deleted and expired elements and entries |
| [#135](https://github.com/JeremiahDJordan/cassandra/issues/135) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | critical | GROUP BY queries with a limit or paging lose groups and static rows when short read protection reaches a replica that stopped at the group limit |
| [#180](https://github.com/JeremiahDJordan/cassandra/issues/180) | Apache | 5.0, 6.0, trunk | critical | Filtered GROUP BY queries with a limit fail with NullPointerException in Row.Merger on tables with a non-frozen collection |
| [#236](https://github.com/JeremiahDJordan/cassandra/issues/236) | DataStax | main-4.0, main-5.0 | critical | Filtering on a collection matches deleted and expired elements (static collections; regular collections at reconciling consistency levels) |
| [#237](https://github.com/JeremiahDJordan/cassandra/issues/237) | DataStax | main-4.0, main-5.0 | critical | GROUP BY queries with a limit or paging lose groups and static rows when short read protection reaches a replica that stopped at the group limit |
| [#243](https://github.com/JeremiahDJordan/cassandra/issues/243) | DataStax | main-4.0, main-5.0 | critical | Replica filtering protection returns deleted rows, and misses rows, for filters on static columns when replicas diverge |
| [#272](https://github.com/JeremiahDJordan/cassandra/issues/272) | DataStax | main-4.0, main-5.0 | critical | GROUP BY with a limit or paging fails with "open RT bound as its last item", or counts deleted rows, when the limits stop in a range tombstone |
| [#425](https://github.com/JeremiahDJordan/cassandra/issues/425) | DataStax | main-5.0 | critical | Replica filtering protection returns rows deleted by a range tombstone that spans the pages of a single-partition read with LIMIT |
| [#813](https://github.com/JeremiahDJordan/cassandra/issues/813) | DataStax | main-5.0 | critical | Paged filtering reads with a static column expression miss rows that only replicas returning nothing for the page hold |
| [#132](https://github.com/JeremiahDJordan/cassandra/issues/132) | Apache | 5.0, 6.0, trunk | high | PER PARTITION LIMIT returns too many rows when replica filtering protection splits a single-partition read |
| [#238](https://github.com/JeremiahDJordan/cassandra/issues/238) | DataStax | main-5.0 | high | PER PARTITION LIMIT returns too many rows when replica filtering protection splits a single-partition read |
| [#240](https://github.com/JeremiahDJordan/cassandra/issues/240) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | high | Filtering without an index misses a row whose match needs one replica's static row and another replica's row |
| [#241](https://github.com/JeremiahDJordan/cassandra/issues/241) | DataStax | main-4.0, main-5.0 | high | Filtering without an index misses a row whose match needs one replica's static row and another replica's row |
| [#196](https://github.com/JeremiahDJordan/cassandra/issues/196) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | medium | GROUP BY queries with PER PARTITION LIMIT fail with IllegalArgumentException "Cannot create bounds/boundary objects without clustering values" when ... |
| [#201](https://github.com/JeremiahDJordan/cassandra/issues/201) | DataStax | main-4.0, main-5.0 | medium | Paged queries with LIMIT or PER PARTITION LIMIT and an OR of a static and a regular column return more rows than the limit |
| [#167](https://github.com/JeremiahDJordan/cassandra/issues/167) | Apache | 5.0, 6.0, trunk | (none) | Replica filtering protection makes needless short read protection requests: range reads ask each replica for the rest of a partition even when the ... |
| [#231](https://github.com/JeremiahDJordan/cassandra/issues/231) | DataStax | main-5.0 | (none) | Replica filtering protection makes needless short read protection requests: range reads ask each replica for the rest of a partition even when the ... |
| [#645](https://github.com/JeremiahDJordan/cassandra/issues/645) | DataStax | (none) | (none) | GROUP BY queries make short read protection requests to every replica when the replicas agree |

### CQL literals, bound collection values, functions and restrictions

Collection, tuple and UDT literals in the selection and as function arguments, bound collection values stored unsorted or with duplicates, zero-length elements, `ReversedType` (descending clustering columns), `IN` with null or unset elements, multi-column relations, and user-defined functions returning collections.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#829](https://github.com/JeremiahDJordan/cassandra/issues/829) | Apache | 6.0, trunk | critical | A set or map nested in a bound collection value is stored unsorted and with duplicates |
| [#842](https://github.com/JeremiahDJordan/cassandra/issues/842) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | critical | A set or map in a bound value for a descending clustering column, a tuple, a UDT or a vector is stored unsorted and with duplicates |
| [#843](https://github.com/JeremiahDJordan/cassandra/issues/843) | DataStax | main-4.0, main-5.0 | critical | A set or map in a bound value for a descending clustering column, a tuple or a UDT is stored unsorted and with duplicates |
| [#136](https://github.com/JeremiahDJordan/cassandra/issues/136) | Apache | 4.1, 5.0 | medium | IN with a null element before other elements matches nothing in filtered queries |
| [#213](https://github.com/JeremiahDJordan/cassandra/issues/213) | DataStax | main-4.0, main-5.0 | medium | Reads that select an open-ended collection slice (m[1..], s[..5]) fail with UnsupportedOperationException when they are sent to a replica, including ... |
| [#664](https://github.com/JeremiahDJordan/cassandra/issues/664) | DataStax | main-4.0, main-5.0 | medium | != on a counter column fails the read with UnsupportedOperationException |
| [#760](https://github.com/JeremiahDJordan/cassandra/issues/760) | DataStax | main-4.0 | medium | RowFilter reads a deleted static cell as an empty value on the replica |
| [#827](https://github.com/JeremiahDJordan/cassandra/issues/827) | Apache/DataStax | 4.0, 4.1, 5.0, main-4.0, main-5.0 | medium | A bound frozen list with a zero-length element is stored with a null element, and commit log replay then fails |
| [#837](https://github.com/JeremiahDJordan/cassandra/issues/837) | Apache | 6.0, trunk | medium | A zero-length element bound in IN ? is treated as null: the query fails, and an IN condition gives the wrong outcome |
| [#844](https://github.com/JeremiahDJordan/cassandra/issues/844) | Apache | 6.0, trunk | medium | Multi-column restrictions reject set and list literals, and {} for a map, on descending clustering columns |
| [#7](https://github.com/JeremiahDJordan/cassandra/issues/7) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | to_json and SELECT JSON fail with NullPointerException on a collection literal with a null element |
| [#8](https://github.com/JeremiahDJordan/cassandra/issues/8) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | Function calls with a list or map literal of another type, like count([v1, v2]), fail with a server error |
| [#9](https://github.com/JeremiahDJordan/cassandra/issues/9) | Apache | 5.0, 6.0, trunk | low | Whether function calls on nested collection literals are accepted depends on the JVM run |
| [#21](https://github.com/JeremiahDJordan/cassandra/issues/21) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | Set and map literals in the selection fail with NullPointerException when an element or a key is null |
| [#22](https://github.com/JeremiahDJordan/cassandra/issues/22) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | Collection literals in the selection accept elements of another type, such as [i, txt] or [v, (i)], and return malformed values |
| [#90](https://github.com/JeremiahDJordan/cassandra/issues/90) | DataStax | main-4.0, main-5.0 | low | CQL rejects mixing frozen and non-frozen UDT values in collection literals, casts and generic functions where Apache Cassandra accepts them |
| [#94](https://github.com/JeremiahDJordan/cassandra/issues/94) | DataStax | main-5.0 | low | mask_replace fails an assertion for two tuple literals of the same type |
| [#150](https://github.com/JeremiahDJordan/cassandra/issues/150) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | NullPointerException in multi-column relations with a tuple bound to null |
| [#159](https://github.com/JeremiahDJordan/cassandra/issues/159) | Apache | 4.1, 5.0 | low | An unset element of a filtered IN restriction matches the rows whose value is empty |
| [#169](https://github.com/JeremiahDJordan/cassandra/issues/169) | Apache | 5.0, 6.0, trunk | low | Function calls with a list literal argument are rejected with "Unexpected receiver type" when an overload of another type is tested, and a slice of ... |
| [#171](https://github.com/JeremiahDJordan/cassandra/issues/171) | DataStax | main-4.0, main-5.0 | low | List and map literals passed to a function parameter of another type give a server error, and fail overload resolution in INSERT, UPDATE and WHERE |
| [#182](https://github.com/JeremiahDJordan/cassandra/issues/182) | DataStax | main-4.0, main-5.0 | low | A filtered IN with a null element fails with NullPointerException; Apache Cassandra 5.0 ignores the null element |
| [#188](https://github.com/JeremiahDJordan/cassandra/issues/188) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | SELECT JSON and to_json fail with IndexOutOfBoundsException on a tuple value with fewer components than its type |
| [#192](https://github.com/JeremiahDJordan/cassandra/issues/192) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | List and set literals in the selection get the type of their first element even when a later element's type is wider, so SELECT [tp, tp3] drops a ... |
| [#233](https://github.com/JeremiahDJordan/cassandra/issues/233) | DataStax | main-4.0, main-5.0 | low | NullPointerException in multi-column relations with a tuple bound to null |
| [#502](https://github.com/JeremiahDJordan/cassandra/issues/502) | DataStax | main-4.0, main-5.0 | low | CONTAINS and CONTAINS KEY on a descending frozen collection clustering column are rejected (CASSANDRA-21742 missing) |
| [#507](https://github.com/JeremiahDJordan/cassandra/issues/507) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | The empty literal {} for a descending frozen map clustering column fails with ClassCastException |
| [#508](https://github.com/JeremiahDJordan/cassandra/issues/508) | DataStax | main-4.0, main-5.0 | low | The empty literal {} for a descending frozen map clustering column fails with ClassCastException |
| [#512](https://github.com/JeremiahDJordan/cassandra/issues/512) | DataStax | main-4.0 | low | Filtering with IN treats a 4-byte inet address and its IPv4-mapped form as equal on main-4.0 |
| [#643](https://github.com/JeremiahDJordan/cassandra/issues/643) | DataStax | main-4.0, main-5.0 | low | SELECT [vi, i] with a varint vi and an int i is rejected; Apache Cassandra 5.0 returns a list<varint> |
| [#709](https://github.com/JeremiahDJordan/cassandra/issues/709) | DataStax | main-5.0 | low | CREATE OR REPLACE FUNCTION/AGGREGATE that changes only DETERMINISTIC or MONOTONIC is not applied |
| [#840](https://github.com/JeremiahDJordan/cassandra/issues/840) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | A set or map returned by a Java user-defined function or aggregate is not sorted, and is used unsorted in keys, IN, conditions and element selection |
| [#841](https://github.com/JeremiahDJordan/cassandra/issues/841) | DataStax | main-4.0, main-5.0 | low | A set or map returned by a Java user-defined function or aggregate is not sorted, and is used unsorted in keys, IN, conditions and element selection |
| [#853](https://github.com/JeremiahDJordan/cassandra/issues/853) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | A user-defined function returning a collection is rejected for a descending frozen collection clustering column, and in multi-column restrictions |
| [#857](https://github.com/JeremiahDJordan/cassandra/issues/857) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | A ReversedType type hint for a map, tuple or list literal in the selection fails with a server error |

### Mixed versions and storage compatibility

DataStax nodes running beside Apache Cassandra nodes, or rolling back to them: row filter features and verbs that Apache Cassandra 5.0 replicas can't read, schema table layouts, SSTable format versions and deletion times after 2038, gossip value sizes, error codes.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#257](https://github.com/JeremiahDJordan/cassandra/issues/257) | DataStax | main-5.0 | critical | Row filter features that Apache Cassandra 5.0 can't read are sent to Apache Cassandra 5.0 replicas: OR is evaluated as AND, map entry ranges as ... |
| [#689](https://github.com/JeremiahDJordan/cassandra/issues/689) | DataStax | main-5.0 | critical | SAI range reads send the DataStax-only MULTI_RANGE_REQ verb to Apache Cassandra 5.0 replicas |
| [#739](https://github.com/JeremiahDJordan/cassandra/issues/739) | DataStax | main-5.0 | critical | DataStax 5.0 coordinators send SAI-filtered != and non-strict IN reads that Apache Cassandra 5.0 replicas ignore or refuse |
| [#814](https://github.com/JeremiahDJordan/cassandra/issues/814) | DataStax | main-5.0 | critical | BTI da sstables encode a null field of a tuple, UDT or frozen collection key differently from Apache Cassandra 5.0: reads of the other ... |
| [#96](https://github.com/JeremiahDJordan/cassandra/issues/96) | DataStax | main-5.0 | high | Nodes cannot start with a table that has a duration in a sorted position (CNDB-9753 compatibility mode missing) |
| [#649](https://github.com/JeremiahDJordan/cassandra/issues/649) | DataStax | main-5.0 | high | BTI cc sstables record a max local deletion time past 2038 as 2038, so compaction drops them whole in 2038 while their data is live |
| [#662](https://github.com/JeremiahDJordan/cassandra/issues/662) | DataStax | main-5.0 | high | BTI writes "cc" in every storage compatibility mode, so partition deletions after 2038 are purgeable at once |
| [#677](https://github.com/JeremiahDJordan/cassandra/issues/677) | DataStax | main-5.0 | high | ANN queries with synthetic scores fail on Apache Cassandra 5.0 replicas, which can't deserialize the score column |
| [#740](https://github.com/JeremiahDJordan/cassandra/issues/740) | DataStax | main-5.0 | high | DataStax 5.0 coordinators send SAI-filtered CONTAINS on frozen collections that Apache Cassandra 5.0 replicas evaluate on the whole collection |
| [#816](https://github.com/JeremiahDJordan/cassandra/issues/816) | DataStax | main-5.0 | medium | A node bootstrapping beside Apache Cassandra 5.0 nodes can fail schema readiness: a peer's schema version that becomes the local version by merging ... |
| [#650](https://github.com/JeremiahDJordan/cassandra/issues/650) | Apache | 5.0, 6.0, trunk | low | Droppable tombstone estimate ignores deletion times after 2038: TombstoneHistogram#sum narrows gcBefore to int |
| [#665](https://github.com/JeremiahDJordan/cassandra/issues/665) | DataStax | main-5.0 | low | SAI index status gossip format is chosen by release version; the legacy format can exceed the gossip value limit |
| [#668](https://github.com/JeremiahDJordan/cassandra/issues/668) | DataStax | main-4.0 | low | SAI index status gossiped by 4.0 can exceed the 65535-byte limit of a gossiped value |
| [#675](https://github.com/JeremiahDJordan/cassandra/issues/675) | DataStax | main-5.0 | low | CASSANDRA_4: system_schema.functions and aggregates carry DataStax-only columns that Apache Cassandra 4.x can't read, blocking a rollback |
| [#678](https://github.com/JeremiahDJordan/cassandra/issues/678) | DataStax | main-4.0, main-5.0 | low | Vector types of tuples and nested frozen types are stored in system_schema differently from Apache Cassandra 5.0 |
| [#679](https://github.com/JeremiahDJordan/cassandra/issues/679) | DataStax | main-4.0, main-5.0 | low | Unknown table and column failures reach Apache Cassandra coordinators as UNKNOWN instead of INCOMPATIBLE_SCHEMA |
| [#692](https://github.com/JeremiahDJordan/cassandra/issues/692) | DataStax | main-5.0 | low | CASSANDRA_4: system_schema.tables and views store the memtable option in the CC4 map layout, which Apache Cassandra reads as an invalid configuration |
| [#755](https://github.com/JeremiahDJordan/cassandra/issues/755) | DataStax | main-4.0, main-5.0 | low | After a rolling upgrade, coordinators keep refusing features that need a newer messaging version until their idle connections reconnect |
| [#652](https://github.com/JeremiahDJordan/cassandra/issues/652) | DataStax | main-5.0 | (none) | Nodes refuse SSTable versions they can read when the configured write version is older (e.g. bti da while writing cc) |

### SAI resources, metrics, options and documentation

Leaked iterators and file handles, cache pollution by index builds, needless work in compaction, metrics and guardrail counting, option validation, and documentation that describes the wrong defaults or operators.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#605](https://github.com/JeremiahDJordan/cassandra/issues/605) | DataStax | main-4.0, main-5.0 | high | SAI: an analyzer config without a tokenizer is rejected with "CQL type text cannot be analyzed" |
| [#267](https://github.com/JeremiahDJordan/cassandra/issues/267) | DataStax | main-4.0, main-5.0 | medium | SAI leaks the partition iterator or the posting lists of a query that times out |
| [#270](https://github.com/JeremiahDJordan/cassandra/issues/270) | DataStax | main-4.0, main-5.0 | low | SAI rejects valid index options on JVMs whose default locale is Turkish or Azeri |
| [#274](https://github.com/JeremiahDJordan/cassandra/issues/274) | DataStax | main-5.0 | low | sai_sstable_indexes_per_query counts the SSTable indexes of all the expressions of a query together, so it warns about and fails queries too early |
| [#278](https://github.com/JeremiahDJordan/cassandra/issues/278) | DataStax | main-4.0, main-5.0 | low | SAI queries don't count the tombstones of the rows they read and drop, so tombstone thresholds, metrics and tracing don't apply to them |
| [#364](https://github.com/JeremiahDJordan/cassandra/issues/364) | DataStax | main-4.0, main-5.0 | low | SAI: IndexFileUtils.openBlockingInput leaks the FileHandle when it can't create the reader |
| [#365](https://github.com/JeremiahDJordan/cassandra/issues/365) | DataStax | main-5.0 | low | nodetool import --require-index-components leaks a reader per imported SSTable and caches its SAI descriptor |
| [#366](https://github.com/JeremiahDJordan/cassandra/issues/366) | DataStax | main-4.0, main-5.0 | low | SAI index writers log aborts without a cause, such as an interrupted compaction, at WARN |
| [#466](https://github.com/JeremiahDJordan/cassandra/issues/466) | DataStax | main-4.0, main-5.0 | low | A class named in SAI analyzer options is initialized before its type is checked |
| [#470](https://github.com/JeremiahDJordan/cassandra/issues/470) | DataStax | main-4.0, main-5.0 | low | The index hints serializer asserts fewer than 32767 indexes, which validation accepts |
| [#609](https://github.com/JeremiahDJordan/cassandra/issues/609) | DataStax | main-4.0, main-5.0 | low | SAI: no n-gram client warning when the n-gram tokenizer or filter is named "nGram" |
| [#362](https://github.com/JeremiahDJordan/cassandra/issues/362) | DataStax | main-5.0 | (none) | Compaction reads every fully expired SSTable in full on tables with an SAI index (or NoopIndex) |
| [#367](https://github.com/JeremiahDJordan/cassandra/issues/367) | DataStax | main-5.0 | (none) | conf/cassandra.yaml and NEWS.txt say sai_options.prioritize_over_legacy_index defaults to false, but it defaults to true |
| [#450](https://github.com/JeremiahDJordan/cassandra/issues/450) | DataStax | main-5.0 | (none) | Cache RowFilter#hasStaticExpression, which copies the filter's expressions on each call |
| [#459](https://github.com/JeremiahDJordan/cassandra/issues/459) | DataStax | main-4.0, main-5.0 | (none) | SAI index builds, verify, import and filter rebuilds fill the chunk cache with the sstable they read |
| [#460](https://github.com/JeremiahDJordan/cassandra/issues/460) | Apache | 5.0, 6.0, trunk | (none) | SAI index builds still fill the chunk cache and the key cache through a per-key position lookup |
| [#461](https://github.com/JeremiahDJordan/cassandra/issues/461) | DataStax | main-5.0 | (none) | SecondaryIndexManager builds the indexers of every write to an indexed table with a stream |
| [#462](https://github.com/JeremiahDJordan/cassandra/issues/462) | DataStax | main-4.0, main-5.0 | (none) | The SAI README describes DSE's SAI: "for DSE", a proprietary trie, RLAC, only token ordering |
| [#463](https://github.com/JeremiahDJordan/cassandra/issues/463) | DataStax | main-5.0 | (none) | The SAI documentation lists only =, ranges, CONTAINS and AND as supported operators, and LIKE as not supported |

### Trie memtable and byte-comparable values

DataStax trie memtables: writes during schema changes, deletions with equal timestamps, flushes that skip or drop data, size accounting, LWT stats, and decimal values whose scale changes through the byte-comparable encoding.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#112](https://github.com/JeremiahDJordan/cassandra/issues/112) | DataStax | main-5.0 | blocker | Trie memtable stores a write in flight during a schema change under the wrong columns |
| [#114](https://github.com/JeremiahDJordan/cassandra/issues/114) | DataStax | main-5.0 | blocker | Trie memtable rejects a write to a column that is being added |
| [#215](https://github.com/JeremiahDJordan/cassandra/issues/215) | DataStax | main-5.0 | blocker | TrieMemtableStage3: after a partition deletion, flush can skip the partition, so deleted rows come back and a row written with the deletion is lost |
| [#25](https://github.com/JeremiahDJordan/cassandra/issues/25) | DataStax | main-5.0 | critical | LWTs can lose updates on tables with trie memtables: withUpdatedTimestamps keeps the old stats |
| [#217](https://github.com/JeremiahDJordan/cassandra/issues/217) | DataStax | main-5.0 | critical | Trie memtable with an index: a deletion that overlaps an existing deletion fails the write, and commit log replay of it stops the node |
| [#218](https://github.com/JeremiahDJordan/cassandra/issues/218) | DataStax | main-4.0, main-5.0 | critical | A write that a trie memtable fails to apply still reaches the indexes, so SAI and legacy indexes miss live rows |
| [#220](https://github.com/JeremiahDJordan/cassandra/issues/220) | DataStax | main-5.0 | critical | SAI keeps the old values of a row that a range or partition deletion removed from a trie memtable, so a row written again is found by its old text ... |
| [#686](https://github.com/JeremiahDJordan/cassandra/issues/686) | DataStax | main-4.0, main-5.0 | critical | Trie memtables change the scale of decimal clustering keys, set elements and map keys (and other values with equal byte-comparable forms) |
| [#214](https://github.com/JeremiahDJordan/cassandra/issues/214) | DataStax | main-5.0 | high | Trie memtable: a paged query that resumes after the last selected row of a partition fails with ReadFailure ("... is not a subset of ...") |
| [#216](https://github.com/JeremiahDJordan/cassandra/issues/216) | DataStax | main-5.0 | high | Trie memtable: a write that fails while it is merged can leave the partition counters at zero, and flush then drops the partition's live rows |
| [#219](https://github.com/JeremiahDJordan/cassandra/issues/219) | DataStax | main-4.0, main-5.0 | high | Trie memtable live data size goes negative after some deletion sequences, and SAI queries then fail with "matchingKeyCount must not be negative" |
| [#35](https://github.com/JeremiahDJordan/cassandra/issues/35) | DataStax | main-5.0 | medium | TrieMemtable's row count estimate is biased low, which skews SAI query planning and the EstimatedRowCount metric |
| [#36](https://github.com/JeremiahDJordan/cassandra/issues/36) | DataStax | main-5.0 | medium | A write carrying a cell of a dropped column fails on tables with a trie memtable: Column c not present in row columns set |
| [#228](https://github.com/JeremiahDJordan/cassandra/issues/228) | DataStax | main-5.0 | low | Trie memtable returns a frozen list or set map key with a null element in place of an empty one; SAI ENTRIES index compaction and rebuild then fail ... |
| [#593](https://github.com/JeremiahDJordan/cassandra/issues/593) | DataStax | main-5.0 | low | trie_stage2 memtable: a row marker or row deletion added to an existing row is not counted on-heap |
| [#619](https://github.com/JeremiahDJordan/cassandra/issues/619) | Apache | 5.0, 6.0, trunk | low | DecimalType byte-comparable decoding turns a decimal with a scale of Integer.MAX_VALUE into one with a scale of Integer.MIN_VALUE |
| [#621](https://github.com/JeremiahDJordan/cassandra/issues/621) | DataStax | main-4.0, main-5.0 | low | Trie memtables return and flush wrong decimal clustering keys, set elements and map keys for a scale of Integer.MAX_VALUE |

### Compaction, SSTables and tools

Compaction and scrub with dropped columns, cursor compaction, expired TTL row markers, snapshot names, tombstone compaction selection, incremental repair finalization, and the sstable tools on legacy SSTables.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#2](https://github.com/JeremiahDJordan/cassandra/issues/2) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | blocker | Compaction output header can give a column an earlier type than its data, and scrub then removes the rows |
| [#824](https://github.com/JeremiahDJordan/cassandra/issues/824) | DataStax | main-4.0, main-5.0 | blocker | Cursor compaction of a static row and a range deletion from the partition start in different sstables fails on 5.0 and writes a corrupt sstable on 4.0 |
| [#130](https://github.com/JeremiahDJordan/cassandra/issues/130) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | critical | A row kept alive by cells older than its expired TTL row marker is missing from selections that don't read those cells |
| [#232](https://github.com/JeremiahDJordan/cassandra/issues/232) | DataStax | main-4.0, main-5.0 | critical | A row kept alive by cells older than its expired TTL row marker is missing from selections that don't read those cells |
| [#695](https://github.com/JeremiahDJordan/cassandra/issues/695) | DataStax | main-4.0, main-5.0 | high | Parallel major compactions with snapshot_before_compaction fail: their tasks pick the same snapshot name |
| [#728](https://github.com/JeremiahDJordan/cassandra/issues/728) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | high | snapshot_before_compaction: compactions of a table that start in the same millisecond take the same snapshot name and fail |
| [#37](https://github.com/JeremiahDJordan/cassandra/issues/37) | DataStax | main-4.0, main-5.0 | medium | sstabledump fails with CorruptSSTableException on legacy sstables whose header records a frozen tuple without a frozen wrapper |
| [#58](https://github.com/JeremiahDJordan/cassandra/issues/58) | DataStax | main-4.0, main-5.0 | medium | TRUNCATE during upgradesstables can wait for the whole upgrade instead of interrupting it |
| [#116](https://github.com/JeremiahDJordan/cassandra/issues/116) | DataStax | main-5.0 | medium | Single-sstable tombstone compaction never chooses an sstable that overlaps another (inverted estimate check) |
| [#579](https://github.com/JeremiahDJordan/cassandra/issues/579) | DataStax | main-4.0, main-5.0 | medium | Incremental repair finalization can demote just-repaired sstables or drop sstables from a new session |
| [#43](https://github.com/JeremiahDJordan/cassandra/issues/43) | DataStax | main-5.0 | low | Every offline open of a BIG sstable by a standalone tool warns about a pass-through bloom filter |
| [#84](https://github.com/JeremiahDJordan/cassandra/issues/84) | DataStax | main-5.0 | low | garbagecollect and sstable splitting fail with a NullPointerException instead of "Not enough space for compaction" |
| [#120](https://github.com/JeremiahDJordan/cassandra/issues/120) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | sstabledump, sstablemetadata and sstablepartitions fail with EOFException on 3.x sstables whose header does not mark frozen user type columns as ... |
| [#122](https://github.com/JeremiahDJordan/cassandra/issues/122) | DataStax | main-4.0, main-5.0 | low | sstabledump, sstablemetadata -s and sstablepartitions fail on legacy 3.0-written sstables with frozen user types |
| [#750](https://github.com/JeremiahDJordan/cassandra/issues/750) | DataStax | main-4.0, main-5.0 | low | Cache invalidation of an sstable file removed after its exists() check logs NoSuchFileException at ERROR |
| [#441](https://github.com/JeremiahDJordan/cassandra/issues/441) | DataStax | main-5.0 | (none) | Compaction reads compressed sstables one chunk per read: the compressed read-ahead buffer is never used |

### Cluster metadata (TCM) and upgrades to 6.0

Cassandra 6.0 cluster metadata: snapshots that lose table parameters, schema statements replayed from the metadata log with other guardrail results, CMS initialization during an upgrade, hints and Accord migration.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#666](https://github.com/JeremiahDJordan/cassandra/issues/666) | Apache | 6.0, trunk | high | A new node started during an upgrade before CMS initialization can initialize the CMS with its empty metadata, unregistering the upgraded nodes and ... |
| [#729](https://github.com/JeremiahDJordan/cassandra/issues/729) | Apache | 6.0, trunk | high | Commit log replay fails for writes to a table that has started migrating to Accord since they were written, and the node doesn't start |
| [#654](https://github.com/JeremiahDJordan/cassandra/issues/654) | Apache | 6.0, trunk | medium | Cluster metadata snapshots don't keep a table's auto_repair settings: TableParams.Serializer omits them |
| [#655](https://github.com/JeremiahDJordan/cassandra/issues/655) | Apache | 6.0, trunk | medium | CREATE TABLE ... WITH auto_repair loses the setting when the coordinator has auto-repair scheduling disabled: the expanded CQL leaves it out |
| [#656](https://github.com/JeremiahDJordan/cassandra/issues/656) | Apache | 6.0, trunk | medium | A node can't load a cluster metadata snapshot with a table whose memtable configuration its cassandra.yaml doesn't define: TableParams.Serializer ... |
| [#658](https://github.com/JeremiahDJordan/cassandra/issues/658) | Apache | 6.0, trunk | medium | ALTER TABLE applies an option removed by table_properties_ignored on nodes that replay it from the metadata log |
| [#667](https://github.com/JeremiahDJordan/cassandra/issues/667) | Apache | 6.0, trunk | medium | Before CMS initialization, upgraded nodes address hints with node ids, so a node still on 5.0 stores them instead of applying them |
| [#720](https://github.com/JeremiahDJordan/cassandra/issues/720) | Apache | 6.0, trunk | medium | Hints rejected for the other transaction system are lost: a second rejection counts as delivered, and the retry picks hints by index |
| [#722](https://github.com/JeremiahDJordan/cassandra/issues/722) | Apache | 6.0, trunk | medium | A hint at messaging version 14 for a dropped table fails on every delivery with CoordinatorBehindException, so hint delivery to that node stops ... |
| [#724](https://github.com/JeremiahDJordan/cassandra/issues/724) | Apache | 6.0, trunk | medium | ALTER MATERIALIZED VIEW runs the table_properties guardrail again on replay from the metadata log, with the replaying node's settings |
| [#776](https://github.com/JeremiahDJordan/cassandra/issues/776) | Apache | 6.0, trunk | medium | ALTER KEYSPACE applies an option removed by keyspace_properties_ignored on nodes that replay it from the metadata log |
| [#777](https://github.com/JeremiahDJordan/cassandra/issues/777) | Apache | 6.0, trunk | medium | CREATE TABLE ... LIKE applies an option removed by table_properties_ignored on nodes that replay it from the metadata log |
| [#778](https://github.com/JeremiahDJordan/cassandra/issues/778) | Apache | 6.0, trunk | medium | A node that was down while every CMS member it knows left can't fetch the metadata log or commit: RemoteProcessor only contacts the stored CMS members |
| [#657](https://github.com/JeremiahDJordan/cassandra/issues/657) | Apache | 6.0, trunk | low | Directory fails with "Unsupported metadata version" when every node that hasn't left has a later metadata version |
| [#779](https://github.com/JeremiahDJordan/cassandra/issues/779) | Apache | 6.0, trunk | low | A bootstrapping node serves top-k reads and reads with transaction conflicts allowed: the replica-side ownership check skips them |

### only_purge_repaired_tombstones

Tables with `only_purge_repaired_tombstones`: reads, row cache hits, index searches and the repaired data digest that purge unrepaired tombstones or expired cells, and legacy index tables that never purge.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#448](https://github.com/JeremiahDJordan/cassandra/issues/448) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | high | Row cache hits and index searches purge unrepaired tombstones on tables with only_purge_repaired_tombstones |
| [#449](https://github.com/JeremiahDJordan/cassandra/issues/449) | DataStax | main-4.0, main-5.0 | high | Row cache hits and index searches purge unrepaired tombstones on tables with only_purge_repaired_tombstones |
| [#523](https://github.com/JeremiahDJordan/cassandra/issues/523) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | high | Reads purge expired cells of unrepaired data on tables with only_purge_repaired_tombstones |
| [#524](https://github.com/JeremiahDJordan/cassandra/issues/524) | DataStax | main-4.0, main-5.0 | high | Reads purge expired cells of unrepaired data on tables with only_purge_repaired_tombstones |
| [#525](https://github.com/JeremiahDJordan/cassandra/issues/525) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | high | Range reads purge deletions written into the memtable after they opened it on tables with only_purge_repaired_tombstones |
| [#526](https://github.com/JeremiahDJordan/cassandra/issues/526) | DataStax | main-4.0, main-5.0 | high | Range reads purge deletions written into the memtable after they opened it on tables with only_purge_repaired_tombstones |
| [#543](https://github.com/JeremiahDJordan/cassandra/issues/543) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | medium | Legacy secondary index tables never purge tombstones when the base table has only_purge_repaired_tombstones |
| [#544](https://github.com/JeremiahDJordan/cassandra/issues/544) | DataStax | main-4.0, main-5.0 | medium | Legacy secondary index tables never purge tombstones when the base table has only_purge_repaired_tombstones |
| [#443](https://github.com/JeremiahDJordan/cassandra/issues/443) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | Memtables report the 2015 encoding epoch as their min local deletion time, so reads with only_purge_repaired_tombstones purge no tombstones |
| [#444](https://github.com/JeremiahDJordan/cassandra/issues/444) | DataStax | main-4.0, main-5.0 | low | Memtable data reports the 2015 encoding epoch as its min local deletion time, so reads with only_purge_repaired_tombstones purge no tombstones |
| [#549](https://github.com/JeremiahDJordan/cassandra/issues/549) | Apache | (none) | low | Repaired data digest of a read of a table with only_purge_repaired_tombstones can purge a tombstone the response keeps |
| [#550](https://github.com/JeremiahDJordan/cassandra/issues/550) | DataStax | (none) | low | Repaired data digest of a read of a table with only_purge_repaired_tombstones can purge a tombstone the response keeps |

### Schema changes, dropped columns and user types

Cells of dropped columns newer than the drop, column renames, vector columns of user types, schema agreement and table option guardrails.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#27](https://github.com/JeremiahDJordan/cassandra/issues/27) | DataStax | main-5.0 | critical | Schema changes are acknowledged before system.local has the new schema version, so drivers see schema agreement too early |
| [#710](https://github.com/JeremiahDJordan/cassandra/issues/710) | Apache | 5.0, 6.0, trunk | critical | Vector columns of user types: dropping one stores a type no node can load, DROP TYPE and ALTER TYPE ignore them |
| [#680](https://github.com/JeremiahDJordan/cassandra/issues/680) | DataStax | main-4.0 | high | main-4.0: dropping a vector<frozen<udt>> column, or its UDT, leaves a schema that can't be loaded |
| [#1](https://github.com/JeremiahDJordan/cassandra/issues/1) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | medium | NullPointerException in Row.Merger when a dropped complex column has cells newer than the drop |
| [#3](https://github.com/JeremiahDJordan/cassandra/issues/3) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | medium | Partition and range reads fail with ReadFailure when a dropped column has cells newer than the drop |
| [#4](https://github.com/JeremiahDJordan/cassandra/issues/4) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | medium | Renaming a primary key column to the name of a dropped column makes sstables fail to open |
| [#125](https://github.com/JeremiahDJordan/cassandra/issues/125) | DataStax | main-4.0, main-5.0 | medium | NullPointerException in Row.Merger when a dropped complex column has cells newer than the drop |
| [#640](https://github.com/JeremiahDJordan/cassandra/issues/640) | DataStax | main-4.0, main-5.0 | medium | Partition and range reads fail with ReadFailure when a dropped column has cells newer than the drop |
| [#659](https://github.com/JeremiahDJordan/cassandra/issues/659) | Apache | 4.1, 5.0, 6.0, trunk | medium | CREATE TABLE and ALTER TABLE with an option in table_properties_ignored and another option fail with ConcurrentModificationException |
| [#711](https://github.com/JeremiahDJordan/cassandra/issues/711) | DataStax | main-5.0 | medium | ALTER TYPE of a user type used in a vector column fails with "Column mismatch" |
| [#100](https://github.com/JeremiahDJordan/cassandra/issues/100) | DataStax | main-5.0 | low | Dropped columns with cells newer than the drop: scrub removes rows and cursor compaction fails |

### Legacy secondary indexes

Legacy (table-based) secondary indexes on static columns, with TTL changes, `DROP INDEX` during use, and index implementation selection.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#163](https://github.com/JeremiahDJordan/cassandra/issues/163) | Apache | 5.0, 6.0, trunk | critical | Legacy index on a static column misses rows in single-partition queries with a clustering restriction or paging |
| [#427](https://github.com/JeremiahDJordan/cassandra/issues/427) | DataStax | main-4.0, main-5.0 | critical | Legacy index on a static column misses rows in single-partition queries with a clustering restriction or paging |
| [#663](https://github.com/JeremiahDJordan/cassandra/issues/663) | DataStax | main-4.0, main-5.0 | critical | Legacy secondary index loses a row when a replica receives the same value with the same timestamp and a new TTL |
| [#363](https://github.com/JeremiahDJordan/cassandra/issues/363) | DataStax | main-5.0 | medium | UCS drops fully expired SSTables without notifying secondary indexes about their rows |
| [#423](https://github.com/JeremiahDJordan/cassandra/issues/423) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | medium | A write whose index fails in Indexer#finish leaves a skip-list memtable that fails to flush ("[c] is not a subset of [...]") |
| [#424](https://github.com/JeremiahDJordan/cassandra/issues/424) | DataStax | main-4.0, main-5.0 | medium | A write whose index fails in Indexer#finish leaves a memtable that fails to flush ("[c] is not a subset of [...]") or flushes the row without its ... |
| [#592](https://github.com/JeremiahDJordan/cassandra/issues/592) | DataStax | main-4.0, main-5.0 | medium | CREATE CUSTOM INDEX and the Encryptor key_provider option initialize the named class before checking its type |
| [#708](https://github.com/JeremiahDJordan/cassandra/issues/708) | DataStax | main-5.0 | medium | CREATE INDEX ... USING 'legacy_local_table' creates an index of cassandra.default_index_implementation_class |

### Auto-repair

Auto-repair parameters accepted but ignored, priority hosts, host ids after an upgrade to 6.0, and history rows of removed nodes.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#590](https://github.com/JeremiahDJordan/cassandra/issues/590) | Apache | 5.0 | medium | AutoRepair: an unknown priority host is stored as an empty host id and stops auto-repair of that type |
| [#697](https://github.com/JeremiahDJordan/cassandra/issues/697) | Apache | 5.0, 6.0, trunk | medium | auto_repair keys that aren't lower case are accepted and ignored: AutoRepairParams keeps the given case next to the defaults |
| [#856](https://github.com/JeremiahDJordan/cassandra/issues/856) | Apache | 6.0, trunk | medium | Auto-repair uses pre-upgrade host ids after an upgrade to 6.0: priority hosts are lost and upgraded nodes fail the replica check |
| [#859](https://github.com/JeremiahDJordan/cassandra/issues/859) | Apache | 5.0, 6.0, trunk | medium | Auto-repair stops when the history row of a removed node is the oldest and one or two nodes are left: its deletion vote can't pass |
| [#706](https://github.com/JeremiahDJordan/cassandra/issues/706) | Apache | 5.0, 6.0, trunk | low | auto_repair priority outside the int range is accepted and then fails auto-repair planning |
| [#858](https://github.com/JeremiahDJordan/cassandra/issues/858) | Apache | 6.0, trunk | low | Auto-repair host id migration on upgrade to 6.0 drops a pending forced repair and deletes rows whose copy failed |

### Client protocol and buffers

Native protocol STARTUP handling, queue timeout responses, client metrics and the buffer pool allocator.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#554](https://github.com/JeremiahDJordan/cassandra/issues/554) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | high | BufferPoolAllocator.Wrapped keeps a view without a cleaner after a resize into overflow memory, so release frees nothing or throws |
| [#455](https://github.com/JeremiahDJordan/cassandra/issues/455) | DataStax | main-5.0 | medium | Native transport queue timeout response is sent on stream 0 |
| [#555](https://github.com/JeremiahDJordan/cassandra/issues/555) | DataStax | main-4.0, main-5.0 | medium | BufferPoolAllocator.Wrapped keeps a view without a cleaner after a resize into overflow memory, so release doesn't free it |
| [#456](https://github.com/JeremiahDJordan/cassandra/issues/456) | DataStax | main-5.0 | low | CNDB-11070 backport repeats CASSANDRA-19534 checks; Client.Queued counts each request twice |
| [#831](https://github.com/JeremiahDJordan/cassandra/issues/831) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | A rejected protocol V5 STARTUP leaks a reference to the client address's allocator, and a failed STARTUP response write leaks its bytes |
| [#834](https://github.com/JeremiahDJordan/cassandra/issues/834) | DataStax | main-4.0, main-5.0 | low | A V5 STARTUP that is rejected, or whose response can't be written, keeps the client address's allocator reference and the STARTUP's in-flight bytes |

### Hints

Hints for tables a node doesn't know or has dropped, and hints during decommission.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#67](https://github.com/JeremiahDJordan/cassandra/issues/67) | DataStax | main-4.0 | high | Decommission fails with "Failed to transfer all hints" when the node holds hints for a down node |
| [#721](https://github.com/JeremiahDJordan/cassandra/issues/721) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | medium | A node that doesn't know a hint's table yet acknowledges the hint, and the sender deletes it |
| [#731](https://github.com/JeremiahDJordan/cassandra/issues/731) | DataStax | main-4.0, main-5.0 | medium | A node that doesn't know a hint's table yet acknowledges the hint, and the sender deletes it |
| [#723](https://github.com/JeremiahDJordan/cassandra/issues/723) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | A hint for several tables loses all of its updates when one of the tables has been dropped |

### Commit log and durability

Direct I/O commit log flushes on ZFS and writes lost when a table changes its memtable configuration.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#565](https://github.com/JeremiahDJordan/cassandra/issues/565) | Apache | 5.0, 6.0, trunk | high | Direct I/O commit log flush fails with EIO on ZFS when mutations are written into its last block during the write |
| [#566](https://github.com/JeremiahDJordan/cassandra/issues/566) | DataStax | main-5.0 | high | Direct I/O commit log flush fails with EIO on ZFS when mutations are written into its last block during the write |
| [#660](https://github.com/JeremiahDJordan/cassandra/issues/660) | Apache | 4.1, 5.0, 6.0, trunk | low | Changing a table to a memtable with durable writes lets the commit log drop writes of the old memtable before it is flushed |
| [#661](https://github.com/JeremiahDJordan/cassandra/issues/661) | Apache | 4.1, 5.0, 6.0, trunk | low | A write can reach a memtable that relies on the commit log without a commit log entry when the table's memtable stops skipping the commit log |

### Gossip, membership, reads and other

Gossip and peer metadata, read timeouts and speculative retry, row cache, materialized views, encryption options and Paxos cleanup.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#59](https://github.com/JeremiahDJordan/cassandra/issues/59) | DataStax | main-5.0 | high | Node and peer metadata bypass Nodes in gossip, Paxos and startup; removenode leaves a ghost peer on the coordinator |
| [#539](https://github.com/JeremiahDJordan/cassandra/issues/539) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | high | A partially cached partition whose cached rows end inside a range deletion fails the read with an illegal range tombstone bounds sequence |
| [#540](https://github.com/JeremiahDJordan/cassandra/issues/540) | DataStax | main-4.0, main-5.0 | high | A partially cached partition whose cached rows end inside a range deletion fails the read with an illegal range tombstone bounds sequence |
| [#624](https://github.com/JeremiahDJordan/cassandra/issues/624) | DataStax | main-4.0, main-5.0 | medium | Encryptor: default key provider is not a key provider factory, so compression options without key_provider always fail |
| [#694](https://github.com/JeremiahDJordan/cassandra/issues/694) | DataStax | main-4.0, main-5.0 | medium | Reads wait out their timeout after a replica failure when the coordinator won't send a speculative retry |
| [#732](https://github.com/JeremiahDJordan/cassandra/issues/732) | Apache | 4.0, 4.1, 5.0 | medium | A node that comes back at the same address on another storage port loses most of its system.peers row |
| [#733](https://github.com/JeremiahDJordan/cassandra/issues/733) | DataStax | main-5.0 | medium | A peer that comes back at the same address on another storage port loses its system.peers row |
| [#753](https://github.com/JeremiahDJordan/cassandra/issues/753) | Apache | 4.1, 5.0, 6.0, trunk | medium | Paxos v2 cleanup drops its prepare request when the local gossip state changes while the request is sent |
| [#5](https://github.com/JeremiahDJordan/cassandra/issues/5) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | Inbound handshake timeout throws NullPointerException and leaves the connection open when no handshake message was decoded |
| [#28](https://github.com/JeremiahDJordan/cassandra/issues/28) | DataStax | main-5.0 | low | The shutdown announcement of a replacing node that cannot reach a seed is never delivered: GOSSIP_SHUTDOWN without a GossipShutdown payload |
| [#614](https://github.com/JeremiahDJordan/cassandra/issues/614) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | A write to a table with a materialized view times out instead of failing when its retry after a view lock miss throws |
| [#616](https://github.com/JeremiahDJordan/cassandra/issues/616) | DataStax | main-4.0, main-5.0 | low | A write to a table with a materialized view times out instead of failing when its retry after a view lock miss throws |
| [#691](https://github.com/JeremiahDJordan/cassandra/issues/691) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | Range reads count each batch's rows twice when they size the next batch |
| [#73](https://github.com/JeremiahDJordan/cassandra/issues/73) | DataStax | main-5.0 | (none) | NEWS.txt does not document that speculative retry timing differs from HCD 1.2 |

### Test harness, build and CI

Fixes to tests and test frameworks (in-JVM dtest paging, StreamingTest hangs, flaky tests, files left behind), build targets, packaging and CI.

| Issue | Code | Branches | Severity | Title |
|---|---|---|---|---|
| [#64](https://github.com/JeremiahDJordan/cassandra/issues/64) | DataStax | main-4.0, main-5.0 | medium | cqlsh tests never activate their virtualenv, and the deb/rpm packages expect the Apache Cassandra artifact names |
| [#41](https://github.com/JeremiahDJordan/cassandra/issues/41) | DataStax | main-5.0 | low | Unit tests fail under the latest test configuration (testclasslist-latest) |
| [#44](https://github.com/JeremiahDJordan/cassandra/issues/44) | DataStax | main-4.0, main-5.0 | low | StreamingTest mixed-messaging-version tests time out waiting for schema agreement |
| [#45](https://github.com/JeremiahDJordan/cassandra/issues/45) | DataStax | main-4.0, main-5.0 | low | ColumnFamilyStoreTest can hang until the class timeout: testRWCDLocking can leave the operations lock held |
| [#46](https://github.com/JeremiahDJordan/cassandra/issues/46) | DataStax | main-5.0 | low | ant testclasslist-bigtable and test-bigtable fail with a NullPointerException: the testlist-bigtable macro is missing |
| [#47](https://github.com/JeremiahDJordan/cassandra/issues/47) | DataStax | main-4.0, main-5.0 | low | GeoDistanceAccuracyTest.haversineBenchmark is flaky on a loaded host |
| [#52](https://github.com/JeremiahDJordan/cassandra/issues/52) | DataStax | main-4.0, main-5.0 | low | VectorSiftSmallTest.testCompaction fails about one run in three on its recall@1 bound |
| [#53](https://github.com/JeremiahDJordan/cassandra/issues/53) | DataStax | main-5.0 | low | MutatorVetoTest paxos repair refresh tests fail when the replica that committed answers last |
| [#56](https://github.com/JeremiahDJordan/cassandra/issues/56) | DataStax | main-4.0, main-5.0 | low | UnifiedCompactionStrategyTest.testDropExpiredFromBucket fails when two calls land in the same millisecond |
| [#69](https://github.com/JeremiahDJordan/cassandra/issues/69) | DataStax | main-4.0, main-5.0 | low | LongBM25Test cannot run: its corpus is not in the tree (and on 5.0 it runs out of heap) |
| [#74](https://github.com/JeremiahDJordan/cassandra/issues/74) | DataStax | main-5.0 | low | AbstractTypeTest upgrade and downgrade tests are all ignored |
| [#92](https://github.com/JeremiahDJordan/cassandra/issues/92) | DataStax | main-5.0 | low | simulator-dtest fails or stalls on DataStax code (ShortPaxosSimulationTest, TrivialSimulationTest, MonitorMethodTransformerTest) |
| [#98](https://github.com/JeremiahDJordan/cassandra/issues/98) | DataStax | main-4.0, main-5.0 | low | ConnectionTest and ProxyHandlerConnectionsTest run into the class timeout |
| [#103](https://github.com/JeremiahDJordan/cassandra/issues/103) | DataStax | main-4.0, main-5.0 | low | DataStax microbenchmarks fail when they are run (microbench-test) |
| [#118](https://github.com/JeremiahDJordan/cassandra/issues/118) | DataStax | main-5.0 | low | LegacySSTableTest.testVerifyOldDroppedTupleSSTables fails with BIG: dropped tuple columns re-added with other types |
| [#138](https://github.com/JeremiahDJordan/cassandra/issues/138) | Apache | 4.1, 5.0, 6.0, trunk | low | AbstractCommitLogSegmentManager.stopUnsafe does not wait for the segment allocator when assertions are disabled |
| [#139](https://github.com/JeremiahDJordan/cassandra/issues/139) | Apache | 4.1, 5.0, 6.0, trunk | low | In-JVM dtest executeWithPaging fails with "calling without calling .init() first" when a replica reports a read warning |
| [#140](https://github.com/JeremiahDJordan/cassandra/issues/140) | Apache | 5.0, 6.0, trunk | low | In-JVM dtest executeWithPaging returns the columns the server adds for ordering |
| [#165](https://github.com/JeremiahDJordan/cassandra/issues/165) | Apache | 4.0, 4.1, 5.0 | low | In-JVM dtest paged queries execute the first page twice |
| [#229](https://github.com/JeremiahDJordan/cassandra/issues/229) | DataStax | main-5.0 | low | AbstractCommitLogSegmentManager.stopUnsafe does not wait for the segment allocator when assertions are disabled |
| [#230](https://github.com/JeremiahDJordan/cassandra/issues/230) | DataStax | main-5.0 | low | In-JVM dtest executeWithPaging fails with "calling without calling .init() first" when a replica reports a read warning |
| [#234](https://github.com/JeremiahDJordan/cassandra/issues/234) | DataStax | main-4.0, main-5.0 | low | In-JVM dtest executeWithPaging returns the columns the server adds for ordering |
| [#235](https://github.com/JeremiahDJordan/cassandra/issues/235) | DataStax | main-4.0, main-5.0 | low | In-JVM dtest paged queries execute the first page twice |
| [#244](https://github.com/JeremiahDJordan/cassandra/issues/244) | DataStax | main-5.0 | low | RepairErrorsTest.testNoSuchRepairSessionAnticompaction fails when node 1's anticompaction of the first session completes after the repair returns |
| [#245](https://github.com/JeremiahDJordan/cassandra/issues/245) | DataStax | main-5.0 | low | SecondaryIndexTest.testWriteOnlyIndex fails intermittently: the rebuild after DROP INDEX can run before the table is flushed |
| [#268](https://github.com/JeremiahDJordan/cassandra/issues/268) | DataStax | main-4.0, main-5.0 | low | Building an SAI view fails with the interval tree expensive checks when SSTable indexes share a term: RangeTermTree.Term has no equals |
| [#428](https://github.com/JeremiahDJordan/cassandra/issues/428) | DataStax | main-5.0 | low | SecondaryIndexTest queries new indexes before they are queryable and fails with IndexBuildInProgressException |
| [#464](https://github.com/JeremiahDJordan/cassandra/issues/464) | DataStax | main-4.0, main-5.0 | low | QueryMetricsTest#testQueryKindMetrics fails at SAI versions ED and GA: expected 936 cells returned, but was 1248 |
| [#569](https://github.com/JeremiahDJordan/cassandra/issues/569) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | AuditLogViewerTest leaves AuditLogViewer JVMs running after the test |
| [#570](https://github.com/JeremiahDJordan/cassandra/issues/570) | DataStax | main-4.0, main-5.0 | low | AuditLogViewerTest leaves AuditLogViewer JVMs running after the test |
| [#574](https://github.com/JeremiahDJordan/cassandra/issues/574) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | Tests leave files in the checkout root |
| [#575](https://github.com/JeremiahDJordan/cassandra/issues/575) | DataStax | main-4.0, main-5.0 | low | Tests leave files in the checkout root |
| [#805](https://github.com/JeremiahDJordan/cassandra/issues/805) | DataStax | main-4.0, main-5.0 | low | In-JVM dtest instances stay reachable after their cluster closes, so the heap of classes that start many clusters fills up |
| [#818](https://github.com/JeremiahDJordan/cassandra/issues/818) | DataStax | main-4.0, main-5.0 | low | StreamingTest hangs until the class timeout when a stream fails |
| [#821](https://github.com/JeremiahDJordan/cassandra/issues/821) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | low | StreamingTest hangs until the class timeout when a stream fails |
| [#157](https://github.com/JeremiahDJordan/cassandra/issues/157) | Apache | 4.0, 4.1, 5.0, 6.0, trunk | (none) | In-JVM dtests can't read the client warnings of a paged query |

## Apache and DataStax pairs

These issues describe the same bug on the two code bases, with a fix for each. They are expected, not duplicates.

| Apache | DataStax | Apache title |
|---|---|---|
| [#1](https://github.com/JeremiahDJordan/cassandra/issues/1) | [#125](https://github.com/JeremiahDJordan/cassandra/issues/125) | NullPointerException in Row.Merger when a dropped complex column has cells newer than the drop |
| [#3](https://github.com/JeremiahDJordan/cassandra/issues/3) | [#640](https://github.com/JeremiahDJordan/cassandra/issues/640) | Partition and range reads fail with ReadFailure when a dropped column has cells newer than the drop |
| [#8](https://github.com/JeremiahDJordan/cassandra/issues/8) | [#171](https://github.com/JeremiahDJordan/cassandra/issues/171) | Function calls with a list or map literal of another type, like count([v1, v2]), fail with a server error |
| [#10](https://github.com/JeremiahDJordan/cassandra/issues/10) | [#272](https://github.com/JeremiahDJordan/cassandra/issues/272) | GROUP BY with a limit or paging fails with "open RT bound as its last item", or counts deleted rows, when ... |
| [#120](https://github.com/JeremiahDJordan/cassandra/issues/120) | [#122](https://github.com/JeremiahDJordan/cassandra/issues/122) | sstabledump, sstablemetadata and sstablepartitions fail with EOFException on 3.x sstables whose header does ... |
| [#129](https://github.com/JeremiahDJordan/cassandra/issues/129) | [#243](https://github.com/JeremiahDJordan/cassandra/issues/243) | Replica filtering protection returns deleted rows for filters on static columns when replicas diverge |
| [#130](https://github.com/JeremiahDJordan/cassandra/issues/130) | [#232](https://github.com/JeremiahDJordan/cassandra/issues/232) | A row kept alive by cells older than its expired TTL row marker is missing from selections that don't read ... |
| [#131](https://github.com/JeremiahDJordan/cassandra/issues/131) | [#236](https://github.com/JeremiahDJordan/cassandra/issues/236) | Filtering on a collection matches deleted and expired elements and entries |
| [#132](https://github.com/JeremiahDJordan/cassandra/issues/132) | [#238](https://github.com/JeremiahDJordan/cassandra/issues/238) | PER PARTITION LIMIT returns too many rows when replica filtering protection splits a single-partition read |
| [#134](https://github.com/JeremiahDJordan/cassandra/issues/134) | [#368](https://github.com/JeremiahDJordan/cassandra/issues/368) | SAI index created while a stream is received silently misses the rows of the streamed SSTables |
| [#135](https://github.com/JeremiahDJordan/cassandra/issues/135) | [#237](https://github.com/JeremiahDJordan/cassandra/issues/237) | GROUP BY queries with a limit or paging lose groups and static rows when short read protection reaches a ... |
| [#136](https://github.com/JeremiahDJordan/cassandra/issues/136) | [#182](https://github.com/JeremiahDJordan/cassandra/issues/182) | IN with a null element before other elements matches nothing in filtered queries |
| [#137](https://github.com/JeremiahDJordan/cassandra/issues/137) | [#222](https://github.com/JeremiahDJordan/cassandra/issues/222) | Concurrent writes of one row can leave the SAI vector memtable index with a wrong, missing or extra vector ... |
| [#138](https://github.com/JeremiahDJordan/cassandra/issues/138) | [#229](https://github.com/JeremiahDJordan/cassandra/issues/229) | AbstractCommitLogSegmentManager.stopUnsafe does not wait for the segment allocator when assertions are ... |
| [#139](https://github.com/JeremiahDJordan/cassandra/issues/139) | [#230](https://github.com/JeremiahDJordan/cassandra/issues/230) | In-JVM dtest executeWithPaging fails with "calling without calling .init() first" when a replica reports a ... |
| [#140](https://github.com/JeremiahDJordan/cassandra/issues/140) | [#234](https://github.com/JeremiahDJordan/cassandra/issues/234) | In-JVM dtest executeWithPaging returns the columns the server adds for ordering |
| [#150](https://github.com/JeremiahDJordan/cassandra/issues/150) | [#233](https://github.com/JeremiahDJordan/cassandra/issues/233) | NullPointerException in multi-column relations with a tuple bound to null |
| [#161](https://github.com/JeremiahDJordan/cassandra/issues/161) | [#368](https://github.com/JeremiahDJordan/cassandra/issues/368) | An SAI index created just before streamed or imported SSTables are added misses their rows, and a failed ... |
| [#163](https://github.com/JeremiahDJordan/cassandra/issues/163) | [#427](https://github.com/JeremiahDJordan/cassandra/issues/427) | Legacy index on a static column misses rows in single-partition queries with a clustering restriction or ... |
| [#165](https://github.com/JeremiahDJordan/cassandra/issues/165) | [#235](https://github.com/JeremiahDJordan/cassandra/issues/235) | In-JVM dtest paged queries execute the first page twice |
| [#167](https://github.com/JeremiahDJordan/cassandra/issues/167) | [#231](https://github.com/JeremiahDJordan/cassandra/issues/231) | Replica filtering protection makes needless short read protection requests: range reads ask each replica for ... |
| [#169](https://github.com/JeremiahDJordan/cassandra/issues/169) | [#171](https://github.com/JeremiahDJordan/cassandra/issues/171) | Function calls with a list literal argument are rejected with "Unexpected receiver type" when an overload of ... |
| [#174](https://github.com/JeremiahDJordan/cassandra/issues/174) | [#368](https://github.com/JeremiahDJordan/cassandra/issues/368) | An SAI index created while the node is starting misses the rows of SSTables that a stream or an import is ... |
| [#192](https://github.com/JeremiahDJordan/cassandra/issues/192) | [#643](https://github.com/JeremiahDJordan/cassandra/issues/643) | List and set literals in the selection get the type of their first element even when a later element's type ... |
| [#240](https://github.com/JeremiahDJordan/cassandra/issues/240) | [#241](https://github.com/JeremiahDJordan/cassandra/issues/241) | Filtering without an index misses a row whose match needs one replica's static row and another replica's row |
| [#423](https://github.com/JeremiahDJordan/cassandra/issues/423) | [#424](https://github.com/JeremiahDJordan/cassandra/issues/424) | A write whose index fails in Indexer#finish leaves a skip-list memtable that fails to flush ("[c] is not a ... |
| [#443](https://github.com/JeremiahDJordan/cassandra/issues/443) | [#444](https://github.com/JeremiahDJordan/cassandra/issues/444) | Memtables report the 2015 encoding epoch as their min local deletion time, so reads with ... |
| [#448](https://github.com/JeremiahDJordan/cassandra/issues/448) | [#449](https://github.com/JeremiahDJordan/cassandra/issues/449) | Row cache hits and index searches purge unrepaired tombstones on tables with only_purge_repaired_tombstones |
| [#495](https://github.com/JeremiahDJordan/cassandra/issues/495) | [#479](https://github.com/JeremiahDJordan/cassandra/issues/479) | After a TRUNCATE without a flush, a write racing it can leave the truncated memtable's SAI index live, and ... |
| [#503](https://github.com/JeremiahDJordan/cassandra/issues/503) | [#504](https://github.com/JeremiahDJordan/cassandra/issues/504) | SAI orders and compares inet values as IPv6, so range and equality queries return other rows than filtering ... |
| [#507](https://github.com/JeremiahDJordan/cassandra/issues/507) | [#508](https://github.com/JeremiahDJordan/cassandra/issues/508) | The empty literal {} for a descending frozen map clustering column fails with ClassCastException |
| [#513](https://github.com/JeremiahDJordan/cassandra/issues/513) | [#514](https://github.com/JeremiahDJordan/cassandra/issues/514) | SAI: a replica whose index of the same name was recreated with other options answers reads with its own ... |
| [#515](https://github.com/JeremiahDJordan/cassandra/issues/515) | [#516](https://github.com/JeremiahDJordan/cassandra/issues/516) | SAI: a replica that doesn't know a non-tokenizing index answers = and CONTAINS reads without its options and ... |
| [#523](https://github.com/JeremiahDJordan/cassandra/issues/523) | [#524](https://github.com/JeremiahDJordan/cassandra/issues/524) | Reads purge expired cells of unrepaired data on tables with only_purge_repaired_tombstones |
| [#525](https://github.com/JeremiahDJordan/cassandra/issues/525) | [#526](https://github.com/JeremiahDJordan/cassandra/issues/526) | Range reads purge deletions written into the memtable after they opened it on tables with ... |
| [#527](https://github.com/JeremiahDJordan/cassandra/issues/527) | [#528](https://github.com/JeremiahDJordan/cassandra/issues/528) | Secondary indexes: a node that gets an index dropped and created again with other options in one schema ... |
| [#534](https://github.com/JeremiahDJordan/cassandra/issues/534) | [#372](https://github.com/JeremiahDJordan/cassandra/issues/372) | Secondary indexes: the build of a dropped index changes the build state of a new index of the same name |
| [#538](https://github.com/JeremiahDJordan/cassandra/issues/538) | [#369](https://github.com/JeremiahDJordan/cassandra/issues/369) | SAI: a new index uses the files of a dropped index of the same name while a query holds them |
| [#539](https://github.com/JeremiahDJordan/cassandra/issues/539) | [#540](https://github.com/JeremiahDJordan/cassandra/issues/540) | A partially cached partition whose cached rows end inside a range deletion fails the read with an illegal ... |
| [#541](https://github.com/JeremiahDJordan/cassandra/issues/541) | [#542](https://github.com/JeremiahDJordan/cassandra/issues/542) | SAI returns a wide partition without a partition deletion written during the search, with the deletion ... |
| [#543](https://github.com/JeremiahDJordan/cassandra/issues/543) | [#544](https://github.com/JeremiahDJordan/cassandra/issues/544) | Legacy secondary index tables never purge tombstones when the base table has only_purge_repaired_tombstones |
| [#549](https://github.com/JeremiahDJordan/cassandra/issues/549) | [#550](https://github.com/JeremiahDJordan/cassandra/issues/550) | Repaired data digest of a read of a table with only_purge_repaired_tombstones can purge a tombstone the ... |
| [#554](https://github.com/JeremiahDJordan/cassandra/issues/554) | [#555](https://github.com/JeremiahDJordan/cassandra/issues/555) | BufferPoolAllocator.Wrapped keeps a view without a cleaner after a resize into overflow memory, so release ... |
| [#565](https://github.com/JeremiahDJordan/cassandra/issues/565) | [#566](https://github.com/JeremiahDJordan/cassandra/issues/566) | Direct I/O commit log flush fails with EIO on ZFS when mutations are written into its last block during the ... |
| [#569](https://github.com/JeremiahDJordan/cassandra/issues/569) | [#570](https://github.com/JeremiahDJordan/cassandra/issues/570) | AuditLogViewerTest leaves AuditLogViewer JVMs running after the test |
| [#574](https://github.com/JeremiahDJordan/cassandra/issues/574) | [#575](https://github.com/JeremiahDJordan/cassandra/issues/575) | Tests leave files in the checkout root |
| [#585](https://github.com/JeremiahDJordan/cassandra/issues/585) | [#586](https://github.com/JeremiahDJordan/cassandra/issues/586) | SAI: TRUNCATE makes an index queryable that misses an SSTable the truncate kept |
| [#614](https://github.com/JeremiahDJordan/cassandra/issues/614) | [#616](https://github.com/JeremiahDJordan/cassandra/issues/616) | A write to a table with a materialized view times out instead of failing when its retry after a view lock ... |
| [#619](https://github.com/JeremiahDJordan/cassandra/issues/619) | [#621](https://github.com/JeremiahDJordan/cassandra/issues/621) | DecimalType byte-comparable decoding turns a decimal with a scale of Integer.MAX_VALUE into one with a scale ... |
| [#710](https://github.com/JeremiahDJordan/cassandra/issues/710) | [#711](https://github.com/JeremiahDJordan/cassandra/issues/711) | Vector columns of user types: dropping one stores a type no node can load, DROP TYPE and ALTER TYPE ignore ... |
| [#721](https://github.com/JeremiahDJordan/cassandra/issues/721) | [#731](https://github.com/JeremiahDJordan/cassandra/issues/731) | A node that doesn't know a hint's table yet acknowledges the hint, and the sender deletes it |
| [#725](https://github.com/JeremiahDJordan/cassandra/issues/725) | [#758](https://github.com/JeremiahDJordan/cassandra/issues/758) | SAI: the restrictions left to filtering on a column with an analyzed index (!=, IN, NOT IN, NOT CONTAINS, ... |
| [#726](https://github.com/JeremiahDJordan/cassandra/issues/726) | [#696](https://github.com/JeremiahDJordan/cassandra/issues/696) | SAI filtering of an unindexed map entry fails the read when a row holds an entry whose key or value is over ... |
| [#728](https://github.com/JeremiahDJordan/cassandra/issues/728) | [#695](https://github.com/JeremiahDJordan/cassandra/issues/695) | snapshot_before_compaction: compactions of a table that start in the same millisecond take the same snapshot ... |
| [#730](https://github.com/JeremiahDJordan/cassandra/issues/730) | [#739](https://github.com/JeremiahDJordan/cassandra/issues/739) | During an upgrade from 5.0, 6.0 coordinators send SAI reads with != or non-strict IN that 5.0 replicas ... |
| [#732](https://github.com/JeremiahDJordan/cassandra/issues/732) | [#733](https://github.com/JeremiahDJordan/cassandra/issues/733) | A node that comes back at the same address on another storage port loses most of its system.peers row |
| [#785](https://github.com/JeremiahDJordan/cassandra/issues/785) | [#249](https://github.com/JeremiahDJordan/cassandra/issues/249) | SAI term size failure threshold isn't applied when the warning threshold is disabled, or when the analyzer ... |
| [#797](https://github.com/JeremiahDJordan/cassandra/issues/797) | [#249](https://github.com/JeremiahDJordan/cassandra/issues/249) | SAI term size guardrail isn't applied to a partition key component written by a static-only insert: the ... |
| [#798](https://github.com/JeremiahDJordan/cassandra/issues/798) | [#249](https://github.com/JeremiahDJordan/cassandra/issues/249) | SAI sstable index writer checks the size of an analyzed value instead of its terms: compaction and index ... |
| [#821](https://github.com/JeremiahDJordan/cassandra/issues/821) | [#818](https://github.com/JeremiahDJordan/cassandra/issues/818) | StreamingTest hangs until the class timeout when a stream fails |
| [#829](https://github.com/JeremiahDJordan/cassandra/issues/829) | [#843](https://github.com/JeremiahDJordan/cassandra/issues/843) | A set or map nested in a bound collection value is stored unsorted and with duplicates |
| [#840](https://github.com/JeremiahDJordan/cassandra/issues/840) | [#841](https://github.com/JeremiahDJordan/cassandra/issues/841) | A set or map returned by a Java user-defined function or aggregate is not sorted, and is used unsorted in ... |
| [#842](https://github.com/JeremiahDJordan/cassandra/issues/842) | [#843](https://github.com/JeremiahDJordan/cassandra/issues/843) | A set or map in a bound value for a descending clustering column, a tuple, a UDT or a vector is stored ... |

63 Apache issues have a DataStax counterpart. [#827](https://github.com/JeremiahDJordan/cassandra/issues/827) is one issue with both labels and PRs for both code bases.

## Duplicates

Same root cause and same fix within one code base. Proposal only: nothing was closed as a duplicate.

| Duplicate | Keep | Code | Why |
|---|---|---|---|
| [#169](https://github.com/JeremiahDJordan/cassandra/issues/169) | [#8](https://github.com/JeremiahDJordan/cassandra/issues/8) | Apache | Same cause (a list literal checked against a type that is neither a list nor a vector); #169 says so, and one PR, #18, fixes both. |
| [#174](https://github.com/JeremiahDJordan/cassandra/issues/174) | [#161](https://github.com/JeremiahDJordan/cassandra/issues/161) | Apache | The startup path of the same per-table lock; without the fix for #161 it is part of #161. One PR, #162, fixes both. |
| [#724](https://github.com/JeremiahDJordan/cassandra/issues/724) | [#658](https://github.com/JeremiahDJordan/cassandra/issues/658) | Apache | The same mechanism (the guardrail runs in `validate`, which a node replaying the metadata log doesn't run) for `ALTER MATERIALIZED VIEW`; one PR, #770, fixes both. |
| [#857](https://github.com/JeremiahDJordan/cassandra/issues/857) | [#844](https://github.com/JeremiahDJordan/cassandra/issues/844) | Apache | Both come from literal assignment tests that don't unwrap `ReversedType`; one PR, #855, fixes both. |
| [#786](https://github.com/JeremiahDJordan/cassandra/issues/786) | [#249](https://github.com/JeremiahDJordan/cassandra/issues/249) | DataStax | The analyzed-term case of the term size guardrail; #337, the PR for #249, was amended to fix it. |
| [#635](https://github.com/JeremiahDJordan/cassandra/issues/635) | [#586](https://github.com/JeremiahDJordan/cassandra/issues/586) | DataStax | Its restart case is #360 (fixed by #397 and #413) and its truncate case is #586 (fixed by #588 and #589, which already list #635); what is left is the race filed as #636. |

No two open issues share a title within one code base; the duplicates above share a fix.

## Merge proposals

Closely related issues whose fixes would naturally be one patch and one review. Fewer issues mean fewer CI runs and reviews; unrelated fixes that only touch the same file are not proposed. Proposals only: no issue or PR was merged.

| Keep | Merge into it | Code | Why |
|---|---|---|---|
| [#134](https://github.com/JeremiahDJordan/cassandra/issues/134) | [#161](https://github.com/JeremiahDJordan/cassandra/issues/161), [#174](https://github.com/JeremiahDJordan/cassandra/issues/174) | Apache | SAI indexes created while streamed or imported SSTables are added: the fixes are stacked (#146, then #162) on the same receiver, importer and index build code. The DataStax counterpart is already one issue (#368). |
| [#742](https://github.com/JeremiahDJordan/cassandra/issues/742) | [#744](https://github.com/JeremiahDJordan/cassandra/issues/744), [#775](https://github.com/JeremiahDJordan/cassandra/issues/775) | Apache | Vector search with another restriction that is applied after the top-k (`IN` on 5.0, `IN` and negations on 6.0/trunk). One upstream ticket with a patch per branch. |
| [#746](https://github.com/JeremiahDJordan/cassandra/issues/746) | [#759](https://github.com/JeremiahDJordan/cassandra/issues/759), [#787](https://github.com/JeremiahDJordan/cassandra/issues/787), [#788](https://github.com/JeremiahDJordan/cassandra/issues/788) | Apache | Vector search on tables with static columns (static vector ordering, restrictions on indexed static columns, static-only partitions); #746 and #759 change the same `QueryController` code. |
| [#785](https://github.com/JeremiahDJordan/cassandra/issues/785) | [#797](https://github.com/JeremiahDJordan/cassandra/issues/797), [#798](https://github.com/JeremiahDJordan/cassandra/issues/798) | Apache | The SAI term size guardrail: thresholds, static-only partition key writes and the SSTable writer checking the analyzed value instead of its terms. One review of the guardrail. |
| [#658](https://github.com/JeremiahDJordan/cassandra/issues/658) | [#724](https://github.com/JeremiahDJordan/cassandra/issues/724), [#776](https://github.com/JeremiahDJordan/cassandra/issues/776), [#777](https://github.com/JeremiahDJordan/cassandra/issues/777) | Apache | Statements replayed from the metadata log skip the table and keyspace properties guardrails; same mechanism for `ALTER TABLE`, `ALTER MATERIALIZED VIEW`, `ALTER KEYSPACE` and `CREATE TABLE ... LIKE`. |
| [#654](https://github.com/JeremiahDJordan/cassandra/issues/654) | [#656](https://github.com/JeremiahDJordan/cassandra/issues/656), [#655](https://github.com/JeremiahDJordan/cassandra/issues/655) | Apache | `TableParams` in cluster metadata: the snapshot serializer omits `auto_repair` and fails on unknown memtable configurations, and the expanded CQL drops `auto_repair`. |
| [#660](https://github.com/JeremiahDJordan/cassandra/issues/660) | [#661](https://github.com/JeremiahDJordan/cassandra/issues/661) | Apache | Both are writes lost when a table changes its memtable configuration, in the same `ColumnFamilyStore#reload` / commit log path. |
| [#721](https://github.com/JeremiahDJordan/cassandra/issues/721) | [#723](https://github.com/JeremiahDJordan/cassandra/issues/723) | Apache | Hints for a table the receiving node doesn't know or has dropped, in the same hint verb handler. |
| [#856](https://github.com/JeremiahDJordan/cassandra/issues/856) | [#858](https://github.com/JeremiahDJordan/cassandra/issues/858) | Apache | Auto-repair host id migration on upgrade to 6.0. |
| [#697](https://github.com/JeremiahDJordan/cassandra/issues/697) | [#706](https://github.com/JeremiahDJordan/cassandra/issues/706) | Apache | Validation of the `auto_repair` table parameters (`AutoRepairParams`). |
| [#448](https://github.com/JeremiahDJordan/cassandra/issues/448) | [#523](https://github.com/JeremiahDJordan/cassandra/issues/523), [#525](https://github.com/JeremiahDJordan/cassandra/issues/525), [#549](https://github.com/JeremiahDJordan/cassandra/issues/549) | Apache | Reads of tables with `only_purge_repaired_tombstones` that purge unrepaired data. #549 exists only with the fix for #525, and #448 and #525 change the same read path (`PurgeFunction`, `ReadCommand`). |
| [#21](https://github.com/JeremiahDJordan/cassandra/issues/21) | [#7](https://github.com/JeremiahDJordan/cassandra/issues/7) | Apache | Null elements of collection literals in the selection (NullPointerException in the selector and in `to_json`). |
| [#22](https://github.com/JeremiahDJordan/cassandra/issues/22) | [#192](https://github.com/JeremiahDJordan/cassandra/issues/192) | Apache | Element types of collection literals in the selection; both change `Selectable` and #192 builds on the fix for #22. |
| [#136](https://github.com/JeremiahDJordan/cassandra/issues/136) | [#159](https://github.com/JeremiahDJordan/cassandra/issues/159) | Apache | `IN` restrictions with null or unset elements in filtered queries. |
| [#842](https://github.com/JeremiahDJordan/cassandra/issues/842) | [#829](https://github.com/JeremiahDJordan/cassandra/issues/829) | Apache | Sets and maps inside bound values stored unsorted and with duplicates (nested in collections on 6.0/trunk; in descending columns, tuples, UDTs and vectors on all branches). |
| [#827](https://github.com/JeremiahDJordan/cassandra/issues/827) | [#837](https://github.com/JeremiahDJordan/cassandra/issues/837) | Apache | A zero-length element of a bound value treated as null. |
| [#840](https://github.com/JeremiahDJordan/cassandra/issues/840) | [#853](https://github.com/JeremiahDJordan/cassandra/issues/853) | Apache | Collections returned by user-defined functions. |
| [#527](https://github.com/JeremiahDJordan/cassandra/issues/527) | [#534](https://github.com/JeremiahDJordan/cassandra/issues/534), [#538](https://github.com/JeremiahDJordan/cassandra/issues/538) | Apache | An index dropped and created again under the same name (`SecondaryIndexManager` state and SAI files kept by name). |
| [#112](https://github.com/JeremiahDJordan/cassandra/issues/112) | [#36](https://github.com/JeremiahDJordan/cassandra/issues/36), [#114](https://github.com/JeremiahDJordan/cassandra/issues/114) | DataStax | Trie memtable writes during `ALTER TABLE ... ADD/DROP`; the PRs are stacked on each other (#642, #39, #115). |
| [#686](https://github.com/JeremiahDJordan/cassandra/issues/686) | [#621](https://github.com/JeremiahDJordan/cassandra/issues/621) | DataStax | Decimal values whose scale changes through the trie memtable's byte-comparable encoding. |
| [#221](https://github.com/JeremiahDJordan/cassandra/issues/221) | [#222](https://github.com/JeremiahDJordan/cassandra/issues/222) | DataStax | Concurrent writes of one row on a skip-list memtable: the same retry loop and index transaction drop or corrupt the row in the SAI memtable index (regular and vector). |
| [#227](https://github.com/JeremiahDJordan/cassandra/issues/227) | [#225](https://github.com/JeremiahDJordan/cassandra/issues/225), [#226](https://github.com/JeremiahDJordan/cassandra/issues/226) | DataStax | The SAI memtable index update path (old and new values of an updated row). |
| [#239](https://github.com/JeremiahDJordan/cassandra/issues/239) | [#242](https://github.com/JeremiahDJordan/cassandra/issues/242) | DataStax | Strict filtering in SAI (CASSANDRA-19018 and CASSANDRA-20189); neither has a PR, and #242 follows #239. |
| [#246](https://github.com/JeremiahDJordan/cassandra/issues/246) | [#247](https://github.com/JeremiahDJordan/cassandra/issues/247), [#594](https://github.com/JeremiahDJordan/cassandra/issues/594) | DataStax | SAI on descending clustering columns. |
| [#253](https://github.com/JeremiahDJordan/cassandra/issues/253) | [#254](https://github.com/JeremiahDJordan/cassandra/issues/254), [#256](https://github.com/JeremiahDJordan/cassandra/issues/256) | DataStax | SAI negations missing rows; stacked PRs on the same negation planning code. |
| [#279](https://github.com/JeremiahDJordan/cassandra/issues/279) | [#429](https://github.com/JeremiahDJordan/cassandra/issues/429) | DataStax | Analyzed `NOT CONTAINS`. |
| [#469](https://github.com/JeremiahDJordan/cassandra/issues/469) | [#763](https://github.com/JeremiahDJordan/cassandra/issues/763) | DataStax | An analyzed match of a value the analyzer makes no terms of. |
| [#276](https://github.com/JeremiahDJordan/cassandra/issues/276) | [#281](https://github.com/JeremiahDJordan/cassandra/issues/281) | DataStax | Partition keys that share a token in SAI. |
| [#368](https://github.com/JeremiahDJordan/cassandra/issues/368) | [#371](https://github.com/JeremiahDJordan/cassandra/issues/371), [#647](https://github.com/JeremiahDJordan/cassandra/issues/647) | DataStax | SAI builds of streamed and imported SSTables; #647 is filed against the fix stack of #368. |
| [#369](https://github.com/JeremiahDJordan/cassandra/issues/369) | [#559](https://github.com/JeremiahDJordan/cassandra/issues/559) | DataStax | SAI files of an index of the same name. |
| [#372](https://github.com/JeremiahDJordan/cassandra/issues/372) | [#613](https://github.com/JeremiahDJordan/cassandra/issues/613), [#634](https://github.com/JeremiahDJordan/cassandra/issues/634) | DataStax | `SecondaryIndexManager` build state kept by index name. |
| [#586](https://github.com/JeremiahDJordan/cassandra/issues/586) | [#582](https://github.com/JeremiahDJordan/cassandra/issues/582), [#583](https://github.com/JeremiahDJordan/cassandra/issues/583), [#636](https://github.com/JeremiahDJordan/cassandra/issues/636) | DataStax | TRUNCATE and stopped builds that leave an SAI index queryable without an SSTable, or non-queryable; the PRs are stacked. |
| [#376](https://github.com/JeremiahDJordan/cassandra/issues/376) | [#381](https://github.com/JeremiahDJordan/cassandra/issues/381), [#382](https://github.com/JeremiahDJordan/cassandra/issues/382) | DataStax | An SSTable removed and added again during a build or `rebuild_index`. |
| [#373](https://github.com/JeremiahDJordan/cassandra/issues/373) | [#612](https://github.com/JeremiahDJordan/cassandra/issues/612) | DataStax | SAI and early-opened compaction results. |
| [#630](https://github.com/JeremiahDJordan/cassandra/issues/630) | [#631](https://github.com/JeremiahDJordan/cassandra/issues/631) | DataStax | Two builds of the same index on one SSTable (main-4.0). |
| [#649](https://github.com/JeremiahDJordan/cassandra/issues/649) | [#662](https://github.com/JeremiahDJordan/cassandra/issues/662) | DataStax | BTI SSTables and deletion times after 2038. |
| [#665](https://github.com/JeremiahDJordan/cassandra/issues/665) | [#668](https://github.com/JeremiahDJordan/cassandra/issues/668) | DataStax | Size of the SAI index status gossip value. |
| [#675](https://github.com/JeremiahDJordan/cassandra/issues/675) | [#692](https://github.com/JeremiahDJordan/cassandra/issues/692) | DataStax | `CASSANDRA_4` mode: `system_schema` columns and layouts Apache Cassandra 4.x can't read. |
| [#257](https://github.com/JeremiahDJordan/cassandra/issues/257) | [#689](https://github.com/JeremiahDJordan/cassandra/issues/689), [#739](https://github.com/JeremiahDJordan/cassandra/issues/739), [#740](https://github.com/JeremiahDJordan/cassandra/issues/740) | DataStax | DataStax coordinators sending reads that Apache Cassandra 5.0 replicas can't serve; one messaging-version gate. |
| [#199](https://github.com/JeremiahDJordan/cassandra/issues/199) | [#200](https://github.com/JeremiahDJordan/cassandra/issues/200), [#201](https://github.com/JeremiahDJordan/cassandra/issues/201) | DataStax | `OR` of a static and a regular column; #201 references both. |
| [#202](https://github.com/JeremiahDJordan/cassandra/issues/202) | [#203](https://github.com/JeremiahDJordan/cassandra/issues/203), [#204](https://github.com/JeremiahDJordan/cassandra/issues/204), [#205](https://github.com/JeremiahDJordan/cassandra/issues/205) | DataStax | SAI `ORDER BY` and ANN with static rows; the PRs are stacked. |
| [#182](https://github.com/JeremiahDJordan/cassandra/issues/182) | [#183](https://github.com/JeremiahDJordan/cassandra/issues/183) | DataStax | Filtered `IN` with a null element. |
| [#237](https://github.com/JeremiahDJordan/cassandra/issues/237) | [#645](https://github.com/JeremiahDJordan/cassandra/issues/645) | DataStax | #645 exists only with the fix for #237 (#295). |
| [#449](https://github.com/JeremiahDJordan/cassandra/issues/449) | [#524](https://github.com/JeremiahDJordan/cassandra/issues/524), [#526](https://github.com/JeremiahDJordan/cassandra/issues/526), [#550](https://github.com/JeremiahDJordan/cassandra/issues/550) | DataStax | As for the Apache issues: reads that purge unrepaired data; #550 exists only with the fix for #526. |
| [#462](https://github.com/JeremiahDJordan/cassandra/issues/462) | [#463](https://github.com/JeremiahDJordan/cassandra/issues/463) | DataStax | SAI documentation. |
| [#592](https://github.com/JeremiahDJordan/cassandra/issues/592) | [#466](https://github.com/JeremiahDJordan/cassandra/issues/466) | DataStax | Classes named in options initialized before their type is checked. |
| [#605](https://github.com/JeremiahDJordan/cassandra/issues/605) | [#609](https://github.com/JeremiahDJordan/cassandra/issues/609) | DataStax | SAI analyzer configuration (`LuceneCustomAnalyzerConfig`). |

With all duplicates and merges, 388 open issues would become 308.

## Closed in this triage

Each issue was fixed on every branch it is labelled for. Each got a comment naming the commit, and its PRs were closed with a comment pointing to it. No other open PR was based on these PRs except #156 on #78, closed too.

| Issue | PRs closed | Branch | Upstream commit | Evidence |
|---|---|---|---|---|
| [#71](https://github.com/JeremiahDJordan/cassandra/issues/71) Replica sensors are not tracked for SAI range reads with endpoint grouping (CNDB-19694 ... | [#76](https://github.com/JeremiahDJordan/cassandra/pull/76) | `main-5.0` | [`26b505b7b11`](https://github.com/datastax/cassandra/commit/26b505b7b11bc439f0c278195d2d1ad9ecae266f) CNDB-19694 Fix sensors not tracked for SAI range reads | Same change as #76 (`ReadRequestCallback`, implemented by `ReadCallback` and `SingleEndpointCallback`). |
| [#72](https://github.com/JeremiahDJordan/cassandra/issues/72) SSTableIntervalTree latency metric from main-4.0 (HCD-468) is missing | [#78](https://github.com/JeremiahDJordan/cassandra/pull/78) | `main-5.0` | [`2c6c2670e84`](https://github.com/datastax/cassandra/commit/2c6c2670e840ba0e37a65de1d941d52ea5b48b18) CNDB-20249: Port IntervalTree add/replace support and SSTableIntervalTree latency metric from main-4.0<br>[`ccd81c61cc9`](https://github.com/datastax/cassandra/commit/ccd81c61cc959e21d458e27373360cc9435a9872) CNDB-20249: Address Sonar findings and add tests | `ViewSSTableIntervalTree` metrics and the `TableMetricsTest` check are on `datastax/main`. |
| [#155](https://github.com/JeremiahDJordan/cassandra/issues/155) IntervalTree add and replace from main-4.0 (HCD-468) are missing | [#156](https://github.com/JeremiahDJordan/cassandra/pull/156) | `main-5.0` | [`2c6c2670e84`](https://github.com/datastax/cassandra/commit/2c6c2670e840ba0e37a65de1d941d52ea5b48b18) CNDB-20249: Port IntervalTree add/replace support and SSTableIntervalTree latency metric from main-4.0<br>[`ccd81c61cc9`](https://github.com/datastax/cassandra/commit/ccd81c61cc959e21d458e27373360cc9435a9872) CNDB-20249: Address Sonar findings and add tests | `IntervalTree.java` is the same as in #156; `IntervalTreeSequenceTest` is identical. |
| [#194](https://github.com/JeremiahDJordan/cassandra/issues/194) Trie memtable: a partition deletion and a range deletion with the same timestamp fail ... | [#195](https://github.com/JeremiahDJordan/cassandra/pull/195) | `main-5.0` | [`52acec20ef3`](https://github.com/datastax/cassandra/commit/52acec20ef376276acc590aa31a80475f7707d8c) CNDB-20251: Break ties between equal deletion times by kind in trie memtables | Same tie-break in `TrieTombstoneMarker.combine`, with `EqualTimestampTombstoneTest`. |
| [#300](https://github.com/JeremiahDJordan/cassandra/issues/300) SAI memtable index fails the read for a contradictory range such as v >= 5 AND v < 3 | [#318](https://github.com/JeremiahDJordan/cassandra/pull/318) | `main-4.0` | [`c8456a5dcdc`](https://github.com/datastax/cassandra/commit/c8456a5dcdc8bc792ab8d3790b156f3f3015ccfd) CNDB-19679: Fix SAI assertion on empty range queries against the memtable index (#2821) | `ContradictoryRangeTest` from #318 fails on `datastax/base-4.0` and passes on `datastax/main-4.0`. |

## Partly fixed upstream

No issue is fixed on only some of its labelled branches, so no branch label was removed.

[#131](https://github.com/JeremiahDJordan/cassandra/issues/131) (filtering on a collection matches deleted elements) is fixed in part on `cassandra-5.0`, `cassandra-6.0` and `trunk` by [`77a030d8c92`](https://github.com/apache/cassandra/commit/77a030d8c92cc6e2315af77b590eb8824bb7da0b) Assorted fixes for post-filtering on collection elements and inet columns (CASSANDRA-21863): `RowFilter` now evaluates static column expressions on a purged static row when the read doesn't need reconciliation. The single-node test from [#143](https://github.com/JeremiahDJordan/cassandra/pull/143) now passes on `cassandra-5.0` (it fails at `fb7f1b7aeeb`), but the multi-replica test still fails, so the issue stays open on every branch; a comment says so. [#143](https://github.com/JeremiahDJordan/cassandra/pull/143) conflicts with that commit and needs a rebase.

## Checked and still open

The PR tests below were run on the upstream tip without the fix, because an upstream commit changed the same code. All still fail, so the bugs are still there.

| Issue | PR | Run on | Result |
|---|---|---|---|
| [#128](https://github.com/JeremiahDJordan/cassandra/issues/128) | [#141](https://github.com/JeremiahDJordan/cassandra/pull/141) | `cassandra-5.0` `3e3892ce99a` | `NonStrictFilteringMultiDatacenterTest`: 2 of 2 fail |
| [#129](https://github.com/JeremiahDJordan/cassandra/issues/129) | [#142](https://github.com/JeremiahDJordan/cassandra/pull/142) | `cassandra-5.0` `3e3892ce99a` | `ReplicaFilteringStaticDivergenceTest#testDivergentReplicas` fails (also at `fb7f1b7aeeb`) |
| [#133](https://github.com/JeremiahDJordan/cassandra/issues/133) | [#145](https://github.com/JeremiahDJordan/cassandra/pull/145) | `cassandra-5.0` `3e3892ce99a` | 4 paging tests of `StrictFilteringTest` fail |
| [#136](https://github.com/JeremiahDJordan/cassandra/issues/136) | [#148](https://github.com/JeremiahDJordan/cassandra/pull/148) | `cassandra-5.0` `3e3892ce99a` | `SelectSingleColumnRelationTest#testInRestrictionsWithNullElementsAndAllowFiltering` fails |
| [#150](https://github.com/JeremiahDJordan/cassandra/issues/150) | [#151](https://github.com/JeremiahDJordan/cassandra/pull/151) | `cassandra-5.0` `3e3892ce99a` | `SelectMultiColumnRelationTest#testWithNullTupleValues` fails |
| [#503](https://github.com/JeremiahDJordan/cassandra/issues/503) | [#517](https://github.com/JeremiahDJordan/cassandra/pull/517) | `cassandra-5.0` `3e3892ce99a` | `InetOrderTest` (5 of 5) and `InetAddressTypeEquivalencyTest` fail; CASSANDRA-21863 changed the same `inet` comparison but still compares as IPv6 |
| [#199](https://github.com/JeremiahDJordan/cassandra/issues/199) | [#383](https://github.com/JeremiahDJordan/cassandra/pull/383) | `datastax/main` `f7a7fc031a3` | `StaticColumnDisjunctionTest` (15 of 15) and 3 `RowFilterTest` cases fail |
| [#207](https://github.com/JeremiahDJordan/cassandra/issues/207) | [#305](https://github.com/JeremiahDJordan/cassandra/pull/305) | `datastax/main` | `BM25Test#testRowDeletedAndWrittenAgain` fails at every version |
| [#226](https://github.com/JeremiahDJordan/cassandra/issues/226) | [#340](https://github.com/JeremiahDJordan/cassandra/pull/340) | `datastax/main` | `EmptyValueUpdateTest` and `BM25UpdateTest` fail (the PR is stacked on the fix for #225) |
| [#248](https://github.com/JeremiahDJordan/cassandra/issues/248) | [#334](https://github.com/JeremiahDJordan/cassandra/pull/334) | `datastax/main` | `EmptyBoundsTest` and `EmptyBoundSearchTest` fail with the read failure the issue describes |
| [#275](https://github.com/JeremiahDJordan/cassandra/issues/275) | [#311](https://github.com/JeremiahDJordan/cassandra/pull/311), [#312](https://github.com/JeremiahDJordan/cassandra/pull/312) | `datastax/main`, `datastax/main-4.0` | `NotEqualsEmptyValueTest` fails on both; CNDB-19679 changed the same `TrieMemoryIndex` code |
| [#479](https://github.com/JeremiahDJordan/cassandra/issues/479) | [#498](https://github.com/JeremiahDJordan/cassandra/pull/498) | `datastax/main` | `TrackerTest#testMemtableReplacement` and `NativeIndexDDLTest#testTruncateWithoutFlushDuringWrite` fail |
| [#257](https://github.com/JeremiahDJordan/cassandra/issues/257), [#307](https://github.com/JeremiahDJordan/cassandra/issues/307) | [#348](https://github.com/JeremiahDJordan/cassandra/pull/348), [#321](https://github.com/JeremiahDJordan/cassandra/pull/321) | `datastax/main` | their tests use classes and constants the fix adds, so the fix is not there |

## Unclear cases and follow-ups

- PRs that no longer apply to the upstream tip of their base and need a rebase: [#143](https://github.com/JeremiahDJordan/cassandra/pull/143), [#517](https://github.com/JeremiahDJordan/cassandra/pull/517) (on `cassandra-5.0`, after CASSANDRA-21863 and CASSANDRA-21897); [#311](https://github.com/JeremiahDJordan/cassandra/pull/311), [#312](https://github.com/JeremiahDJordan/cassandra/pull/312), [#321](https://github.com/JeremiahDJordan/cassandra/pull/321), [#348](https://github.com/JeremiahDJordan/cassandra/pull/348), [#383](https://github.com/JeremiahDJordan/cassandra/pull/383), [#498](https://github.com/JeremiahDJordan/cassandra/pull/498) (on `datastax/main`, after CNDB-19679 and CNDB-20249). Most Apache 5.0 PRs are based on `fb7f1b7aeeb`, 11 commits behind `cassandra-5.0`; apart from `CHANGES.txt` they still apply cleanly.

- [#240](https://github.com/JeremiahDJordan/cassandra/issues/240), [#515](https://github.com/JeremiahDJordan/cassandra/issues/515) and [#541](https://github.com/JeremiahDJordan/cassandra/issues/541) have no PR and were filed against `cassandra-5.0` before CASSANDRA-21897 (`47227d98bf0`, SAI reconciliation with range tombstones and multi-cell collections). That commit changes `RowFilter#isMutableIntersection` and how `StorageAttachedIndexSearcher` keeps range tombstones, not the static row matching, unknown-index and partition-deletion paths these issues describe, so they are most likely still there; they were not run again.

- [#165](https://github.com/JeremiahDJordan/cassandra/issues/165) and [#235](https://github.com/JeremiahDJordan/cassandra/issues/235) are fixed on Apache `trunk` by CASSANDRA-20089 and labelled only for the branches without it, which is correct.

- [#613](https://github.com/JeremiahDJordan/cassandra/issues/613) says the DataStax mainline merge branch is not affected; that branch is not upstream, so the issue stays open for `datastax/main` and `main-4.0`.

- CNDB-19679 on `datastax/main` (`8e7a2c6cccf`, "Remove empty ranges from RowFilter") rewrites parts of `RowFilter`, `StatementRestrictions` and `SelectStatement`. The DataStax PRs that touch those files still apply, and the issues tested above still fail, but PRs that change `RowFilter` should be rebased and their tests run again before review.
