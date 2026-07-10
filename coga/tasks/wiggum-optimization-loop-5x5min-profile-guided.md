---
slug: wiggum-optimization-loop-5x5min-profile-guided
title: Wiggum optimization loop (5x5min, profile-guided)
status: active
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

Run the "wiggum loop" — a profile-guided hill-climb that optimizes Spark's
TPC-DS runtime. Up to 5 attempts, each capped at ~5 minutes: an attempt
applies one targeted change, re-runs the benchmark, and scores total runtime
against the locked baseline. Keep the best-scoring attempt. Goal: a measurable
speedup over baseline at SF=1, with the winning config saved for verification.

## Context

- Depends on: set-up-spark-tpc-ds-baseline-sf-1 (needs the locked baseline +
  the top-5 slowest-query profile before this can start).
- Loop shape: greedy best-of-5. Baseline is measured once; each attempt is a
  diff scored against it; best total runtime wins. The 5-min cap keeps each
  attempt to a fast pass at SF=1 — which is also why attempts are config/plan
  changes only (a Spark rebuild can't fit the cap).
- Attempts are profile-guided, not random — each targets a diagnosed
  bottleneck from ticket 1. Lever menu, roughly by payoff:
  - AQE: coalesce shuffle partitions, skew-join handling, dynamic partition
    pruning.
  - Shuffle/parallelism: `spark.sql.shuffle.partitions`, executor/core/memory
    sizing.
  - Joins: broadcast-join threshold, join-strategy hints on the heavy queries.
  - I/O: Parquet, partitioning/bucketing, column pruning, predicate pushdown.
  - CBO: `ANALYZE TABLE` stats, exchange reuse.
- Out of scope: JVM hot-method profiling / patching Spark source (locked with
  owner 2026-07-09 — config/query-plan tuning only for the POC).
- Deliverable: the winning config (as a diff/properties file) plus the score
  table showing each attempt's total vs baseline.
- Open question (owned by ticket 1): scoring on the full 99-query suite vs a
  representative subset within the 5-min cap — read the answer from ticket
  1's results before starting.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.

Prior run of this ticket (2026-07-07) is recoverable via
`git show 9253fab^:coga/tasks/wiggum-optimization-loop-5x5min-profile-guided.md`.

## Scoring set resolved by baseline ticket (2026-07-09)

- Score every attempt on the full 103-file TPC-DS suite (99 query numbers plus
  four `a`/`b` variants), not a representative subset.
- Locked baseline: 103/103 successful, 165.837s total wall runtime on
  Spark 3.5.8 `local[16]`, leaving about 134s inside the five-minute cap.
- Read `bench/results/` and `bench/profile/top5_slowest.md` before choosing
  attempt 1.
