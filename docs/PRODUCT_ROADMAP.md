# VERGE Platform — Product Roadmap

**Goal**: A SaaS-deployable governed AI agent platform where agents call one
MCP endpoint, collectors push data via Kafka, and the system handles routing,
verification, execution, governance, and audit — invisibly.

**Architecture principle**: MCP calls in, governed execution out. Agents don't
know how the system works. Domain kits are configuration, not code. The oracle
emerges from Cascade's memory, bootstrapped by Kafka replay.

---

## Phase 1: MCP Unification (2 weeks)
**Goal**: Every component callable through ONE MCP endpoint.

### 1.1 Unified MCP Gateway
- Single FastAPI service exposing all tools via JSON-RPC
- Proxies to individual components (VERGE, eval-cards, Cascade, ARE, etc.)
- Bearer-token authn with per-principal authorization (agent-passport)
- Rate limiting and usage metering

### 1.2 MCP-wrap remaining components
| Component | Tools to expose | Effort |
|---|---|---|
| Cascade | `classify_signal`, `should_invoke_llm`, `recall_similar`, `get_memory_stats` | 1 day |
| EAK | `check_authority`, `get_agent_tier`, `grant_authority` (admin) | 1 day |
| Guardian | `validate_action`, `get_compliance_status` | 1 day |
| agent-passport | `verify_agent`, `issue_credential` (admin) | 1 day |
| SDAVL | `select_capability`, `match_workload` | 1 day |
| VEF | `score_value`, `get_roi` | 1 day |
| ARE | `verify_receipt`, `get_chain_status` | 1 day |

### 1.3 Tool catalog
```
Agent calls ONE endpoint, gets ALL tools:
  ├── qualify_model(model, task_class)        → eval-cards
  ├── select_model(task_class, constraints)   → SDAVL
  ├── classify_signal(signal)                 → Cascade
  ├── should_invoke_llm(signal)               → Cascade
  ├── recall_similar(signal, limit)           → Cascade memory
  ├── submit_task(task)                       → VERGE
  ├── list_capabilities()                     → VERGE
  ├── get_capability(id)                      → VERGE
  ├── check_authority(agent, action)          → EAK + passport
  ├── validate_action(action)                 → Guardian
  ├── verify_receipt(receipt)                 → ARE
  ├── score_value(outcome)                    → VEF
  ├── system_status()                         → all components
  └── replay_history(topic, from, to)         → Kafka
```

**Deliverable**: One MCP URL, one token, all tools.

---

## Phase 2: Event Backbone (3 weeks, overlaps Phase 1)
**Goal**: Kafka as the nervous system connecting all components.

### 2.1 Deploy AMQ Streams (Kafka) on OpenShift
- Strimzi operator / AMQ Streams
- Topics with appropriate retention and partitioning
- TLS + SCRAM-SHA-512 authn

### 2.2 Topics
```
signals.raw              ← collectors (K8s, Prom, Jira, Git, custom)
signals.classified       ← Cascade output
tasks.submitted          ← arf / agent requests
tasks.completed          ← VERGE execution results
capabilities.admitted    ← VERGE promotion events
capabilities.revoked     ← VERGE revocation events
evidence.produced        ← eval-cards / VERGE verification evidence
authority.changed        ← EAK grant/revoke events
model.performance        ← eval-cards qualification updates
audit.all                ← ARE event mirror
```

### 2.3 Collector adapters (Kafka Connect)
| Source | Connector | Status |
|---|---|---|
| Kubernetes events | Built into K8s / custom | Cascade has it |
| Prometheus alerts | Alertmanager webhook → Kafka | Build |
| Jira | Kafka Connect JDBC/source | Build |
| GitHub | Kafka Connect source | Build |
| Confluence | Kafka Connect source | Build |
| Custom REST | Kafka Connect HTTP source | Config |

### 2.4 Historical replay
- Load an internal cluster's 17.6M historical signals into `signals.raw`
- Cascade consumes from beginning → builds memory → oracle ready
- New VERGE instance bootstraps by replaying the same topic

**Deliverable**: Kafka cluster with topics, collectors publishing, historical data loaded.

---

## Phase 3: Oracle Bridge (1 week, after Phase 2)
**Goal**: Cascade's memory becomes VERGE's verification oracle.

### 3.1 CascadeMemoryOracle adapter
```python
class CascadeMemoryOracle:
    """Uses Cascade's recall engine as a verification oracle."""
    
    def expected_output(self, signal):
        similar = cascade.recall(signal, limit=10)
        return consensus([m.outcome for m in similar])
    
    def verify(self, graph, signal):
        actual = verge.execute(graph, signal)
        expected = self.expected_output(signal)
        return actual == expected
```

### 3.2 Feedback loop wiring
- VERGE publishes execution outcomes to `tasks.completed`
- Cascade's consumer group sees outcomes → memory reinforced/weakened
- eval-cards' consumer group sees outcomes → qualification updated
- The oracle improves with use

### 3.3 Cold-start protocol
1. Load historical data into Kafka
2. Start Cascade with `--from-beginning`
3. Wait for memory to build (minutes, not days)
4. Start VERGE → queries Cascade memory for verification
5. System is operational

**Deliverable**: New capabilities verified against Cascade's accumulated experience.

---

## Phase 4: First Domain Kit — Observability/Ops (2 weeks, after Phase 3)
**Goal**: End-to-end product for one domain.

### 4.1 Domain kit definition
```yaml
apiVersion: verge.dev/v1
kind: DomainKit
metadata:
  name: observability-ops
spec:
  collectors:
    - type: kubernetes-events
    - type: prometheus-alerts
  domain_pack: cascade-k8s (existing)
  prompt_template: |
    You are enriching observability signals for a Kubernetes cluster.
    Given the signal type and content, produce a structured enrichment...
  oracle:
    source: cascade-memory
    recall_limit: 10
    consensus_threshold: 0.7
  bootstrap:
    kafka_topic: signals.raw
    replay_from: beginning
```

### 4.2 Example workflows
- Alert enrichment: signal → Cascade routes → VERGE executes cached enrichment → zero LLM
- Incident summarization: signal cluster → Cascade groups → LLM proposes summary → verify → cache
- Threshold tuning: metric trend → Cascade detects → LLM proposes new threshold → verify against historical outcomes → cache

### 4.3 Metrics
- Cache hit rate (target: >90% after bootstrap)
- LLM calls avoided (target: >99% of signal volume)
- Verification precision (target: >95%)
- Time-to-capability (target: <5 min from first novel signal)

**Deliverable**: Working product for observability domain, deployable to a customer.

---

## Phase 5: SaaS-ification (4 weeks, after Phase 4)
**Goal**: Multi-tenant, self-service, metered.

### 5.1 Multi-tenancy
- Tenant isolation at the Kafka topic level (`signals.raw.{tenant}`)
- VERGE registry partitioned by tenant
- Per-tenant Cascade memory (separate consumer groups or topic filtering)
- Per-tenant authority domains (EAK scoping)

### 5.2 Self-service domain kit onboarding
```yaml
# Customer submits:
apiVersion: verge.dev/v1
kind: DomainKit
metadata:
  name: acme-corp-fraud-detection
  tenant: acme
spec:
  collectors:
    - type: kafka-connect-http
      url: https://acme.com/fraud-events
  domain_pack:
    signal_types: [transaction, account_change, login_attempt]
  prompt_template: |
    You are analyzing financial transactions for fraud indicators...
  oracle:
    source: cascade-memory
    bootstrap_data: s3://acme/historical-transactions.jsonl
```

Platform provisions: Kafka topics, Cascade consumer group, VERGE namespace,
EAK authority domain, eval-cards project. All from one YAML.

### 5.3 Metering and chargeback
- Per-tenant: LLM calls, tokens, capability executions, verification runs
- Integration with llm-d-fleet's existing UsageRecord/CostLineItem
- VEF scorecard: capability → outcome → value → confidence
- Monthly invoice: "your agents executed 2.3M verified tasks; 99.7% from cache; 
  cost: $X (vs $Y without the platform)"

### 5.4 Customer dashboard
- Cache hit rate over time
- Tokens saved
- Capability catalog with health status
- Evidence chain for any execution (click through to ARE proof receipt)
- Domain kit management (add collectors, update prompts, view oracle health)

**Deliverable**: Deployable SaaS product.

---

## Timeline

```
Week 1-2:   Phase 1 (MCP Unification)
Week 2-5:   Phase 2 (Kafka backbone)     ← overlaps Phase 1
Week 5-6:   Phase 3 (Oracle Bridge)      ← after Phase 2
Week 6-8:   Phase 4 (First Domain Kit)   ← after Phase 3
Week 8-12:  Phase 5 (SaaS-ification)     ← after Phase 4

Total: ~12 weeks to SaaS
```

## What already exists (no work needed)

| Component | Status | Location |
|---|---|---|
| VERGE (execution + verification) | 880 tests, MCP server | `verge` |
| eval-cards (qualification) | Deployed on an internal cluster | `eval-cards` |
| Cascade (routing) | Deployed, domain packs | `cascade-compression` |
| ARE (audit) | Deployed on an internal cluster | `are-immutable-ledger` |
| EAK (authority) | Built, tested | `earned-authority-kit` |
| GCL (governance) | Built, tested | `gcl-oss` |
| agent-passport (identity) | Built (Go) | `agent-passport` |
| Guardian (safety) | Built (Rust, 28 modules) | `guardian` |
| SDAVL (selection) | Built (Python) | `sdavl-gate4` |
| Praxis (gateway) | Reference implementation | `praxis-lab` |
| VEF (value) | Framework | `value-evidence-framework` |
| NovaScan (deployment) | Built | `novascan` |
| arf (orchestration) | 121 modules | `arf-metadata-learning` |

## What needs to be built

| Item | Effort | Phase |
|---|---|---|
| Unified MCP gateway | 1 week | 1 |
| MCP wrappers for 7 components | 1 week | 1 |
| Kafka deployment (AMQ Streams) | 1 week | 2 |
| Topic design + collector adapters | 1 week | 2 |
| Historical data loading | 2 days | 2 |
| CascadeMemoryOracle adapter | 3 days | 3 |
| Feedback loop wiring | 2 days | 3 |
| Domain kit YAML schema + provisioning | 1 week | 4 |
| Observability workflow examples | 1 week | 4 |
| Multi-tenant partitioning | 2 weeks | 5 |
| Self-service onboarding | 1 week | 5 |
| Metering + chargeback | 1 week | 5 |

## Risk register

| Risk | Mitigation |
|---|---|
| Kafka operational complexity | AMQ Streams (managed Kafka on OpenShift) |
| Multi-tenant isolation is hard | Start single-tenant, add isolation later |
| Domain kit authorship bottleneck | Cascade-as-oracle eliminates manual test cases |
| Circular oracle (Cascade wrong → oracle wrong) | Shadow validation + consensus oracle + human review for high-stakes |
| Customer data sovereignty | Per-tenant Kafka clusters, on-prem deployment option |
| LLM cost variability | Pool cheap proposers + one frontier call (E9 pattern) |
