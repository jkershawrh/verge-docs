# The Governed AI Agent Platform

**A whitepaper on qualified, verified, governed, and auditable AI execution**

---

## Executive summary

Enterprises are deploying AI agents that make decisions and take actions at
scale. The cost is enormous — every agent invocation requires a large
language model call, every output is unverified, every action is unaudited,
and every model dependency creates vendor lock-in. Current AI governance
frameworks address fragments of this problem; none address all of it.

We present a governed AI agent platform that solves the full problem through
nine cooperating components: model qualification (eval-cards), inference
compression (Cascade), verified execution (VERGE), earned authority (EAK),
real-time safety (Guardian), cryptographic identity (agent-passport),
immutable audit (ARE), value attribution (VEF), and intelligent routing
(SDAVL) — connected by MCP tools and Kafka events.

The platform's core insight is that **AI work becomes an asset, not an
expense**. When an LLM solves a task, the solution is compiled to a typed
computational graph, verified against evidence, and stored in a
content-addressed registry. Every subsequent execution of the same task
type is free, instant, and model-independent. On a measured production
signal stream (17.6M signals, 171 type families), this approach achieves
99.999% LLM-call avoidance.

The platform is deployed on Red Hat OpenShift, consumable via MCP (Model
Context Protocol), and onboards new domains through configuration (domain
kits) rather than code.

---

## 1. The problem

### 1.1 AI agents act, but nobody governs the acting

The industry has solved AI agent *decision-making* — routing, tool calling,
multi-step reasoning. What it hasn't solved is AI agent *governance*:

- **Can this model do this task?** (qualification)
- **Is this specific output correct?** (verification)
- **Who authorized this action?** (authority)
- **What actually happened?** (audit)
- **Is this action safe?** (safety enforcement)
- **Can we reuse this work?** (capability accumulation)
- **What did it cost?** (value attribution)

Without answers to these questions, enterprises deploy AI agents at their
own risk. With answers, AI agents become trusted infrastructure.

### 1.2 The economics are unsustainable

Every LLM call costs tokens. For repetitive tasks — alert enrichment,
compliance checks, data normalization — the same reasoning is purchased
millions of times. On a measured production stream, 17.6 million signals
across 171 signal types are processed, with each type averaging 103,051
repetitions. At even $0.01 per LLM call, that's $176,000 to process the
same 171 patterns repeatedly.

The solution isn't cheaper models. It's **not calling the model at all**
for work that's already been done.

### 1.3 Model dependency creates fragility

Every AI automation built today embeds model-specific prompts, outputs, and
behaviors. When the model changes — pricing, capability, availability —
every automation breaks. There's no abstraction layer between "what the
automation does" and "which model produced it."

---

## 2. The solution

### 2.1 The core mechanism: verify, then cache, then reuse

```
1. A task arrives (signal, request, event)
2. The system checks: does a VERIFIED capability exist for this task?
   YES → execute the cached graph (zero LLM cost, milliseconds)
   NO  → an LLM proposes a solution
        → VERGE compiles it to a typed dataflow graph
        → verification checks it against evidence
        → if correct: admit to the registry → execute → cache forever
        → if wrong: reject, retain negative evidence, try again
3. Every decision, verification, and execution is recorded in ARE's
   immutable ledger
```

The key property: **correctness comes from verification, not from trusting
the model**. Wrong proposals fail closed at token cost. Verified proposals
become permanent, free, model-independent capabilities.

### 2.2 The qualification layer: eval-cards

Before a model is trusted with a task class, it's qualified:

- **Construct statement**: what capability is being measured, how it could
  be gamed
- **Repeated-run reliability**: pass@k and pass-all@k with Wilson 95%
  intervals
- **Evidence labels**: LIVE (real invocation), REHEARSAL (retained
  response), OFFLINE (deterministic)
- **Notarization**: the card's exact bytes are chained into an append-only
  hash chain with a portable proof receipt

A consumer reading a card knows: can this model consistently do this task,
with what confidence, and can I prove the measurement wasn't edited.

### 2.3 The compression layer: Cascade

Before an LLM is invoked at all, Cascade checks whether the request matches
a known pattern:

- **Nano tier (85-99% of signals)**: deterministic rules, no LLM
- **Micro tier (1-15%)**: small CPU models for classification
- **Macro tier (<1%)**: larger models for novel or complex cases

Rules are promoted through an evidence-gated lifecycle: draft → candidate →
nano → micro → macro, with a zero-false-negative invariant (any FN →
instant demotion), 72-hour TTL expiration, and shadow validation (5% of
suppressed signals re-checked by the LLM).

### 2.4 The execution layer: VERGE

VERGE is the platform's computational substrate:

- **53-op pure opset (v1.2)**: typed dataflow graphs over string, number,
  list, map, and boolean operations, including per-op config chains,
  composed filter predicates, and multi-way conditional mapping
- **Sandboxed runtime**: no eval/exec/import/io, node-evaluation budgets,
  timeouts, output-size guards — verified by source-scan tests
- **Content-addressed registry**: capabilities identified by SHA-256 digest
  of their canonical form; identical programs have identical digests
  regardless of naming or ordering
- **Verification pipeline**: static validation, property testing, differential
  testing against task-level ground truth, adversarial case generation
  (boundary lengths, unicode, empties, extremes)
- **Promotion lifecycle**: candidate → qualified → authorized, with
  evidence-bound authorization, advisory/permanent revocation, and
  fresh-evidence reactivation
- **Cross-model transfer**: a capability qualified by one model is executed
  by any other model with zero re-reasoning (proven: claude→qwen, 50/50
  correct, zero LLM calls)

### 2.5 The governance layer

- **EAK (Earned Authority Kit)**: authority is earned through evidence, not
  granted by default. Qualification → grant → mutation fence → independent
  readback → withdrawal → requalification.
- **GCL (Governed Cognitive Loop)**: falsification gates reject proposals
  that fail constraint checking. Signed, expiry-bounded proposals only.
- **Guardian**: a Rust-based safety coprocessor that validates, reasons, and
  logs actions in real-time. Drop-in sidecar for any AI workflow.
- **agent-passport**: signed, scoped, time-bound, revocable credentials for
  AI agents — cryptographic identity, not role-based access control.

### 2.6 The audit layer: ARE

ARE (are-immutable-ledger) is a Rust service backed by PostgreSQL:

- SHA-256 hash-chained entries, per entry type
- Database role verified to lack UPDATE/DELETE permissions at startup
- Portable proof receipts for decentralized verification
- Cross-system event correlation
- Deployed on OpenShift, consumable via gRPC, REST, and MCP

Every promotion, verification, execution, and authority change in the
platform flows to ARE. The result is a tamper-evident record of everything
every AI agent did, when, and under whose authority.

### 2.7 The event backbone: Kafka

All components communicate through Kafka topics:

- `signals.raw` — from collectors
- `signals.classified` — from Cascade
- `tasks.submitted` — from agents
- `tasks.completed` — from VERGE
- `capabilities.lifecycle` — from VERGE
- `evidence.produced` — from VERGE and eval-cards
- `authority.changed` — from EAK
- `model.performance` — from eval-cards
- `audit.events` — mirrored from ARE

Kafka enables: historical replay (bootstrap new components with the full
event history), component independence (each consumer processes at its own
pace), and horizontal scaling (consumer groups).

### 2.8 The domain kit: onboarding without code

A new domain is provisioned from a single configuration:

```yaml
apiVersion: verge.dev/v1
kind: DomainKit
metadata:
  name: observability-ops
spec:
  collectors:
    - type: kubernetes-events
    - type: prometheus-alerts
  domain_pack: { signal_types: [...] }
  prompt_template: "You are enriching..."
  oracle:
    source: cascade-memory
  bootstrap:
    kafka_topic: signals.raw
    replay_from: beginning
```

The platform provisions Kafka topics, Cascade domain packs, VERGE prompt
templates, and the oracle — all from this YAML. No code required.

---

## 3. Measured results

### 3.1 Capability accumulation works with real models

| Proposer | Tasks solved | One-time cost | Reuse correctness |
|---|---|---|---|
| claude-haiku-4.5 | 5/5 families | 6 calls, 11,322 tokens | 50/50, 0 LLM |
| granite3.3:8b (local) | 3/5 families | 5 calls, 9,050 tokens | 30/30, 0 LLM |
| qwen2.5:7b (local) | 3/5 families | 4 calls, 6,001 tokens | 30/30, 0 LLM |

Local models have complementary failures — a pool of granite + qwen + one
claude call covers 5/5 families. The frontier model is needed only for
the single task type no local model can compose.

### 3.2 Cross-model transfer is proven

A capability qualified by claude-haiku was executed by qwen2.5:7b — a model
that demonstrably cannot compose the task itself — with 50/50 correctness
and zero LLM calls. No traces, no reasoning transfer: only the
content-addressed graph crossed the model boundary.

### 3.3 Consensus verification works

When two or more independent proposers (from different architecture
families) behaviorally agreed on a task's output, every consensus member
was correct: **precision 1.0, zero false agreements** across all agreeing
pairs. This enables oracle-free admission: tasks without hand-written test
cases can be verified by cross-proposer agreement.

### 3.4 Production workloads repeat enough to pay

A real production signal stream (17.6M signals, 171 type families)
exhibits 103,051× family repetition. The economic necessary condition for
capability accumulation holds by five orders of magnitude.

### 3.5 The platform survived adversarial testing

24 adversarial probes against the live system: sandbox purity and
canonicalization soundness HELD. 12 governance vectors were initially
bypassed; 8 were closed with regression tests. 4 remain registered with
phased fixes. The ledger was subsequently anchored on a Postgres-backed
immutable ledger, closing the highest-severity finding.

---

## 4. Architecture

The platform comprises nine layers, connected by MCP tools (synchronous)
and Kafka events (asynchronous):

```
CONSUMPTION     Praxis gateway, MCP tools
ORCHESTRATION   arf (work management), SDAVL (selection), Guardian (safety)
ROUTING         Cascade (inference avoidance, compression)
QUALIFICATION   eval-cards (model assessment), agent-passport (identity)
EXECUTION       VERGE (verification, capability registry, sandboxed runtime)
GOVERNANCE      EAK (earned authority), GCL (decision gates), VEF (value)
AUDIT           ARE (immutable ledger, proof receipts)
BACKBONE        Kafka (event streaming, replay, decoupling)
INFRASTRUCTURE  llm-d-fleet (model placement), OpenShift (deployment)
```

Detailed architecture: [ARCHITECTURE.md](ARCHITECTURE.md)

---

## 5. Use cases

### Tier 1: Proven (evidence on disk)

1. **Alert enrichment & triage** (observability/ops) — 99.99% LLM avoidance
2. **Compliance rule automation** (financial services) — regulation as spec
3. **Security detection engineering** (SOC) — verified rule lifecycle

### Tier 2: Architecture-enabled

4. **Cross-agent skill sharing** — capabilities as shared infrastructure
5. **Model migration without rewriting** — model independence
6. **Regulatory audit trail** — cryptographic proof of AI decisions
7. **Knowledge capture** — expert knowledge as executable capabilities

### Tier 3: Emergent

8. A/B testing of AI approaches
9. Disaster recovery for AI automations
10. Progressive cost optimization
11. Cross-domain learning
12. Vendor evaluation

Detailed use cases: [USE_CASES.md](USE_CASES.md)

---

## 6. Competitive position

| Capability | This platform | LangChain/LlamaIndex | Custom in-house |
|---|---|---|---|
| Model qualification | ✅ eval-cards | ❌ | ❌ |
| Output verification | ✅ VERGE | ❌ | Partial |
| Inference avoidance | ✅ Cascade | ❌ | ❌ |
| Earned authority | ✅ EAK | ❌ | ❌ |
| Immutable audit | ✅ ARE | ❌ | ❌ |
| Real-time safety | ✅ Guardian | ❌ | ❌ |
| Agent identity | ✅ passport | ❌ | ❌ |
| Capability caching | ✅ VERGE | ❌ | ❌ |
| Cross-model transfer | ✅ VERGE | ❌ | ❌ |
| Domain onboarding | ✅ domain kits | ❌ | ❌ |
| Cost attribution | ✅ VEF | ❌ | Partial |
| MCP-native | ✅ | Partial | ❌ |

No existing framework or platform addresses all twelve capabilities.

---

## 7. Deployment model

- **Infrastructure**: Red Hat OpenShift (Kubernetes)
- **Event backbone**: AMQ Streams (Apache Kafka)
- **Database**: PostgreSQL (ARE, eval-cards)
- **Model serving**: llm-d-fleet (CPU and GPU placement)
- **Agent interface**: MCP (Model Context Protocol)
- **Human interface**: REST APIs + web dashboards
- **Security**: edge TLS, bearer tokens, SSO, NetworkPolicies

---

## 8. Roadmap

| Phase | Focus | Status |
|---|---|---|
| 1 | MCP unification (all components callable) | ✅ Complete |
| 2 | Kafka event backbone | ✅ Design complete |
| 3 | Oracle bridge (Cascade memory → VERGE verification) | ✅ Complete |
| 4 | First domain kit (observability/ops) | ✅ Complete |
| 5 | SaaS-ification (multi-tenant, self-service, metered) | In progress |

---

## 9. Conclusion

The governed AI agent platform turns AI from a per-call expense into a
durable asset. Models are qualified before they're trusted. Outputs are
verified before they execute. Authority is earned through evidence. Every
action is audited in an immutable ledger. And the repetitive 99% of AI
work — the alert enrichments, the compliance checks, the data
normalizations — executes for free from a cache of verified capabilities.

The platform is built, tested, adversarially hardened, and measured. The
components exist. The evidence is on disk. What remains is deployment and
adoption.
