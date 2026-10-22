# VERGE Experiments

Protocols are frozen before evaluation (EDD). Each experiment has a manifest
conforming to `verge.experiment` in `experiments/<id>/manifest.json` with the
dataset digest recorded before any candidate is measured.

## MVP experiment set (Phase 3)

| ID | Title | Compares | Frozen dataset | Runner |
|---|---|---|---|---|
| e1-sim | Cross-backend graph agreement (simulated) | backend α vs backend β on identical tasks: canonical-identity rate, behavioral equivalence rate, vs direct-codegen baseline | `experiments/e1-sim/dataset.jsonl` | `uv run python -m experiments.e1_sim.run` |
| e2-selfcompile | Self-compiling workflow | first-pass backend+trace vs subsequent registry executions: output match, LLM calls, cost | `experiments/e2-selfcompile/dataset.jsonl` | `uv run python -m experiments.e2_selfcompile.run` |
| e5-cumulative | Cumulative learning vs baselines | VERGE vs always-LLM vs no-memory vs static-registry, rounds of related tasks, held-out final round | `experiments/e5-cumulative/dataset.jsonl` | `uv run python -m experiments.e5_cumulative.run` |
| e6-economics | Total cost per verified task | cost ledger decomposition across arms; compilation break-even volume | e5 dataset + volume sweep | `uv run python -m experiments.e6_economics.run` |
| e7-revocation | Drift, failure, revocation | adversarial scenarios: tampered manifest, schema change, revoked/expired execution attempts, failed verification | `experiments/e7-revocation/scenarios.json` | `uv run python -m experiments.e7_revocation.run` |
| e8-transfer | Cross-backend capability transfer | backend α qualifies; backend β retrieves+executes with no trace access | e1 dataset, reversed roles | `uv run python -m experiments.e8_transfer.run` |

## Phase 5a/5b — real-model experiments

| ID | Title | Models | Status |
|---|---|---|---|
| e1-real | Real cross-model graph agreement | granite3.3:8b + qwen2.5:7b (run-1); claude-haiku-4.5 (API) (run-2-a LiteLLM API gateway) | **PASS** — tier curve 5/5, 3/5, 3/5 (complementary; union 4/5); cross-model agreement 10/10 on jointly-solved |
| e8-real | Real cross-model transfer | claude→qwen (run-2-a LiteLLM API gateway); granite→qwen (run-1) | **PASS** — 50/50 and 30/30 correct, zero transferer LLM calls |
| e9-proposer-pooling | Multi-proposer pooled registry with cheap-first fallback | granite + qwen + claude on one registry; isolated pairwise equivalence (phase B) | **PASS** — 5/5 coverage, ONE frontier call, 50/50 reuse zero-LLM; 80/80 pairwise equivalence |
| e10-opset-expressiveness | Opset coverage on realistic 15-task corpus | claude-haiku | **MEASURED** — 2/15 as-is, 4/15 with repair; 10 genuine gaps in two classes; ranked v1.2 agenda |
| e11-repetition | Family-repetition on REAL production stream (an internal cluster) | n/a (data analysis) | **PASS** — 17.6M signals / 171 families (103,051×); 99.999% avoidance |
| e12-consensus | Consensus oracle (cross-proposer agreement) | claude + granite + qwen | **PASS** — precision 1.0, zero false agreements, coverage 5/20 (conservative) |
| e13-coding-ladder | Graded coding-task difficulty frontier | claude + granite + qwen | **v1.1 baseline complete; v1.2 re-run in flight** — D1-only frontier (25% admission); D2-D5 at 0% admission despite 75-92% compile rates |

## Protocol rules (all experiments)

1. **Freeze first**: dataset file written + SHA-256 recorded in the manifest
   before any run; thresholds declared in the manifest before any run.
2. **Deterministic**: fixed seeds for case generation; simulated backends are
   pure functions of (task, seed, backend identity).
3. **Reproducible command**: the manifest's `runner.command` re-executes the
   experiment end-to-end; result manifests are written to
   `experiments/<id>/results/run-<n>/result.json` including environment
   (python version, verge version, git commit).
4. **Negative evidence retained**: failed runs and their artifacts stay on
   disk; a hypothesis is never marked on intention.
5. **Honest labeling**: result manifests carry `backend_kind: "simulated"`
   and claim scopes copied from the underlying evidence records.

## Dataset construction (MVP)

Task families are pure data transformations over synthetic data, e.g.:
- `data.string.normalize_email_list` (trim + lowercase)
- `data.list.extract_domain_counts` (split, map, distinct, count via group)
- `data.record.filter_and_sort` (filter by predicate, sort)
- `data.string.template_assembly` (concat/slice/replace pipelines)
- `data.number.aggregate_metrics` (sum/min/max arithmetic)

Each family: 10 declared cases with expected outputs + schema for generated
cases. Held-out split for E5 final round is generated with a different seed
and never used during registry evolution.

## Full E1–E8 charter mapping

E3 (autonomous capability discovery) and E4 (recursive graph evolution) are
Phase 4: E3 needs the dedup/equivalence machinery over a broader proposal
space; E4 needs the optimizer (rewrite generation + benchmark-guided
selection). Their MVP precursors run inside e1-sim/e5-cumulative (duplicate
detection via content hash is already enforced by the registry).
