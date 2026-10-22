# VERGE Iteration Report — Round 2 (Phase 4: governance hardening + graph evolution)

Charter §13 format. Continues `ITERATION_REPORT_ROUND1.md`. All claims link
to on-disk evidence; scope: research prototype, simulated backends, single
machine.

## Hypothesis

Two: (a) R-13 governance — permanent revocation must be un-circumventable at
the API level; (b) E4 — verified rewrites can be generated, retained, and
selected by benchmark, beating naive first-implementation retention.

## Implementation

- **Revocation classes** (P4-A, verified): `revocation_class` contract field
  (advisory | permanent), ledger-replay verdict, permanent-reactivate
  refusal, content blacklist on `admit`, graceful engine degradation.
- **Adversarial hardening round 1** (principal): closed 6 of 12 bypassed
  vectors — evidence-graph binding at authorize, fresh-evidence requirement
  for reactivation, `get_graph` content re-hash, engine intake guards
  (canonical-JSON + own-schema), ledger-authoritative `authorized()`
  (closes the wiring-omission vector). 8 red-first regression tests that
  reproduce the original bypasses.
- **E4 graph evolution** (P4-B, verified): opset v1.1.0 map-chain fusion
  (additive config form; pin-satisfaction version equality, documented),
  rewrite engine (fusion + commutative-swap), paired benchmark with
  cost-floor rule, registry-closing benchmark selector on the engine's
  `selection_policy` seam, full experiment with EDD-frozen manifest.

## Verification (what actually ran)

- Independent re-verification of P4-A and P4-B deliveries (suite re-runs +
  source audit; P4-B's version-pin mechanism reviewed for equality symmetry).
- Adversarial validation: 24 probes, all artifacts retained under
  `experiments/adversarial/` (12 BYPASSED / 4 PARTIAL / 8 HELD).
- Round-2 five-part matrix: **PASS, all GREEN** — TDD 757, BDD 8, EDD 3,
  CDD 23, CBT 10 (801 executed tests).
- E4 experiment: PASS on all four frozen thresholds; determinism of
  capability/evidence ids across runs verified.

## Performance (measured)

- E4 fused rewrite: strictly fewer node evaluations on every input
  (200015 → 200010 on the 20k stress case); paired median wall speedup
  ~1.007x — interpreter-internal and near machine noise, reported as such.
  The deterministic improvement signal is the evaluation count, not wall ms.
- Selection demo: a second engine with the benchmark selector serves the
  fused capability from the registry lane with correct output.

## Economics

No new cost claims this round. E4's rewrite+verify investment is measurable
in the cost ledger; the honest conclusion is that at prototype scale the
fusion saving does not justify its verification cost — the mechanism's value
is structural (retained alternatives + selection), and its economics must be
re-measured where per-execution cost is real.

## Failures and uncertainties

- 6 adversarial bypasses remain open with concrete attack evidence:
  ledger rewrite/truncation (R-07, upgraded), forged evidence +
  proposer-as-oracle (R-15), unauthenticated principals (R-16), learnable
  generator distribution + weak vacuity guard (R-17), runtime memory/intra-op
  timeout (R-18), alias-spelling duplicates (R-19).
- E4 coverage is workload-bounded: only 1 of 5 families has a fusable chain.
- The ~1% wall speedup is within machine noise; only the evaluation-count
  reduction is deterministic.

## Decision

**Retain.** R-13 is closed end-to-end; six adversarial vectors are closed
with regression pins; E4's mechanism (rewrite → verify → retain → select) is
demonstrated with honest near-noise speedup framing. The remaining six
vectors are registered with phased fixes and are the top of the Phase 5
backlog alongside real-model experiments.

## Next experiment

**Phase 5 gate decision**: (1) anchor the ledger (are-immutable-ledger
deployment) to close R-07 — the highest-severity open finding; or (2) run
E1/E8 against real model backends behind an authorized spending limit — the
central hypothesis's real test. Both need owner input (infrastructure or
budget authorization); the smallest next action that needs no authorization
is R-17's adversarial case generation (mutation-based inputs outside the
generator's distribution), which hardens every verification claim at once.
