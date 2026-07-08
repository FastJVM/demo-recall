---
slug: verify-winning-config-and-write-report
title: Verify winning config and write report
status: draft
mode: agent
owner: nicktoper
human: nicktoper
agent: claude
assignee: nicktoper
contexts: []
skills: []
workflow: direct/body
secrets: null
script: null
---

## Description

Confirm the loop's winning config is a real, reproducible speedup — not noise —
and write it up. Re-run the winning config clean against the same SF=1 dataset,
compare to the locked baseline, and produce a before/after report: total runtime
delta plus per-query deltas, with the config change that caused it. This is the
project's demo artifact and its definition of done.

## Context

- Depends on: wiggum-optimization-loop-5x5min-profile-guided (needs the winning
  config + its score table).
- Verify reproducibility: run the winner clean (fresh session, cold-ish start)
  and check the speedup holds, not just the single lucky loop run. Note the
  variance if runs differ.
- Deliverable: a before/after report — total + per-query runtime, the winning
  config diff, and a one-line "what made it faster" per notable query.
- Acceptance bar (the project's done-state / demo): a reproducible before/after
  showing a measurable total-runtime improvement on TPC-DS SF=1, runnable end to
  end on the single machine.
- POC, single machine, no external sign-off — owner runs it end to end.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
