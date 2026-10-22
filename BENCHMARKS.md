# VERGE Benchmarks

## Primary objective

**Minimize total cost per successfully completed, independently verified
task**, subject to correctness, security and reliability constraints.

## Total cost model (normative)

For one task completion:

```
cost(task) = C_llm + C_classify + C_compile + C_verify + C_lookup + C_execute + C_ops
```

- `C_llm`: LLM calls × priced tokens (simulated backends report token
  counts; USD only when a real priced backend is configured)
- `C_classify`: cascade-lane classification time
- `C_compile`: trace→graph compile time
- `C_verify`: static + property + differential verification time (case
  generation + executions)
- `C_lookup`: registry exact-signature lookup time
- `C_execute`: sandboxed execution time
- `C_ops`: ledger append + manifest projection writes

All components measured in wall-clock ms and, where applicable, token counts.
Every arm of every experiment reports the full decomposition — no single
number without its parts. Savings are reported as measured deltas with
visible assumptions; **never inferred from synthetic compression to
production**.

## Secondary metrics

- end-to-end latency (p50/p95/p99 across the frozen dataset)
- tokens consumed per task
- verified-capability reuse rate (executions from registry / total tasks)
- execution graph coverage (families served without LLM after warm-up)
- compilation + verification overhead per admitted capability (the
  "investment" side of E6)
- requalification cost on drift (E7)

## Baselines (E5 arms)

1. **always-llm**: backend solves every task, no registry.
2. **no-memory**: multi-round but registry cleared between rounds.
3. **static-registry**: pre-registered capabilities, no learning loop.
4. **cascade-only**: classification-only routing, no capability execution
   (demonstrates inference avoidance without compilation).
5. **verge**: full vertical slice.

## Measurement discipline

- Wall-clock via `time.monotonic()`; each arm runs on the same machine,
  sequential, with warm-up excluded and stated.
- p50/p95/p99 over per-task samples; n reported with every percentile.
- Deterministic seeds; repeated runs (n=3) reported with spread when
  timing-sensitive claims are made.
- Machine context (model, OS, python version) recorded in every result
  manifest.

## Status

Methodology frozen (this document). No benchmark has run yet — see
EVIDENCE_REGISTER.md for the current evidence state; BENCHMARKS result tables
appear here only after runs exist on disk.
