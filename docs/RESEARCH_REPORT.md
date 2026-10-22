# VERGE: Verified Execution & Recursive Graph Evolution

**A model-agnostic computational substrate for accumulating verified
executable capabilities — system, evidence, and boundaries.**

Research report. Every claim in this report links to an on-disk artifact;
see the evidence register for the full claim chain.

---

## Abstract

We test the hypothesis that a model-independent computational substrate
can accumulate verified executable capabilities through program synthesis,
graph composition, and governed optimization — reducing the inference and
computational cost of future tasks without retraining foundation models or
sacrificing correctness. We built VERGE: a platform that compiles LLM
solution proposals into typed dataflow graphs over a closed pure-op
operating set, admits them only through static, property-based,
differential, and adversarially-seeded verification, and serves them
thereafter from a content-addressed registry at zero LLM cost. In
experiments on frozen datasets with real models, a frontier-class proposer
(claude-haiku-4.5) qualified five task families for a one-time cost of 6
LLM calls / 11,322 tokens; subsequent execution — including by a weaker
model that cannot compose the tasks itself — was 100% correct at zero
marginal LLM cost. Local 7–8B proposers admitted 3/5 families each with
complementary failures (union 4/5), and their independently-derived
programs were behaviorally equivalent (10/10) on jointly-solved families.
Cross-proposer consensus showed precision 1.0 with zero false agreements
(n=20 tasks, three independent architectures). A production signal stream
(17.6M signals, 171 type families) exhibited 103,051× family repetition,
satisfying the economic necessary condition by five orders of magnitude.
Adversarial validation (24 probes) bypassed 12 governance vectors; 8 were
closed with regression tests. We conclude the mechanism is validated
within its scope (pure data-transformation families, synthetic frozen
datasets, prototype infrastructure), and we identify opset expressiveness
— not the accumulation mechanism — as the binding constraint on
generality.

## 1. The hypothesis

> A model-independent computational substrate can accumulate verified
> executable capabilities through multi-agent interaction, program
> synthesis, graph composition, and governed optimization — reducing the
> inference, latency, and computational cost of future tasks without
> retraining foundation models or sacrificing correctness.

Treated as falsifiable throughout: every experiment had pre-frozen
datasets, pre-declared thresholds, and retained negative evidence.

## 2. System definition

VERGE (Python, ~10 modules, 900+ tests across five suites) implements one
loop:

1. **Task intake** — typed, versioned contracts with acceptance criteria;
   non-canonical inputs and self-inconsistent schemas rejected at the door.
2. **Routing** — Cascade-derived classifier (domain-consumer adapter):
   known family → registry; novel → generative lane.
3. **Proposal** — any OpenAI-compatible endpoint or Ollama server emits a
   structured solution trace (bounded retries with format feedback; real
   token accounting; no self-oracle).
4. **Compilation** — traces compile to typed dataflow graphs over a closed
   53-op pure opset (v1.2); greedy canonicalization makes structural
   identity naming- and order-independent; graphs are content-addressed.
5. **Verification** — static validation, property testing (declared
   expectations + schema-conformant generation), differential testing
   against task-level ground truth, and an adversarial case catalogue
   (boundary lengths, unicode, empties, extremes). Empirical confidence is
   never conflated with proof; every evidence record carries a mandatory
   claim scope.
6. **Governance** — candidate → qualified → authorized, with evidence-bound
   authorization, advisory/permanent revocation with content blacklisting,
   fresh-evidence reactivation, and a hash-chained promotion ledger as the
   authority source of truth (optionally anchored on a Postgres-backed
   immutable ledger for tamper evidence).
7. **Execution** — sandboxed interpreter: closed op registry, no
   eval/exec/import/io, node-evaluation budgets, timeouts, output-size
   guards.
8. **Memory** — content-addressed capability/graph/evidence store with
   exact-signature lookup; cross-model reuse requires no access to the
   originating reasoning.

The design separates **discovery, verification, authorization, and
execution**; a verified capability never automatically gains authority.

## 3. Method

- **Rubric**: TDD/EDD/CDD/BDD/CBT with red/amber/green evidence statuses;
  a passing unit test never turns a capability green; statuses are never
  averaged into production claims.
- **EDD**: all datasets and thresholds frozen before measurement.
- **Independent verification**: every component built by a subagent was
  re-run and source-audited; a cold reviewer reproduced the quality gate
  and all experiments; an adversarial agent attacked the live system
  (24 probes, artifacts retained).

## 4. Results

### 4.1 The tier curve (real models, frozen 50-case dataset)

| Proposer | Admission | One-time qualification cost | Reuse correctness |
|---|---|---|---|
| claude-haiku-4.5 (API) | **5/5** | 6 calls, 11,322 tokens | 50/50, 0 LLM |
| granite3.3:8b (local) | 3/5 | 5 calls, 9,050 tokens | 30/30, 0 LLM |
| qwen2.5:7b (local) | 3/5 | 4 calls, 6,001 tokens | 30/30, 0 LLM |

- Local failures were **complementary** — a two-proposer local pool covers
  4/5 families.
- Cross-model behavioral agreement on jointly-solved families: **10/10**.
- **Multi-proposer pooling**: a cheap-first pool (granite → qwen → claude)
  reached **5/5 coverage with the frontier model invoked exactly once**
  (1,841 tokens) — for the single family no local model could compose —
  and 50/50 reuse at zero LLM. Pairwise equivalence across all three
  proposer pairs: **80/80**.

### 4.2 Cross-model transfer (the thesis in one result)

Capabilities qualified by claude-haiku were executed by qwen2.5:7b — a
model that cannot compose these tasks itself — **50/50 correct with zero
transferer LLM calls**. No traces, no reasoning transfer: only the
content-addressed graph crossed the model boundary.

### 4.3 Consensus oracle (oracle-free admission evidence)

When two or more independent proposers behaviorally agreed on a task's
output (across adversarial + seeded probe inputs), **every consensus
member was correct** (precision 1.0, 5/5 consensus tasks, zero false
agreements across all agreeing pairs). Solo baselines: claude 7/20,
granite 4/20, qwen 4/20. Consensus is a **precision-first, low-recall
oracle** — it admits only what independent minds agree on (25% coverage).

### 4.4 Accumulation and economics

- Self-compilation: 40 tasks, 100% correct, 5 LLM calls (one per family).
- Cumulative learning: 5 LLM calls vs 90 for always-LLM at equal 100%
  held-out correctness.
- Production repetition: **17,621,793 signals / 171 type families
  (103,051× repetition)**; hypothetical accumulation avoidance 99.999%.
- Graph evolution: map-chain-fusion rewrites verified and retained;
  benchmark-guided selection serves the fused capability.

### 4.5 Difficulty frontier (graded coding-task ladder)

On a five-level ladder (D1 lexical → D2 structural → D3 conditional →
D4 tree recursion → D5 codegen), with opset v1.1:

| Level | Validity | Compile | Admission | Median nodes |
|---|---|---|---|---|
| D1 lexical | 75% | 75% | **25%** | 5 |
| D2 structural | 92% | 92% | **0%** | 4 |
| D3 conditional | 75% | 75% | **0%** | 2 |
| D4 tree | 42% | 42% | **0%** | 1 |
| D5 codegen | 83% | 83% | **0%** | 6 |

D2 and D3 have high compile rates but zero admission — **the barrier is
opset expressiveness, not proposer format-ability**. Opset v1.2 (per-op
config chains, composed filter predicates, logic.case) addresses D2/D3.

### 4.6 Governance and adversarial evaluation

- Enforcement: revoked/expired/tampered/schema-drifted/unevidenced
  capabilities all refused (6/6 scenarios).
- Adversarial validation: 24 probes — sandbox purity and canonicalization
  soundness HELD; 12 governance vectors BYPASSED, of which 8 were closed
  with regression tests. 4 remain registered with phased fixes.

## 5. Interpretation

The mechanism chain — **real proposal → verification → admission →
zero-LLM cross-model reuse** — held at every model tier tested. The
economically interesting shape emerged exactly as the hypothesis
predicted: correctness is enforced by verification rather than trusted
from the proposer, so cheap, unreliable, and complementary proposers
become safe contributors, and one strong proposer's work becomes every
model's capability.

## 6. Limitations and threats to validity

1. **Opset expressiveness bounds generality** — measured: 2/15 realistic
   transform tasks as-is; 10 genuine gaps in two classes (missing op
   families; element-op composition limits).
2. **Five synthetic families** — no claim about production workloads
   beyond the measured repetition rate.
3. **Verification is empirical, never proof** — finite-sample agreement
   with declared oracles; adversarial catalogue is learnable.
4. **One frontier proposer tier** — haiku-4.5 only; stronger tiers untested.
5. **Hardware nondeterminism** at temperature 0 — rates, not capability
   ids, are comparable across runs.
6. **The economics assumption is measured on one production stream** —
   observability signals, not data-transform tasks.

## 7. Related work

VERGE builds on an existing governed-inference ecosystem: Cascade
Compression (selective reasoning — VERGE is a domain consumer);
earned-authority-kit and agent-promotion (qualification lifecycles — VERGE
implements the capability-scoped variant); are-immutable-ledger (the
production evidence substrate); gcl-oss (proposal-only governance kernel).
The novel combination is the **content-addressed, verification-gated,
cross-model capability registry** — the demonstration that a capability
qualified by one model can be executed correctly and LLM-free by a
different model that cannot derive it is not present in any surveyed
repository.

## 8. Reproduction

See REPRODUCING.md in the repository. Everything runs on a single laptop.

## 9. Status

Phases 0–5a complete; the platform includes a unified MCP gateway
(14 tools), a Cascade-memory oracle (verification without hand-written
test cases), Kafka topic design, domain kit provisioning, and a five-phase
product roadmap. The full decision trail is in DECISION_LOG.md.
