# Architecture — The Governed AI Agent Platform

## System overview

```
╔═══════════════════════════════════════════════════════════════════════════╗
║                    CONSUMPTION LAYER                                       ║
║                                                                           ║
║  Agents (any AI, any human tool) call ONE MCP endpoint                    ║
║  ┌─────────────────────────────────────────────────────────────────┐      ║
║  │ Unified MCP Gateway (VERGE gateway, port 8791)                  │      ║
║  │ 14 tools: submit_task, qualify_model, classify_signal,          │      ║
║  │ recall_similar, check_authority, validate_action, ...           │      ║
║  │ Bearer-token authn, fail-closed, graceful degradation           │      ║
║  └─────────────────────────────────────────────────────────────────┘      ║
║  ┌─────────────────────────────────────────────────────────────────┐      ║
║  │ Praxis (AI Gateway)                                             │      ║
║  │ One consistent interface for applications to call approved      │      ║
║  │ models. Platform team controls endpoints, credentials, routing. │      ║
║  └─────────────────────────────────────────────────────────────────┘      ║
╚═══════════════════════════════════════════════════════════════════════════╝
                            │ MCP / REST
                            ▼
╔═══════════════════════════════════════════════════════════════════════════╗
║                    ORCHESTRATION LAYER                                     ║
║                                                                           ║
║  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────────┐   ║
║  │ arf              │  │ SDAVL            │  │ Guardian (Rust)        │   ║
║  │ (command-center) │  │ Sense→Decide→    │  │ Safety coprocessor:   │   ║
║  │                  │  │ Act→Verify→      │  │ validates, reasons,   │   ║
║  │ Work items       │  │ Learn            │  │ logs in real-time     │   ║
║  │ Specialist       │  │                  │  │                       │   ║
║  │ routing (DAG)    │  │ Capability       │  │ capability_gate       │   ║
║  │ Execution        │  │ selection,       │  │ chain_of_custody      │   ║
║  │ governance       │  │ admission,       │  │ compliance            │   ║
║  │ Journey review   │  │ workload         │  │ audit                 │   ║
║  └──────────────────┘  │ matching         │  └────────────────────────┘   ║
║                        └──────────────────┘                               ║
╚═══════════════════════════════════════════════════════════════════════════╝
                            │
                            ▼
╔═══════════════════════════════════════════════════════════════════════════╗
║                    DECISION & ROUTING LAYER                                ║
║                                                                           ║
║  ┌─────────────────────────────────────────────────────────────────┐      ║
║  │ Cascade Compression + domain packs                              │      ║
║  │                                                                 │      ║
║  │ Nano tier (85-99%): deterministic agents, no LLM                │      ║
║  │ Micro tier (1-15%): small CPU models via semantic classifier    │      ║
║  │ Macro tier (<1%): larger models, generative fallback            │      ║
║  │                                                                 │      ║
║  │ Rule promotion: evidence-gated, zero-FN invariant, 72h TTL      │      ║
║  │ Shadow validation: 5% of suppressed signals re-checked          │      ║
║  │ Memory: survivors become institutional knowledge                │      ║
║  │ Federation: cross-instance memory sharing                       │      ║
║  └─────────────────────────────────────────────────────────────────┘      ║
╚═══════════════════════════════════════════════════════════════════════════╝
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
╔═══════════════════════╗ ╔══════════════════════════════════════════════╗
║  CAPABILITY CACHE     ║ ║  MODEL QUALIFICATION & IDENTITY              ║
║  (zero LLM)           ║ ║                                              ║
║  ┌─────────────────┐  ║ ║  ┌──────────────┐  ┌──────────────────────┐  ║
║  │ VERGE           │  ║ ║  │ eval-cards   │  │ agent-passport (Go)  │  ║
║  │                 │  ║ ║  │              │  │                      │  ║
║  │ Registry:       │  ║ ║  │ Qualification│  │ Signed, scoped,      │  ║
║  │  content-       │  ║ ║  │ cards with   │  │ time-bound,          │  ║
║  │  addressed      │  ║ ║  │ pass-all@k,  │  │ revocable            │  ║
║  │  capabilities   │  ║ ║  │ Wilson CIs,  │  │ credentials for      │  ║
║  │                 │  ║ ║  │ construct    │  │ AI agents            │  ║
║  │ Runtime:        │  ║ ║  │ statements, │  │                      │  ║
║  │  53-op opset    │  ║ ║  │ notarized    │  │ Behavioral           │  ║
║  │  sandboxed      │  ║ ║  │ into hash    │  │ authorization        │  ║
║  │  interpreter    │  ║ ║  │ chain        │  │ layer                │  ║
║  │                 │  ║ ║  └──────────────┘  └──────────────────────┘  ║
║  │ Verification:   │  ║ ║                                              ║
║  │  static +       │  ║ ╚══════════════════════════════════════════════╝
║  │  property +     │  ║
║  │  differential + │  ║
║  │  adversarial +  │  ║
║  │  consensus      │  ║
║  │                 │  ║
║  │ Promotion:      │  ║
║  │  candidate →    │  ║
║  │  qualified →    │  ║
║  │  authorized     │  ║
║  │                 │  ║
║  │ MCP server:     │  ║
║  │  4 tools        │  ║
║  └─────────────────┘  ║
╚═══════════════════════╝
                            │
                            ▼
╔═══════════════════════════════════════════════════════════════════════════╗
║                    GOVERNANCE LAYER                                        ║
║                                                                           ║
║  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────────┐   ║
║  │ EAK              │  │ GCL / gcl-oss    │  │ VEF                    │   ║
║  │ Earned authority │  │ Governed         │  │ Value-evidence         │   ║
║  │                  │  │ decisions,       │  │ framework: capability  │   ║
║  │ Qualify → grant  │  │ falsification    │  │ → outcome → value →    │   ║
║  │ → revoke →       │  │ gates, signed    │  │ confidence rating      │   ║
║  │ requalify        │  │ proposals        │  │ (refuses fake ROI)     │   ║
║  │                  │  │                  │  │                       │   ║
║  │ Mutation fences  │  │ Constraint       │  │ Counterfactual        │   ║
║  │ Independent      │  │ checking         │  │ attribution           │   ║
║  │ readback         │  │                  │  │                       │   ║
║  └──────────────────┘  └──────────────────┘  └────────────────────────┘   ║
╚═══════════════════════════════════════════════════════════════════════════╝
                            │
                            ▼
╔═══════════════════════════════════════════════════════════════════════════╗
║                    AUDIT LAYER                                             ║
║                                                                           ║
║  ┌─────────────────────────────────────────────────────────────────┐      ║
║  │ ARE (are-immutable-ledger)                                      │      ║
║  │                                                                 │      ║
║  │ SHA-256 hash-chained entries, per entry type                    │      ║
║  │ PostgreSQL-backed, role-verified no-UPDATE/DELETE               │      ║
║  │ V3 canonical entry hashes                                       │      ║
║  │ Portable proof receipts                                         │      ║
║  │ Cross-system event correlation                                 │      ║
║  │ gRPC + REST + MCP-accessible                                    │      ║
║  └─────────────────────────────────────────────────────────────────┘      ║
╚═══════════════════════════════════════════════════════════════════════════╝
                            │
                            ▼
╔═══════════════════════════════════════════════════════════════════════════╗
║                    EVENT BACKBONE                                          ║
║                                                                           ║
║  ┌─────────────────────────────────────────────────────────────────┐      ║
║  │ Kafka (AMQ Streams on OpenShift)                                │      ║
║  │                                                                 │      ║
║  │ Topics: signals.raw, signals.classified, tasks.submitted,       │      ║
║  │   tasks.completed, capabilities.lifecycle, evidence.produced,   │      ║
║  │   authority.changed, model.performance, audit.events            │      ║
║  │                                                                 │      ║
║  │ Capabilities: replay (rewind to any offset), partitioning,      │      ║
║  │   consumer groups (independent scaling), configurable retention │      ║
║  └─────────────────────────────────────────────────────────────────┘      ║
╚═══════════════════════════════════════════════════════════════════════════╝
                            │
                            ▼
╔═══════════════════════════════════════════════════════════════════════════╗
║                    INFRASTRUCTURE LAYER                                    ║
║                                                                           ║
║  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────────┐   ║
║  │ llm-d-fleet      │  │ OpenShift        │  │ Model Gateways         │   ║
║  │ Model placement, │  │ Kubernetes       │  │ (OpenAI-compatible,    │   ║
║  │ routing,         │  │ Container        │  │  Ollama local,         │   ║
║  │ metering,        │  │ orchestration    │  │  vLLM, managed APIs)  │   ║
║  │ chargeback       │  │                  │  │                       │   ║
║  └──────────────────┘  └──────────────────┘  └────────────────────────┘   ║
╚═══════════════════════════════════════════════════════════════════════════╝
```

## Component details

### VERGE (execution + verification)

| Aspect | Detail |
|---|---|
| Opset | 53 pure operations (v1.2) — string, number, list, map, boolean, logic.case, composed chains |
| Runtime | Sandboxed interpreter — no I/O, no eval, budgets, timeouts |
| Registry | Content-addressed (SHA-256), ledger-authoritative |
| Verification | Static + property + differential + adversarial + consensus |
| Promotion | candidate → qualified → authorized; advisory/permanent revocation |
| Transfer | Cross-model: one model's capability executed by any other |
| MCP | 4 tools (submit_task, list/get capabilities, stats) |
| Tests | 900+ across five suites (TDD, BDD, EDD, CDD, CBT) |

### Cascade (routing + compression)

| Aspect | Detail |
|---|---|
| Tiers | Nano (deterministic, 85-99%), Micro (CPU models, 1-15%), Macro (LLM, <1%) |
| Rules | Evidence-gated promotion with zero-FN invariant, 72h TTL |
| Memory | Content-hash deduped, strength-weighted, capacity-bounded |
| Recall | Composite similarity: type (0.4) + label Jaccard (0.2) + cosine (0.2) + trigram (0.2) |
| Oracle | Memory becomes VERGE's verification reference (CascadeMemoryOracle) |
| Federation | Cross-instance memory sharing via export/import |

### eval-cards (qualification)

| Aspect | Detail |
|---|---|
| Cards | Construct statement + reliability + evidence labels + body digest |
| Metrics | pass@k, pass-all@k, Wilson 95% intervals |
| Labels | LIVE / REHEARSAL / OFFLINE (never label replay as live) |
| Notarization | Hash-chain + portable proof receipts |
| MCP | 8 tools (validate, summarize, notarize, verify, list, chain_status) |

### ARE (audit)

| Aspect | Detail |
|---|---|
| Storage | PostgreSQL, role-verified no-UPDATE/DELETE |
| Hashing | SHA-256 per-entry-type chains, V3 canonical |
| Proof | Portable receipts (IssueReceipt, VerifyProof) |
| Interfaces | gRPC (:19292), REST gateway, MCP |
| Verification | Chain verification (VerifyChain), background verifier |
| Deployment | OpenShift, PostgreSQL-backed |

## Data flow summary

```
Agent → MCP Gateway → [qualification check] → [authority check] →
  [safety check] → [Cascade routing] →
    ├── known pattern → cached capability → execute (zero LLM)
    └── novel → LLM proposal → VERGE compile → verify → admit → execute
         → outcome → Kafka → ARE (audit) + eval-cards (feedback)
```

## Trust model

| Question | Answered by | Evidence type |
|---|---|---|
| Can this model do this task? | eval-cards | pass-all@k, Wilson CI |
| Is this specific output right? | VERGE | evidence + claim scope |
| Who decided this? | EAK + GCL | promotion record, policy |
| Was this tampered with? | ARE | chain hash, receipt |
| What actually ran? | VERGE + ARE | graph digest + execution |
| Is this action safe? | Guardian | real-time validation |
| Who is this agent? | agent-passport | signed credential |
| What was it worth? | VEF | capability→outcome→value |
