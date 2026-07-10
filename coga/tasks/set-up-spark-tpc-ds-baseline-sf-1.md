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
baseline the rest of the project measures against. Install Spark, generate the
TPC-DS dataset at scale factor 1, run the full query suite once, and record
per-query + total runtime. This is the hard prerequisite for the optimization
loop — nothing downstream can be scored without it.

## Context

- Environment: single local machine, POC. SF=1 (~1GB) for fast iteration.
  16 cores / 62 GB RAM. No passwordless sudo — get flex/bison/gcc via
  conda-forge into a local buildenv (miniconda3 is present).
- Decisions locked in the prior run of this project (2026-07-07, recovered
  from git history — reuse unless something has changed):
  - Toolchain: `tpcds-kit` (dsdgen) + `databricks/spark-sql-perf` (sbt harness).
  - Spark 3.5.8 (bin-hadoop3); JDK 17 via sdkman (`17.0.9-tem`) — Spark 3.5
    does not support the machine's default Java 25.
  - Layout: everything under `bench/` — `scripts/` and `results/` committed,
    `downloads/`, `opt/`, `src/`, `data/`, `logs/` gitignored.
- Known risk from prior run: spark-sql-perf is unmaintained and its sbt build
  against Spark 3.5 / Scala 2.12 may need version pins. Fallback: raw dsdgen
  + official TPC-DS query templates run via spark-sql.
- Deliverables: (a) a reproducible run script (event-log enabled), (b) a
  locked baseline of per-query + total runtime saved to `bench/results/`,
  (c) a short profile of the top ~5 slowest queries and why they're slow —
  from Spark UI / event logs and EXPLAIN plans (shuffle spill, skew,
  sort-merge join that should broadcast, scan that should be pruned).
  The profile is the key output: it lets the loop target diagnosed
  bottlenecks instead of guessing.
- Out of scope: JVM hot-method profiling / patching Spark source — this
  project is config/query-plan tuning only.
- Open question: score the loop on the full 99-query suite or a
  representative subset within the 5-min attempt cap — decide here once the
  SF=1 suite runtime is known, and record the answer for the loop ticket.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.

Prior-run blackboard (decisions, environment facts, progress) is recoverable
via `git show 9253fab^:coga/tasks/set-up-spark-tpc-ds-baseline-sf-1.md`.

## Recovered state (2026-07-09)

- Locked choices: Spark 3.5.8 bin-hadoop3, JDK 17 (`17.0.9-tem`),
  `tpcds-kit` + `spark-sql-perf`, and all benchmark material under `bench/`.
- Prior run downloaded Spark and cloned both upstream repositories, then stopped
  while creating the conda build environment; no scripts or results were
  committed.
- Current Coga checkout is detached and contains no `bench/` directory. The
  machine default is Java 25, so every benchmark script must pin JDK 17.

## Execution plan

1. Recover any surviving ignored payload from another checkout; otherwise
   recreate the local toolchain and sources.
2. Generate SF=1 Parquet and make generation/runs reproducible from committed
   scripts with Spark event logging enabled.
3. Run all 99 query templates, lock per-query and total timings, inspect event
   logs and physical plans for the five slowest queries, and record the loop
   scoring decision.

## Progress

- Recovered a still-reachable older completed-task blackboard, but verified its
  claimed `bench/` deliverables were never committed and no ignored payload
  survives. Its 103/103 in 140.0s measurement is historical evidence only.
- Downloaded and SHA-512-verified Spark 3.5.8; pinned `tpcds-kit` at
  `1b7fb7529edae091684201fab142d956d6afd881` and `spark-sql-perf` at
  `28d88190f6a5f6698d32eaf4092334c41180b806`.
- Built `dsdgen` using the machine's existing GCC. Flex/bison are unnecessary
  because only `dsdgen`, not `dsqgen`, is used.
- Built a minimal Spark-3.5.8/Scala-2.12 harness from `spark-sql-perf`'s typed
  TPC-DS schemas, with one reproducible compatibility fix for an obsolete
  unused import. Copied all 103 Spark-dialect query files into `bench/queries/`.
- Setup scripts and event-log-aware baseline runner are in place. Next:
  generate SF=1 Parquet, run the suite, then profile the five slowest queries.

## Baseline complete

- Generated all 24 SF=1 tables with `dsdgen`: 610 MiB of Snappy Parquet in
  the ignored `bench/data/sf1-parquet/` payload.
- Full suite: **103/103 succeeded in 165.837s wall time**. Locked CSV, JSON,
  and Markdown results are under `bench/results/`.
- Slowest five: q88 6.593s, q14a 6.539s, q28 6.101s, q4 5.795s, q95 5.756s.
  `bench/profile/top5_slowest.md` correlates each with event-log and physical
  plan evidence. All five had zero spill; repeated scans, 200-way shuffle
  overhead, derived join strategy, and q95 order-key skew dominate.
- Loop scoring decision: use the full 103-file suite. At 165.837s it is safely
  inside the five-minute attempt cap; the decision is also recorded on the
  optimization-loop ticket.

## Usage

{"agent":"codex","cache_creation_input_tokens":null,"cache_read_input_tokens":8122368,"cli":"codex","input_tokens":232490,"model":"gpt-5.6-sol","output_tokens":31474,"provider":"openai","schema":1,"session_id":"019f4a17-df54-7f60-b08d-9610fd07f16e","slug":"set-up-spark-tpc-ds-baseline-sf-1","step":"execute","title":"Set up Spark + TPC-DS baseline (SF=1)","ts":"2026-07-10T04:06:10.898529Z","usage_status":"ok"}
