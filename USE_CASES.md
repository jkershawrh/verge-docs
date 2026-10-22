# Use Cases — The Governed AI Agent Platform

**From proven evidence to emergent capability.** Each use case is grounded
in measured results and architectural properties.

---

## Tier 1: Proven use cases (evidence on disk)

### 1. Alert Enrichment & Triage (Observability/Ops)

**Buyer**: SRE / Ops team lead

**Pain**: Millions of signals per period across hundreds of types. 99%+
are repetitive — same alert types, same enrichment patterns, same routing
decisions. Currently handled by brittle scripts or expensive per-signal
LLM calls.

**How the platform solves it**:
- Cascade classifies signals without LLM invocation (85-99% suppression)
- VERGE caches verified enrichment capabilities per signal type
- Novel signal types → LLM proposes → verified against Cascade's memory
  → admitted → free forever
- Measured: 17.6M signals, 171 types, 103,051× repetition, 99.999% avoidance

**ROI**: One-time qualification (~2k tokens per signal type). Ongoing: $0
per execution. Replace per-signal LLM costs with per-family one-time costs.

---

### 2. Compliance Rule Automation (Financial Services)

**Buyer**: Compliance officer / RegTech lead

**Pain**: Every transaction, claim, or data access goes through the same
regulatory checks. Rules are complex, documented, testable — but hardcoded
into engines that cost millions to update when regulations change.

**How the platform solves it**:
- LLM reads regulation text → proposes compliance check as a typed graph
- VERGE verifies against adjudicated cases (known-compliant/non-compliant)
- Admitted → every subsequent transaction screened for free
- Regulation changes → revocation → re-verification → re-admission
- ARE provides the immutable audit trail regulators require

**ROI**: Replace per-transaction compliance engine licensing with one-time
LLM qualification per rule. Regulatory changes trigger re-verification,
not engineering sprints.

---

### 3. Security Detection Engineering (SOC)

**Buyer**: SOC lead / Detection engineering manager

**Pain**: Detection rules (Sigma, YARA) are hand-crafted by analysts.
Intel-to-production: days. Rules decay as threats evolve. Nobody tracks
which rules still work.

**How the platform solves it**:
- LLM reads threat intelligence → proposes detection rule
- VERGE verifies against red-team test datasets AND benign traffic
  (false-positive rate)
- Cross-proposer consensus (proven: precision 1.0, zero false agreements)
- Admitted → deployed → every event checked for free
- Threat evolves → detection rate drops → automatic revocation → re-verify

**ROI**: Intel-to-rule in minutes, not days. Rules have lifecycle
governance (admission, performance tracking, revocation).

---

## Tier 2: Architecture-enabled use cases

### 4. Cross-Agent Skill Sharing

**Buyer**: Platform team managing multiple AI agents

**Pain**: Agent A (customer service) learns to extract order status. Agent
B (internal ops) needs the same capability. Different teams, duplicate work.

**How the platform solves it**:
- Agent A proposes → VERGE verifies → capability admitted to shared registry
- Agent B queries same task type → registry hit → execute (zero LLM)
- Agent B didn't need to know how Agent A solved it

**ROI**: Agent capabilities become shared infrastructure. "Bus factor" for
automations drops from N (per-agent) to 1 (the platform).

**Evidence**: Cross-model transfer proven (claude→qwen, 50/50, zero LLM).

---

### 5. Model Migration Without Rewriting

**Buyer**: CTO / Architecture board

**Pain**: "We built 200 automations on Claude. Claude's pricing changed.
Now we rewrite everything."

**How the platform solves it**:
- 200 capabilities in registry (compiled from Claude proposals)
- Switch model gateway from Claude to any other model
- All 200 capabilities execute unchanged (they're graphs, not prompts)
- New proposals come from the new model
- Zero rewrites, zero downtime, zero migration cost

**ROI**: Model independence is architectural. The registry IS the
abstraction layer between "what the automation does" and "which model
produced it."

---

### 6. Regulatory Audit Trail for AI Actions

**Buyer**: Risk / Audit / Legal

**Pain**: "An AI agent made this decision. Prove it was correct. Prove it
wasn't tampered with. Prove who authorized it."

**How the platform solves it**:
- Every AI action generates: proposal → verification evidence → authority
  grant → execution record → ARE ledger entry
- All cross-referenced by content digest
- ARE provides cryptographic proof (hash chain, portable receipts)
- Chain verification detects any tampering

**ROI**: AI governance becomes provable, not performative. Every automated
action has a verifiable evidence chain.

---

### 7. Knowledge Capture When Experts Leave

**Buyer**: Engineering manager / Knowledge management

**Pain**: Senior SRE leaves. They knew how to triage 50 alert types.
That knowledge is in their head, maybe in a runbook nobody reads.

**How the platform solves it**:
- While expert is active: handles alerts → LLM proposes → expert provides
  examples → VERGE verifies → capabilities accumulate
- Each capability IS the expert's knowledge, executable
- After expert leaves: capabilities still work; new team inherits the registry

**ROI**: Institutional knowledge becomes executable infrastructure instead
of walking out the door.

---

### 8. Real-Time Safety Enforcement

**Buyer**: Security officer / Risk team

**Pain**: AI agents are taking actions with unknown safety implications.
Nobody validates actions before execution.

**How the platform solves it**:
- Guardian (Rust safety coprocessor) validates every action in real-time
- Checks against safety policies, compliance rules, and chain-of-custody
  requirements
- Blocks unsafe actions before they execute
- Logs all decisions for audit

**ROI**: AI agents can act safely without human review of every action.

---

## Tier 3: Emergent use cases

### 9. A/B Testing of AI Approaches
Submit the same task to multiple proposers. Compare verification outcomes.
Keep the best. Measured: complementary failures across model architectures
(granite and qwen each solve 3/5 families, different families).

### 10. Disaster Recovery for AI Automations
The entire capability registry rebuilds from Kafka event replay + ARE
ledger. "Our cluster died" → replay → registry reconstructed → automations
resume with zero capability loss.

### 11. Progressive Cost Optimization
Start with expensive frontier models. As capabilities accumulate, cache hit
rate rises. LLM costs decline monotonically. End state: one frontier call
per genuinely novel family, everything else free.

### 12. Vendor Evaluation
"Should we use Claude or GLM for this task class?" Submit identical tasks
to both. Compare admission rates, verification outcomes, token costs.
Eval-cards records the answer with Wilson intervals and construct statements.

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
| Safety is "we hope it's fine" | Safety is enforced in real-time |
| Authority is "we gave them admin" | Authority is earned through evidence |
| New domains need new engineering | New domains need a domain kit (config) |

**The platform turns AI from a per-call expense into a durable asset.**
