# Daily digest spool

Producer/consumer queue for `coga digest`. Producers append one JSONL record
at the **bottom** of `## Spool (pending)`; the single consumer (`coga digest`)
advances the `consumed_through:` watermark to the newest record and trims the
consumed prefix, always keeping the newest record in place as an *anchor*.

This file is marked `merge=union` (`.gitattributes`) so two clones appending
concurrently merge without conflict. Together with the top-trim/bottom-append
shape (deletes and appends sit in disjoint hunks separated by the anchor), that
makes the spool mergeable by construction with no lock — see the `coga/sync`
context. The git high-water mark lives separately in the digest ticket's
`### Digest State`, not here.

## Spool (pending)

consumed_through:
{"id":"6460f175ae25","ts":"2026-07-07T21:38","project":"1d8f5516e94146ccbe3d0c71e108bda4","kind":"done","detail":"claude finished: execute → done ✅ — TPC-DS SF=1 baseline locked: 103/103 queries, total 140.0s (Spark 3.5.8, vanilla defaults). Deliverables in bench/: run scripts, locked baseline (results/), profile of top-5 slowest (profile/top5_slowest.md). Top bottleneck: q72 SMJ catalog_sales⋈inventory shuffles 189MB — should broadcast.","ticket":"set-up-spark-tpc-ds-baseline-sf-1","owner":"nicktoper"}
