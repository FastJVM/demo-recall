---
slug: set-up-spark-tpc-ds-baseline-sf-1
title: Set up Spark + TPC-DS baseline (SF=1)
status: in_progress
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
step: 1 (execute)
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

## Decisions (locked with human 2026-07-07)
- **Toolchain**: `tpcds-kit` (dsdgen) + `databricks/spark-sql-perf` (Scala/sbt harness). Spec-faithful route.
- **JVM**: JDK 17 (Spark 3.5 doesn't support the machine's default Java 25). Using sdkman `17.0.9-tem`.
- **Location**: all work under `bench/` (gitignored except scripts/ + results/).
- **Spark**: 3.5.8 (bin-hadoop3), latest 3.5.x on dlcdn.

## Environment facts
- 16 cores, 62 GB RAM. No passwordless sudo → can't apt-install flex/bison.
- conda (miniconda3) present → flex/bison/gcc via conda-forge into `bench/opt/buildenv` (no sudo).
- sdkman present with `17.0.9-tem`, sbt, maven.

## Layout under bench/
- `downloads/` spark tarball (ignored)  · `opt/` JDK, spark install, conda buildenv (ignored)
- `src/tpcds-kit`, `src/spark-sql-perf` (ignored) · `data/` SF=1 Parquet (ignored)
- `scripts/` reproducible run scripts (COMMITTED) · `results/` baseline runtimes + profile (COMMITTED)
- `logs/` build/run logs (ignored)

## Plan / phases
1. [in progress] Install JDK17 (done via sdkman), download Spark 3.5.8 (done), get flex/bison (conda, building).
2. Build tpcds-kit dsdgen with conda toolchain.
3. Build spark-sql-perf jar (sbt) against Spark 3.5 / Scala 2.12.
4. Generate TPC-DS SF=1 → Parquet via spark-sql-perf Tables API.
5. Write reproducible run script (env pinning JAVA_HOME/SPARK_HOME, event-log enabled).
6. Run full 99-query suite once; record per-query + total runtime → results/.
7. Profile top ~5 slowest queries (event logs + EXPLAIN): shuffle spill / skew / SMJ-should-broadcast / full-scan-should-prune.

## Progress log
- 21:49 Spark 3.5.8 tarball downloaded (~401MB). tpcds-kit + spark-sql-perf cloned. conda buildenv (flex/bison/gcc) building.
- Risk: spark-sql-perf is unmaintained; sbt build against Spark 3.5 may need version pins. Will fall back to raw dsdgen + official query templates run via spark-sql if the Scala harness won't build.

## Usage

{"agent":"claude","cache_creation_input_tokens":179644,"cache_read_input_tokens":5285730,"cli":"claude","input_tokens":21580,"model":"claude-opus-4-8","output_tokens":101969,"provider":"anthropic","schema":1,"session_id":"042e765a-df9f-4945-9113-2905239b62e1","slug":"set-up-spark-tpc-ds-baseline-sf-1","step":"execute","title":"Set up Spark + TPC-DS baseline (SF=1)","ts":"2026-07-08T05:22:41.680667Z","usage_status":"ok"}
