---
slug: set-up-spark-tpc-ds-baseline-sf-1
title: Set up Spark + TPC-DS baseline (SF=1)
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
