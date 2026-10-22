# VERGE Platform — Use Cases

**From proven evidence to emergent capability.** Each use case is grounded
in the platform's measured results (E1–E13) and architectural properties.

---

## Tier 1: Proven use cases (evidence on disk)

### 1. Alert Enrichment & Triage (Observability/Ops)

**Buyer**: SRE / Ops team lead

**Pain**: 17.6M signals per period across 171 types. 99% are repetitive —
same alert types, same enrichment patterns, same routing decisions. Currently
handled by brittle scripts or expensive LLM calls per signal.

**How it works**:
```
Signal (event_unhealthy, pod=api-7f8b9)
  → Cascade classifies: "needs_attention" (known pattern, no LLM)
  → VERGE: cached enrichment for event_unhealthy? YES
  → Execute: pod context + recommended action + severity (zero LLM, ~2ms)
  → Novel type → LLM proposes → CascadeMemoryOracle verifies against
    historical outcomes → admit → free forever
```

**ROI**: 99.99% LLM avoidance at 103,051× repetition (E11). Qualification:
~2k tokens per signal type, one-time. Execution: $0.

**Domain kit**: Built (`OBSERVABILITY_KIT`).

**Evidence**: E11 (repetition), E8-real (cross-model transfer), E9 (pooling).

---

### 2. Compliance Rule Automation (Financial Services)

**Buyer**: Compliance officer / RegTech lead

**Pain**: Every transaction, claim, or data access goes through the same
regulatory checks. Rules are complex, documented, testable — but hardcoded
into engines that cost millions to update when regulations change.

**How it works**:
```
Transaction (type=wire_transfer, amount=$50k, origin=high_risk)
  → Cascade routes to compliance check
  → VERGE: cached "AML_screening" capability? YES → execute
  → New regulation → LLM reads rule text → proposes check as graph
    → verify against adjudicated cases (known-compliant/non-compliant)
    → admit → every transaction screened for free
  → Regulation changes → revoke → re-verify → re-admit
```

**ROI**: Replace per-transaction engine licensing with one-time LLM
qualification per rule. Regulatory changes trigger re-verification, not
engineering sprints.

**Domain kit**: Adjudicated case sets + regulatory text corpus + domain ops
(entity-match, blacklist-lookup, threshold-check).

---

### 3. Security Detection Engineering (SOC)

**Buyer**: SOC lead / Detection engineering manager

**Pain**: Detection rules (Sigma, YARA) are hand-crafted by analysts.
Intel-to-production: days. Rules decay as threats evolve. Nobody tracks
which rules still work.

**How it works**:
```
Threat intel: "new ransomware uses XYZ technique"
  → LLM reads intel → proposes Sigma rule
  → VERGE verifies against:
    a. Red-team test dataset (catches known attacks?)
    b. Benign traffic corpus (avoids false positives?)
    c. Cross-proposer consensus (E12: precision 1.0)
  → Admit → deploy to SIEM → every event checked for free
  → Threat evolves → detection rate drops → revoke → re-verify
```

**ROI**: Intel-to-rule in minutes, not days. Rules have lifecycle governance
(admission, performance tracking, revocation) instead of fire-and-forget.

**Domain kit**: Red-team test sets + benign corpus + rule templates +
performance thresholds.

---

## Tier 2: Architecture-enabled use cases

### 4. Cross-Agent Skill Sharing

**Buyer**: Platform team managing multiple AI agents

**Pain**: Agent A (customer service) learns to extract order status. Agent B
(internal ops) needs the same capability. Different teams, duplicate work.

**How it works**:
```
Agent A: "extract_order_status" → propose → verify → admit to registry
Agent B: same task type → registry hit → execute (zero LLM, zero dev)
```

**ROI**: Agent capabilities become shared infrastructure. New agents inherit
the accumulated registry. "Bus factor" for automations drops from N to 1.

**Evidence**: E8-real (claude→qwen transfer, 50/50 correct, zero LLM).

---

### 5. Model Migration Without Rewriting

**Buyer**: CTO / Architecture board

**Pain**: "We built 200 automations on Claude. Claude's pricing changed.
Now we rewrite everything."

**How it works**:
```
200 capabilities in registry (compiled from Claude proposals)
  → Switch gateway from Claude to GLM
  → All 200 capabilities execute unchanged (they're graphs, not prompts)
  → New proposals come from GLM
  → Zero rewrites, zero downtime, zero migration cost
```

**ROI**: Model independence is architectural. The registry IS the abstraction
layer.

**Evidence**: E8 (capabilities outlive their creators), E9 (cross-proposer
equivalence 80/80).

---

### 6. Regulatory Audit Trail for AI Actions

**Buyer**: Risk / Audit / Legal

**Pain**: "An AI agent made this decision. Prove it was correct. Prove it
wasn't tampered with. Prove who authorized it."

**How it works**:
```
Any AI-driven action
  → VERGE: what was proposed, how verified, what evidence
  → EAK: who authorized, when, under what policy
  → ARE: immutable hash chain of every event
  → Audit: "show me the evidence chain for decision X"
  → Cross-referenced by content digest: proposal → verification → 
    authority → execution → ARE receipt
```

**ROI**: AI governance becomes provable. Every action has a cryptographic
evidence chain.

---

### 7. Knowledge Capture When Experts Leave

**Buyer**: Engineering manager / Knowledge management

**Pain**: Senior SRE leaves. They knew how to triage 50 alert types.
That knowledge is in their head, maybe in a runbook nobody reads.

**How it works**:
```
While SRE is active:
  → Handles alerts → LLM proposes → SRE provides examples
  → VERGE verifies → capabilities accumulate
  → Each capability IS the SRE's knowledge, executable

After SRE leaves:
  → 50 capability types still work
  → New team members inherit the registry
  → New alert types → new proposals → system keeps learning
```

**ROI**: Institutional knowledge becomes executable infrastructure.

---

## Tier 3: Emergent use cases

### 8. A/B Testing of AI Approaches
Submit the same task to multiple proposers. Compare verification outcomes.
Keep the best. E9's complementary-failure result is the proof-of-concept.

### 9. Disaster Recovery for AI Automations
The entire registry rebuilds from Kafka replay + ARE ledger.
"Our cluster died" → replay → registry reconstructed → automations resume.

### 10. Progressive Cost Optimization
Start with expensive models for everything. As capabilities accumulate,
cache hit rate rises. LLM costs decline monotonically. E9's pattern
(one frontier call + free local proposers) is the end state.

### 11. Cross-Domain Learning
A capability verified for K8s alerts might adapt to Prometheus alerts.
The registry's semantic search finds cross-domain candidates.
Verification against the new domain's oracle admits or rejects.

### 12. Vendor Evaluation
"Should we use Claude or GLM for this task class?" Submit identical tasks
to both. Compare admission rates, verification outcomes, token costs.
Eval-cards records the answer with Wilson intervals.

---

## The meta-value proposition

| Without the platform | With the platform |
|---|---|
| AI automations are bespoke, per-agent, unverified | Shared, verified, content-addressed |
| Switching models means rewriting everything | Switching models changes nothing |
| Audit is "trust us" | Audit is cryptographic proof |
| Knowledge lives in people's heads | Knowledge lives in the registry |
| Cost is per-call, forever | Cost is one-time per family, then free |
| Correctness is "the model seemed right" | Correctness is verified against evidence |
| New domains need new engineering | New domains need a domain kit (config) |

**The platform turns AI from a per-call expense into a durable asset.**
