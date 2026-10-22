# VERGE Decision Log

Format: ID, date, decision, rationale, alternatives considered, reversal
conditions.

## D-001 — 2026-10-21 — MVP runtime: pure-Python interpreter, not WebAssembly

The charter says "Evaluate WebAssembly as an initial portable execution
target, but select the backend based on measured suitability." With a closed
52-op pure opset, a Python interpreter achieves the same sandbox guarantee
(no eval/exec/import/io, enforced by source-scan test) with zero toolchain
cost and full determinism. WASM adds value only with side-effecting ops or
cross-language portability — neither exists in MVP.
Alternatives: WASM (wasmtime) — rejected for MVP: no measured benefit at this
scope, adds dependency weight. Revisit at Phase 4 with measurements.

## D-002 — 2026-10-21 — Ledger: local hash-chain JSONL stand-in

are-immutable-ledger is the pinned production substrate but requires
PostgreSQL + service deployment. Experiments must be hermetic and
reproducible on a laptop. The JSONL chain implements the identical pattern
(append-only, prev-hash chaining, tamper-detecting verify) and is tested for
it. Upgrade path: swap `HashChainLedger` for the ARE Python SDK behind the
same interface.

## D-003 — 2026-10-21 — Simulated deterministic backends for all experiments

No paid API usage without a configured spending limit (charter §10). All
experiment backends are pure deterministic functions of (task, seed,
identity), labeled `model.kind: "simulated"` in every trace. Real-API
adapters are interface-compatible but only run behind explicit budget
config. Consequence (accepted): E1/E8 demonstrate mechanism, not real-model
independence — recorded in RESEARCH_HYPOTHESES.md.

## D-004 — 2026-10-21 — Cascade integration as domain consumer

Recon confirmed Cascade OSS hard-pins 4 actionability labels. VERGE does not
modify Cascade semantics (charter §L2); it registers its own CascadeAgent
(VERGE task-pattern recognition) against Cascade's published protocol, and
maps verge.task ⇄ cascade.signal (priority→severity). The MVP ships a
documented compatible lane with identical gate semantics; live
cascade-compression service wiring is Phase 2+.

## D-005 — 2026-10-21 — MVP capability scope: pure data transformations only

Permissions contract const-locks `side_effects: "none"`, `network: false`,
`filesystem: false`. Charter §8: "Do not introduce privileged or irreversible
actions in the initial MVP." Side-effecting capabilities require a future
contract major version.

## D-006 — 2026-10-21 — Owner reuse grant recorded

Owner statement (2026-10-21): "anything in my repo is fair game to use."
All jkershawrh repositories — including unlicensed ones (earned-authority-kit,
agent-promotion, governed-cognitive-loop, cost-aware-inference-demo) — are
authorized for code reuse in VERGE. Exclusions: rhpds/launchpad fork
(upstream Red Hat code, reference-only) and any upstream third-party code
retained inside those repos. For any future external release, licensing must
be regularized before publishing those derivatives.

## D-007 — 2026-10-21 — Rubric adoption (TDD/EDD/CDD/BDD/CBT + red/amber/green)

Canonical definitions adopted verbatim from the owner's ecosystem
(`~/.config/opencode/agents/frontend.md`, `tca/rubric/score.py`,
cascade-routing evidence reports): TDD red-first; EDD freeze-before-evaluate;
CDD boundary validation through versioned contracts; BDD given/when/then
including failure paths; CBT capability journeys; RAG statuses never averaged
into production claims. Suite layout mirrors tca. Ordering follows Cascade
house methodology: CDD → TDD → EDD → BDD.

## D-008 — 2026-10-21 — Toolchain: uv + CPython 3.14, editable install

uv 0.11.29 with pinned `uv.lock`; runtime deps minimal (jsonschema only);
dev deps pytest/behave/hypothesis. Hermetic, reproducible, fast.

## D-009 — 2026-10-21 — Verification engine decoupled via dependency injection

The verification engine receives `graph_validator` and `executor` as injected
callables (Protocol-typed) instead of importing IR/runtime. Rationale:
independent testability, independent replaceability, and no import-order
coupling while modules are built concurrently. The engine layer wires the
real implementations.

## D-011 — 2026-10-21 — Revoked-content reactivation policy (reviewer W-2)

When identical capability content is re-derived after revocation and fresh
verification passes, the engine issues a ledger-recorded `reactivate`
transition (revoked → candidate → qualify → authorize) rather than refusing
re-admission. Rationale: revocation's purpose — stopping execution of an
untrusted/stale artifact — is honored because execution halts until fresh
static+property+differential verification passes; the full cycle is
audit-visible in the promotion ledger. KNOWN LIMIT (risk R-13): the
revocation reason does not influence the re-authorization decision, and no
permanent-revocation (content-hash blacklist) exists yet. Phase 4 adds a
revocation class (advisory | permanent) and a blacklist gate in admit/
reactivate. This decision was made at implementation time and is now
explicitly governance-documented per the independent reviewer's finding.

## D-012 — 2026-10-21 — Independent review verdict adopted

Cold reviewer (commit `9154610`): SHIP-WITH-CAVEATS. Quality gate and all
six experiments reproduced exactly; invariants verified live; honesty
posture assessed as exemplary. Caveats W-2 (reactivation governance) and
W-5 (evidence-register staleness) remediated same day (R-13 added, register
rows 7–10 updated to GREEN with evidence, duplicate row removed); W-3
(opset expressiveness) risk-registered as R-14.

## D-013 — 2026-10-21 — Phase 5a: real local-model backends + oracle seam

Real models enter via `OllamaBackend` (OpenAI-compatible local endpoint,
`model.kind: "real-api"`, REAL server-side token counts, bounded
retry-with-feedback on format errors, temperature 0 with documented hardware
nondeterminism — rates, not capability ids, are comparable across runs).
Local Ollama models (0.5b–8b, two families) need no API budget; the same
backend targets a remote GPU cluster vLLM/KServe endpoints when access is sorted
(endpoint-agnostic by design).

The oracle seam: real models have NO `reference` (a model is not its own
ground truth). The engine now takes an injected `oracle_provider`; wiring
supplies task-level ground truth (family references — the task spec, never
the proposer) and skips differential when no oracle exists (property
evidence still gates admission; differential is not a required gate).
First real results (smoke, commit `41067ec`): 7–8b models emit
format-valid, contract-valid, compilable traces at 100% (the hard part);
semantic correctness varies by model and family (granite3.3:8b CORRECT on
abs_sum with self-repair; qwen2.5:7b over-composes on normalize) — wrong
proposals are caught by verification and cost only their tokens, which is
the platform's designed behavior. Prompt template v3 (strategy + stop rule)
is the frozen controlled variable for e1-real/e8-real.

## D-014 — 2026-10-21 — Tiered real-model study design

The capability-threshold curve (proposal validity / admission rate vs model
tier) is the experiment, not a confound: H0's economics favor cheap
proposers + strong verification. Tier ladder: 0.5b → 1.5b/2b → 7b/8b local
(available), a remote GPU cluster Gaudi 8B/30B (blocked on VPN + kubeconfig), frontier
APIs (blocked on budget authorization).

## D-015 — 2026-10-21 — a LiteLLM API gateway authorized for bounded programmatic use

Owner authorization (2026-10-21): a LiteLLM API gateway Claude models may be used for
VERGE's real-model experiments despite the standing modality rule reserving
them for non-text work. Scope of the exception: structured graph proposals
only (no conversation/reasoning work), bounded call counts (~tens per
experiment run), real server-side token accounting, keys loaded from the
mode-600 file (`~/.config/opencode/a LiteLLM API gateway.key`) and never logged or
serialized. e1-real run-2 uses claude-haiku-4-5 (smoke-verified: correct
minimal program, first attempt, 3s, 1583+234 tokens). The OpenAI-compat
transport (`verge.backends.openai_compat`) is endpoint-agnostic — the same
backend targets vLLM/KServe on a remote GPU cluster when access is sorted.

## D-016 — 2026-10-21 — Opset v1.2: composed element operations

E13 measured the difficulty frontier: D1 (lexical) clearable, D2
(structural) and D3 (conditional) blocked by the composed-element-ops gap.
The v1.2 design adds three bounded features, each verifiable:

1. **Per-op configs in element chains**: `list.map` config `{"ops": [...]}`
   extended from name-only to `{"op": <name>, "config": {...}}?` per stage.
   Unlocks D2 (chained map.get with different keys per stage).

2. **Composed filter predicates**: `list.filter` accepts the same chain
   form; the final op's output must be boolean (statically checked).
   Unlocks filter-by-field scenarios.

3. **`logic.case` op**: multi-way threshold mapping —
   `logic.case(value, config={"cases": [{"gte": N, "then": V}, ...],
   "default": D})`. Deterministic, first-match semantics. Unlocks D3.

Deliberately EXCLUDED from v1.2: recursion/tree traversal (D4 — requires
a v2 side-effect-class design), multi-list zip (D4-class), arbitrary
nested composition (the verification tractability boundary).

Verification impact: each chain element validates against its op
signature + config shape; logic.case is property-testable (threshold
boundaries, empty cases, single case); the closed-world guarantee is
preserved (no new op families, only composition of existing ops + one
new bounded op).

## D-010 — 2026-10-21 — Ledger is source of truth; manifests are projections

Authority state lives in the append-only promotion ledger; capability
manifest files are rebuildable projections (pattern adopted from
shared-enterprise-memory's ledger+projection architecture). Manifest tampering
is detectable via content-address bindings and ledger replay.
