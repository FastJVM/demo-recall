---
slug: set-up-spark-tpc-ds-baseline-sf-1
title: Set up Spark + TPC-DS baseline (SF=1)
status: draft
mode: agent
owner: nicktoper
human: nicktoper
agent: claude
assignee: nicktoper
contexts: []
skills: []
workflow: null
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

The blackboard is a notepad to be written to often as the human and agent works through a task.
