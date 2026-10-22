# Learning Architecture — Where the System Learns

The platform has SIX learning loops, each operating at a different level
of abstraction. Together they form a self-improving system that gets
better at routing, executing, qualifying, trusting, and prioritizing —
without any single component needing to understand the whole.

## Loop 1: Signal Learning (Cascade)

**What it learns**: Which signals are normal (suppress) vs. novel (escalate)

**How it learns**:
```
Signal arrives
  → Classified by existing rules
  → Outcome recorded (was the classification correct?)
  → Repeated correct classifications → rule strength increases
  → Shadow validation: 5% of suppressed signals re-checked by LLM
  → If shadow disagrees → rule demoted instantly (zero-FN invariant)
  → Novel patterns detected by corpus analyzer → new rule proposed
  → New rule starts at "draft" → must earn promotion through evidence
  → Memory: survivors become institutional knowledge (content-hash deduped)
  → Consolidation: old memories re-cascaded; noise decays, signal persists
```

**Where it's stored**: Cascade's in-process memory archive + JSON files.
Content-hash deduped, strength-weighted, capacity-bounded (10K default).

**What it enables**: 85-99% of signals handled without LLM invocation.

**Timescale**: Minutes to hours (rule promotion requires 50+ samples at
60% accuracy minimum; nano tier requires 200+ samples at 75% accuracy).

---

## Loop 2: Capability Learning (VERGE)

**What it learns**: How to do specific tasks (as executable graphs)

**How it learns**:
```
Novel task arrives (cache miss)
  → LLM proposes a solution (structured trace)
  → VERGE compiles trace → typed dataflow graph
  → Verification: static + property + differential + adversarial cases
  ├── PASS → admit to registry → available for free, permanent reuse
  └── FAIL → negative evidence retained → bounded repair → retry
             (the failure itself is data: what DOESN'T work)
  
Established capability executes
  → Outcome recorded (correct/incorrect, latency, cost)
  ├── Correct → capability stays, execution count grows
  └── Incorrect → drift detected → revocation → re-verification needed
  
Multiple models propose for the same task
  → Cross-proposer consensus (E12: precision 1.0, zero false agreements)
  → Alternative implementations retained (Pareto frontier)
  → Benchmark-guided selection serves the fastest correct implementation
```

**Where it's stored**: VERGE's content-addressed registry (filesystem,
SHA-256 digests). Capabilities are model-independent graphs — they outlive
and outtravel the models that created them.

**What it enables**: Zero-LLM execution of established task families.
Measured: 94% LLM-call avoidance (90→5 calls across 40 tasks).

**Timescale**: Per-task (one LLM call to create, then free forever).

---

## Loop 3: Model Learning (eval-cards)

**What it learns**: Which models are reliable for which task classes

**How it learns**:
```
Model qualification run
  → k trials per case (k ≥ 2 for reliability claims)
  → pass@k measured (can the model succeed at all?)
  → pass-all@k measured (can it succeed consistently?)
  → Wilson 95% intervals computed (honest uncertainty)
  → Construct statement authored (what's measured, how to game it)
  → Card notarized into hash chain (tamper-evident)
  
Ongoing qualification (from production usage)
  → VERGE admission rates per model → feed into eval-cards
  → Model A proposed 10 capabilities, 8 admitted, 2 failed
  → Model B proposed 10 capabilities, 3 admitted, 7 failed
  → eval-cards updates: "Model A: 80% admission rate on this class"
  → SDAVL uses this to route future proposals to Model A
```

**Where it's stored**: eval-cards (PostgreSQL-backed, hash-chained, with
portable proof receipts). Each card is immutable; updates create new cards.

**What it enables**: Evidence-based model selection. "Use Model A for
task class X because its eval-card shows 85% pass-all@k with Wilson
interval [0.72, 0.92]."

**Timescale**: Per-qualification-run (hours) + continuous from production
telemetry.

---

## Loop 4: Trust Learning (EAK)

**What it learns**: Which agents can be trusted with which authorities

**How it learns**:
```
Agent requests authority
  → EAK checks: qualification evidence? demonstrated competence? prior revocations?
  ├── Qualified → scoped, time-bound authority granted
  └── Not qualified → denied with reason
  
Agent exercises authority
  → Every action recorded (what, when, outcome)
  ├── Good outcomes → authority reinforced (strength increases)
  └── Bad outcomes (falsified actions) → 
      → Immediate revocation (zero-tolerance)
      → Permanent revocation possible (content blacklist)
      → Recorded in ARE (immutable)
  
Authority expires (TTL)
  → Agent must requalify
  → Fresh evidence required (stale evidence refused)
  → New grant if still qualified
```

**Where it's stored**: EAK's authority state (PostgreSQL) + ARE (immutable
audit of every grant, exercise, and revocation).

**What it enables**: Authority that is earned, not assumed. Agents build
trust through demonstrated competence, lose it instantly on evidence of
misuse.

**Timescale**: Per-action (real-time) + per-qualification-cycle (TTL-based).

---

## Loop 5: Cross-Component Learning (the feedback wiring)

**What it learns**: How the components should work together

**How it learns** (this is the loop that doesn't fully exist yet — it's
the Phase 5 wiring):
```
VERGE executes a capability 10,000 times
  → Execution outcomes published to Kafka (tasks.completed)
  → Cascade's consumer group sees the outcomes
    → Memory: "capability X works well for signal type Y" (reinforced)
    → Routing: future signals of type Y routed to capability X directly
  → eval-cards' consumer group sees the outcomes
    → Model qualification updated: "the model that proposed capability X
      has a 99.7% success rate on this task class"
  → SDAVL's consumer group sees the outcomes
    → Capability selection optimized: prefer X over Y for this workload
  → VEF's consumer group sees the outcomes
    → Value attribution: "capability X saved $Y in LLM costs this month"

Cascade misclassifies a signal
  → Wrong routing decision
  → VERGE executes wrong capability (or fails to find one)
  → Negative outcome published
  → Cascade's shadow validation catches the misclassification
  → Rule demoted → better routing next time
  → eval-cards notes: "Cascade's accuracy on signal type Z dropped"
```

**Where it's stored**: Kafka topics (the event backbone). Every component
consumes the topics it needs. Learning is distributed — no single component
needs to understand the whole system.

**What it enables**: The system gets better at routing, selection, and
cost optimization as a WHOLE, not just as individual components.

**Timescale**: Continuous (real-time event stream).

---

## Loop 6: Meta-Learning (the system learns what to learn)

**What it learns**: What to optimize next

**How it learns** (emergent, not fully built):
```
SDAVL analyzes task submission patterns
  → "Task family 'compliance_check' is submitted 500×/day
     but has no cached capability (always cache miss)"
  → Signal: this family is worth investing in (high volume, no cache)
  → Recommendation: "Qualify a model for compliance_check tasks"
  
VEF analyzes value attribution
  → "Capability 'alert_enrichment' saves $X per execution
     but was revoked last week due to drift"
  → Signal: this capability was high-value
  → Recommendation: "Re-verify and re-admit alert_enrichment"
  
Cascade detects novel signal types
  → "Signal type 'new_attack_pattern' appeared 12 times in the last hour"
  → Signal: this is emerging, not yet classified
  → Recommendation: "Route to LLM for proposal; verify against adversarial
     cases; admit if correct"

eval-cards detects model performance degradation
  → "Model A's pass-all@k dropped from 85% to 60% over 2 weeks"
  → Signal: the model is getting worse at this task class
  → Recommendation: "Re-qualify Model A, or route to Model B"
```

**Where it's stored**: Distributed across components. SDAVL holds
capability selection state; VEF holds value measurements; Cascade holds
signal pattern statistics; eval-cards holds model performance trends.

**What it enables**: The system becomes self-directing about optimization.
Instead of a human deciding "we should build a capability for X," the
system detects the need and either builds it automatically (Loop 2) or
recommends it to a human.

**Timescale**: Hours to days (pattern detection requires accumulation).

---

## The learning stack

```
┌─────────────────────────────────────────────────────────────┐
│  Loop 6: META-LEARNING                                      │
│  "What should we optimize next?"                            │
│  → SDAVL detects uncached high-volume families              │
│  → VEF identifies high-value capabilities                   │
│  → Cascade flags novel patterns                             │
│  → eval-cards detects model drift                           │
├─────────────────────────────────────────────────────────────┤
│  Loop 5: CROSS-COMPONENT LEARNING                           │
│  "How should the components work together?"                 │
│  → Execution outcomes → Cascade memory → better routing     │
│  → Admission rates → eval-cards → better model selection    │
│  → Value measurements → VEF → better prioritization         │
├─────────────────────────────────────────────────────────────┤
│  Loop 4: TRUST LEARNING (EAK)                               │
│  "Which agents can be trusted?"                             │
│  → Authority earned through evidence                        │
│  → Revoked instantly on misuse                              │
│  → Requalified through fresh evidence                       │
├─────────────────────────────────────────────────────────────┤
│  Loop 3: MODEL LEARNING (eval-cards)                        │
│  "Which models are reliable for what?"                      │
│  → Qualification runs → pass-all@k + Wilson intervals       │
│  → Production telemetry → ongoing updates                   │
├─────────────────────────────────────────────────────────────┤
│  Loop 2: CAPABILITY LEARNING (VERGE)                        │
│  "How to do specific tasks"                                 │
│  → LLM proposes → verify → admit → free forever             │
│  → Negative evidence retained → repair → retry              │
│  → Alternatives retained → benchmark-guided selection       │
├─────────────────────────────────────────────────────────────┤
│  Loop 1: SIGNAL LEARNING (Cascade)                          │
│  "Which signals are normal vs. novel?"                      │
│  → Rules promoted from evidence                             │
│  → Shadow validation catches errors                         │
│  → Memory: survivors become institutional knowledge         │
└─────────────────────────────────────────────────────────────┘
```

## What makes this architecture work

1. **No single component needs to understand the whole system.** Each
   learning loop is local to its component. Cross-component learning
   happens through Kafka events, not direct API calls.

2. **Learning is always evidence-gated.** Cascade promotes rules based on
   accuracy statistics. VERGE admits capabilities based on verification
   evidence. EAK grants authority based on demonstrated competence.
   Nothing is learned without proof.

3. **Failure is data.** Negative evidence (failed proposals, misclassified
   signals, revoked authorities) is retained and used to improve. The
   system doesn't just learn from success — it learns from failure.

4. **Learning is model-independent.** Capabilities learned by one model
   are used by any other. Model qualification tracks which models are
   good at what, but the capabilities themselves transcend models.

5. **The system bootstraps itself.** Historical data replay through Kafka
   populates Cascade's memory, which becomes the oracle for VERGE's
   verification. The system arrives pre-trained on the domain's own
   experience.

6. **Learning has a timescale hierarchy.** Signal learning (Loop 1) is
   minutes-to-hours. Capability learning (Loop 2) is per-task. Model
   learning (Loop 3) is per-qualification-run. Trust learning (Loop 4) is
   per-action. Cross-component learning (Loop 5) is continuous. Meta-
   learning (Loop 6) is hours-to-days. The loops operate at different
   speeds and don't interfere with each other.

## What's still missing

| Loop | Status | Gap |
|---|---|---|
| 1. Signal (Cascade) | ✅ Built, deployed | — |
| 2. Capability (VERGE) | ✅ Built, 900+ tests | — |
| 3. Model (eval-cards) | ✅ Built, deployed | Production telemetry wiring |
| 4. Trust (EAK) | ✅ Built, tested | Integration with MCP gateway |
| 5. Cross-component | 🔶 Partially built | Kafka consumer wiring for feedback loops |
| 6. Meta-learning | 🔶 Partially built | SDAVL analysis + recommendation engine |

The cross-component wiring (Loop 5) is the Phase 5 SaaS-ification work.
The meta-learning (Loop 6) emerges naturally once Loop 5 is complete.

## Implementation status (updated)

| Loop | Status | Module |
|---|---|---|
| 1. Signal (Cascade) | ✅ Built, deployed | `cascade-compression` |
| 2. Capability (VERGE) | ✅ Built, 927+ tests | `verge` |
| 3. Model (eval-cards) | ✅ Built, deployed | `eval-cards` |
| 4. Trust (EAK) | ✅ Built, tested | `earned-authority-kit` |
| 5. Cross-component | ✅ Wired (FeedbackCollector + EventPublisher) | `verge.feedback` + `verge.kafka_publisher` |
| 6. Meta-learning | 🔶 Emerges from Loop 5 | SDAVL + VEF |

Loop 5 is now fully wired: `FeedbackCollector.record_execution()` →
`drain()` → `EventPublisher.publish()` → Kafka topics. The feedback flows:
- `cascade_feedback(task_type)` → Cascade memory (Loop 1)
- `model_feedback(model)` → eval-cards qualification (Loop 3)
- `model_ranking()` → SDAVL capability selection (Loop 6)
- `value_feedback(tenant)` → VEF value measurement
