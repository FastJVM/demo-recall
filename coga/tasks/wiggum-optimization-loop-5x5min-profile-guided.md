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
TPC-DS runtime. Up to 5 attempts, each capped at ~5 minutes: an attempt applies
one targeted change, re-runs the benchmark, and scores total runtime against the
locked baseline. Keep the best-scoring attempt. The goal is a measurable
speedup over baseline at SF=1, with the winning config saved for verification.

## Context

- Depends on: set-up-spark-tpc-ds-baseline-sf-1 (needs the locked baseline +
  the top-5 slowest-query profile before this can start).
- Loop shape: greedy best-of-5. Baseline is measured once; each attempt is a
  diff scored against it; best total runtime wins. The 5-min cap keeps each
  attempt to a fast pass at SF=1.
- Attempts are **profile-guided**, not random — each targets a diagnosed
  bottleneck from ticket 1. Lever menu, roughly by payoff:
  - AQE: coalesce shuffle partitions, skew-join handling, dynamic partition
    pruning.
  - Shuffle/parallelism: `spark.sql.shuffle.partitions`, executor/core/memory
    sizing.
  - Joins: broadcast-join threshold, join-strategy hints on the heavy queries.
  - I/O: Parquet, partitioning/bucketing, column pruning, predicate pushdown.
  - CBO: `ANALYZE TABLE` stats, exchange reuse.
- Deliverable: the winning config (as a diff/properties file) plus the score
  table showing each attempt's total vs baseline.
- Open question: scoring on the full suite vs a representative subset within the
  5-min cap — decide during ticket 1 based on how long the SF=1 suite actually
  takes.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
