# VERGE Iteration Report — Round 1 (MVP)

Charter §13 format. Evidence-backed; every claim links to on-disk artifacts.
Scope of all claims: research prototype, simulated deterministic backends,
single machine, frozen synthetic datasets.

## Hypothesis

**H0-mvp**: for frozen pure data-transformation task families, after the
first task of a family is solved and verified, subsequent tasks complete
with (a) zero LLM calls, (b) ≥ 99% behavioral correctness on held-out cases,
(c) lower measured total cost per verified task than the always-LLM baseline.

## Implementation

Nine-layer vertical slice, fully wired (`src/verge/`):
- **L1/L2**: `verge.task` contract + Cascade domain-consumer adapter (real
  `cascade-compression@3efd16e` pipeline + structurally-equivalent compat lane).
- **L3/L7/L9**: content-addressed `CapabilityRegistry` + append-only
  hash-chain ledger; legal-transition table; authorization requires passing
  property/differential evidence; ledger-authoritative projection check
  (tampered manifests cannot resurrect revoked capabilities).
- **L4**: simulated α/β backends (honest cost model, `model.kind:
  "simulated"` labels).
- **L5**: IR with greedy canonicalization + structural hashing; trace→graph
  compiler with opset-signature validation.
- **L6**: verification engine (static/property/differential) with
  contract-valid, content-addressed evidence records and policy gates.
- **L8**: 52-op pure opset v1.0.0; sandboxed interpreter (budgets, timeouts,
  output guard; no eval/exec/import/io — source-scan enforced).
- **Engine**: orchestration with verify-before-admission, qualify→authorize,
  revoked-content reactivation only under fresh verification, negative-
  evidence retention, per-phase cost ledger, ledger bootstrap for persistence.

## Alternatives considered

- **WASM runtime** (rejected for MVP — D-001): no measured benefit at 52-pure-op scope.
- **Postgres-backed ARE ledger** (deferred — D-002): identical pattern implemented as hash-chain JSONL for hermetic runs.
- **Embedding EAK/agent-promotion code** (rejected): unpicking fixture-bound services costs more than the ~300-line lifecycle built against the Apache-2.0-aligned contract.
- **Cascade 4-label classifier for task routing** (rejected — D-004): label pinning conflicts; domain-consumer pattern instead.

## Verification (what actually ran)

- Five-part matrix, all GREEN: TDD 692, BDD 8 scenarios/50 steps, EDD 3,
  CDD 23, CBT 9 — `rubric/score.py`, commit `4ae5184`.
- Every module red-first with recorded red→green counts (agents C/D/E/F + principal).
- Six experiments executed with frozen datasets/manifests:
  E1-sim, E2, E5, E6, E7, E8 — all PASS, results under
  `experiments/*/results/run-1/result.json`, commit `713193e`.
- Independent verification of each agent delivery (re-run + source audit), not endorsement.

## Performance (measured)

- E2: 40 tasks, 100% correct, 5 LLM calls (one per family), 3.75 ms/task.
- E5: verge 5 LLM calls vs always-llm/no-memory 90 (94% avoidance);
  held-out correctness 100% on all arms (30 cases, seed 9999).
- E1-sim: 100% behavioral equivalence (α vs β, 50 cases); structural identity
  0% (reported separately — reversed commutative order is correctly NOT folded);
  codegen baseline 0% exact / 0% AST-normalized agreement.
- E8: 100% transfer correctness, zero re-reasoning calls.

## Economics

- E6: 1.84 ms per verified task (full decomposition: classify/llm/compile/
  verify/lookup/execute). Compile+verify investment ≈ 7.0 ms/family;
  break-even 58.9 tasks/family **on local ms** — an artifact of the simulated
  LLM being nearly free locally. The transferable figure is token avoidance:
  85 LLM calls (≈85k+ tokens) avoided on the e5 run. No USD pricing claimed
  (no configured spending limit; tokens counted, not priced).

## Failures and uncertainties

- Simulated backends ⇒ mechanism evidence only, NOT model independence (E1/E8
  claims scoped accordingly; real-model runs need an authorized API budget).
- Canonicalization is sound but incomplete for structural twins (Agent C's
  locked-in negative evidence); no commutative folding (Phase 4 equality saturation).
- Local JSONL ledger cannot detect tail forgery/truncation (unanchored chain);
  production target is are-immutable-ledger.
- E6 ms-based break-even overstates the required volume under real LLM costs
  (favorable direction, still reported honestly).
- E3 (duplicate discovery at scale) and E4 (graph evolution/optimizer) not built (Phase 4).
- cascade-only baseline excluded from E5 with recorded reason (classification
  without execution produces no data-transform outputs).

## Decision

**Retain and promote the MVP as the Phase 3 baseline.** H0-mvp is supported
within its declared scope: (a) zero-LLM reuse ✓, (b) 100% ≥ 99% held-out ✓,
(c) lower cost per verified task than always-llm on token avoidance ✓ (ms
parity explained by free simulated LLM). No safety invariant failed. The
hypothesis is NOT established beyond simulated backends and synthetic
data-transform families — Phase 5 (real models, broader domains) is where it
faces genuine risk.

## Next experiment

**Phase 4, E4 (recursive graph evolution)**: build the rewrite generator +
benchmark-guided selection, retaining multiple implementations per family
(Pareto). It is the highest-value untested mechanism (the "Recursive Graph
Evolution" in the name), it exercises the verification machinery hardest
(rewrites must re-verify), and its failure modes are informative either way.
Precondition: wire llm-d-sc custom taxonomy lane for broader task routing.
