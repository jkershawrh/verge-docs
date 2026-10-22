# VERGE Research Hypotheses

Every hypothesis is falsifiable, with pre-declared thresholds (EDD: frozen
before evaluation). Status changes only via evidence linked in
EVIDENCE_REGISTER.md. The central hypothesis is treated as falsifiable, not
assumed.

## H0 — Central hypothesis

**A model-independent computational substrate can accumulate verified
executable capabilities through multi-agent interaction, program synthesis,
graph composition, and governed optimization — reducing the inference,
latency, and computational cost of future tasks without retraining
foundation models or sacrificing correctness.**

MVP operationalization (H0-mvp): for a frozen set of pure data-transformation
task families, after the first N tasks of a family are solved and verified,
subsequent tasks of the same family complete with (a) zero LLM calls,
(b) ≥ 99% behavioral correctness on held-out cases, (c) lower measured total
cost per verified task than the always-LLM baseline.
Falsified if any of (a),(b),(c) fails on the frozen evaluation set.

## Experiment hypotheses (E1–E8, from the VERGE charter)

| ID | Hypothesis | Success threshold | Status |
|---|---|---|---|
| E1 | Two backends solving identical tasks produce canonical graphs that are structurally identical or behaviorally equivalent more often than direct code generation produces equivalent code | ≥ 90% behavioral equivalence on frozen set; report structural-identity rate separately | **PASS (simulated)** — 100% behavioral equivalence, 100% correctness, structural identity 0% (reported separately); codegen baseline 0% exact / 0% AST-normalized; `experiments/e1_sim/results/run-1/result.json` · **PASS (real, frontier tier)** — claude-haiku-4-5 via a LiteLLM API gateway: 5/5 families admitted, 50/50 reuse correctness, 6 LLM calls / 11.3k tokens total qualification cost; `experiments/e1_real/results/run-2-a LiteLLM API gateway/result.json` · local 7–8b tier run in flight |
| E2 | Successful agent trajectories contain stable computational subgraphs that compile into capabilities whose subsequent executions match the original workflow's outputs | 100% output match on declared cases; ≥ 99% on generated cases | **PASS** — 100% on all 40 frozen cases; 5 LLM calls for 40 tasks; `experiments/e2_selfcompile/results/run-1/result.json` |
| E3 | Models propose duplicate/equivalent operations; dedup by content hash + differential verification admits one canonical capability per equivalence class | 0 unauthorized duplicates in registry; ≥ 1 demonstrated dedup | partial (mechanism present): idempotent admit deduplicates identical content (tested); broader proposal-space dedup deferred to Phase 4 |
| E4 | Alternative execution graphs (rewrites) can be generated, verified, and retained; benchmark-guided selection beats naive retention on cost | ≥ 1 verified alternative retained; measured improvement on its target workload | **PASS (mechanism, simulated)** — 2 alternatives retained (1 map-chain-fused, 1 commutative-swap); fused correctness 12/12 workload + 5/5 stress; strictly fewer node evaluations on every input; paired median speedup ~1.007x on 20k-element stress (interpreter-internal, near machine noise — the deterministic evidence is evaluation count); selection demo serves the fused capability; `experiments/e4_graph_evolution/results/run-1/result.json` |
| E5 | Registry evolution between rounds transfers to unseen structurally-related tasks vs baselines (always-LLM, no-memory, static registry, cascade-only) | VERGE arm ≥ baselines on correctness; strictly lower LLM calls per task; lower total cost per verified task | **PASS (simulated)** — held-out 100% (≥ all baselines), 5 vs 90 LLM calls; cascade-only excluded with recorded reason; `experiments/e5_cumulative/results/run-1/result.json` |
| E6 | Expected value of compilation exceeds its lifecycle cost at realistic task volumes | break-even volume identified and reported with visible assumptions | **PASS (prototype-scale)** — break-even 58.9 tasks/family on local ms; honest caveat: simulated LLM is nearly free locally, token-avoidance (85 calls) is the transferable figure; `experiments/e6_economics/results/run-1/result.json` |
| E7 | Unsafe/invalid/stale/revoked capabilities cannot execute | 100% rejection of revoked/expired/tampered; drift scenarios demote | **PASS (registry scope)** — 6/6 scenarios, 13/13 checks; `experiments/e7_revocation/results/run-1/result.json` |
| E8 | A capability qualified by one backend is reused by another without the original trace | 100% correctness on cross-model reuse cases; zero re-reasoning calls | **PASS (simulated)** — 100% transfer correctness, 0 beta LLM calls; `experiments/e8_transfer/results/run-1/result.json` · **PASS (real)** — claude-haiku-4-5 qualified 5/5 families; qwen2.5:7b (a model that cannot solve these tasks itself, per smoke) executed all 50 frozen cases 50/50 correct with ZERO LLM calls; `experiments/e8_real/results/run-1/result.json` |

## Honesty constraints on all of the above

- MVP experiments run on **simulated deterministic backends** (labeled
  `model.kind: "simulated"` in every trace). They demonstrate the *mechanism*
  (canonical agreement, verification, cross-model reuse) — NOT independence
  between real foundation models. Real-model E1/E8 require an authorized API
  budget and are explicitly out of MVP scope.
- Behavioral equivalence on finite case sets is **empirical confidence, never
  proof**; every evidence record's `claim_scope` states this.
- Synthetic-workload cost savings are **not** production savings; economics
  results are reported as measured with visible assumptions only.
