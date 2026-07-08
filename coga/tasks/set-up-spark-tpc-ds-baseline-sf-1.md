---
slug: set-up-spark-tpc-ds-baseline-sf-1
title: Set up Spark + TPC-DS baseline (SF=1)
status: done
mode: agent
owner: nicktoper
human: nicktoper
agent: claude
assignee: claude
contexts: []
skills: []
workflow:
  name: direct/body
  steps:
  - name: execute
    skills:
    - direct/body
    assignee: agent
secrets: null
script: null
---

## Description

Stand up everything needed to benchmark Spark on TPC-DS locally and lock a
baseline the rest of the project measures against. Download and install Spark,
generate the TPC-DS dataset at scale factor 1, run the full query suite once,
and record per-query + total runtime. This is the hard prerequisite for the
optimization loop — nothing downstream can be scored without it.

## Context

- Environment: single local machine, POC. Scale factor 1 (~1GB) for fast
  iteration, not a cluster.
- Nothing exists yet — this ticket owns downloading/installing Spark and a
  TPC-DS data generator (e.g. the `spark-sql-perf` / `tpcds-kit` route or an
  equivalent), then generating SF=1 data.
- Deliverables: (a) a reproducible run script, (b) a locked baseline of
  per-query and total runtime saved to disk, and (c) a short **profile** of the
  top ~5 slowest queries and *why* they're slow — read from the Spark UI /
  event logs and `EXPLAIN` plans (shuffle spill, task skew, sort-merge join
  that should broadcast, full scan that should be pruned).
- The profile is the key output: it's the input that lets the optimization loop
  target diagnosed bottlenecks instead of guessing.
- Out of scope: JVM hot-method profiling / patching Spark source. At SF=1
  nothing is CPU-bound in a way that rewriting internals would fix — this is
  config/query-plan tuning only.
- Open question: exact TPC-DS generator toolchain to standardize on — pick the
  simplest that produces SF=1 Parquet reliably on a single machine.

<!-- coga:blackboard -->

## Plan & decisions (execute step)

**Environment probed (2026-07-07):** 16 cores, 62GB RAM, 476GB free disk.
Network OK (dlcdn.apache.org 200, pypi 200, github 200). Java 17 available via
sdkman (`17.0.9-tem`) — using it because the box default is Java 25, too new
for Spark (Spark 3.5 supports 8/11/17).

**Open question resolved — generator toolchain:** DuckDB's built-in `tpcds`
extension (`INSTALL tpcds; CALL dsdgen(sf=1)`) → export each table to Parquet.
Simplest reliable SF=1 Parquet on a single machine: no C compilation
(vs tpcds-kit), no stale Scala jar (vs spark-sql-perf). Queries come from
Apache Spark's own TPC-DS test resources (Spark SQL dialect, q1..q99).

**Stack:** Spark 3.5.x distro (tgz from dlcdn) + Java 17. DuckDB (pip) for data
gen. PySpark driver script runs the suite with event logging on for profiling.

**Layout:**
- `bench/` (committed): scripts/, queries/, results/, profile/, README.md
- `bench/.local/` (gitignored): spark distro, parquet data, spark event logs

**Deliverables:** (a) reproducible run script, (b) locked per-query + total
runtime, (c) profile of top ~5 slowest queries w/ diagnosed cause.

### Progress log — DONE
All three deliverables produced under `bench/` (heavy artifacts in
`bench/.local/`, gitignored). Reproduce: `setup.sh` → `run_baseline.sh` →
`profile_top.sh`.

- Spark 3.5.8 + Java 17 installed; DuckDB `tpcds` generated SF=1 (24 tables,
  19.5M rows, 277MB ZSTD parquet ≈ 1GB logical).
- **(a) run scripts:** `bench/scripts/` (env, setup, gen_data, run_baseline,
  profile_top, analyze_eventlog).
- **(b) LOCKED baseline:** `bench/results/` — **103/103 queries OK, total
  140.0s** (vanilla Spark defaults; the fair "before"). CSV + JSON + summary.md.
- **(c) profile:** `bench/profile/top5_slowest.md` — top-5 slowest = q4 (5.36s),
  q14a (5.32s), q72 (4.44s), q14b (3.47s), q23a (3.20s), each with EXPLAIN plan
  + event-log metrics (shuffle/spill/skew).

Key diagnosis for the optimization loop (config/query tuning, no source hacks):
1. **Broadcast threshold too low (10MB)** — q72's `catalog_sales ⋈ inventory`
   on `item_sk` is a SortMergeJoin shuffling 189MB (worst in suite, skew 5.4×)
   only because inventory>10MB; raising `autoBroadcastJoinThreshold` fixes it.
   Same for q23a's small item/customer CTEs.
2. **Task skew 3.2–5.6×** from low-cardinality keys (item_sk ~18k, composite
   brand/class/category); AQE skewJoin thresholds (256MB) never fire at SF=1.
3. **shuffle.partitions=200 too many** for ~1GB → scheduling overhead.
4. **Repeated fact scans** (2–10× per query in q4/q14a/q14b/q23a) → cache/reuse.
5. No spill anywhere — memory is not the constraint at SF=1.

Decisions made: generator=DuckDB (simplest reliable SF=1 parquet); Java 17
(box default Java 25 unsupported by Spark 3.5); q30 column-name shim
(`c_last_review_date` ↔ `_sk`) so canonical query text stays unmodified.


## Usage

{"agent":"claude","cache_creation_input_tokens":196642,"cache_read_input_tokens":6480532,"cli":"claude","input_tokens":23569,"model":"claude-opus-4-8","output_tokens":116036,"provider":"anthropic","schema":1,"session_id":"90320791-69c0-451a-a35e-740a9809f582","slug":"set-up-spark-tpc-ds-baseline-sf-1","step":"execute","title":"Set up Spark + TPC-DS baseline (SF=1)","ts":"2026-07-08T04:38:38.688993Z","usage_status":"ok"}
