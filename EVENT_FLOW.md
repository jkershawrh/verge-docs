# Event Flow — The Governed AI Agent Platform

How events move through the system, from signal arrival to audited outcome.

## The happy path (cache hit — 99%+ of traffic after bootstrap)

```
┌──────────┐     ┌─────────┐     ┌──────────┐     ┌─────────┐     ┌──────────┐
│ Collector │     │  Kafka  │     │ Cascade  │     │  VERGE  │     │   ARE    │
│ (K8s/Prom)│────▶│signals. │────▶│ classifies│────▶│registry │────▶│  ledger  │
│           │     │  raw    │     │ (no LLM) │     │  HIT!   │     │          │
└──────────┘     └─────────┘     └──────────┘     └────┬────┘     └────▲─────┘
                                                     │               │
                                                     ▼               │
                                              ┌─────────────┐        │
                                              │ Execute graph│        │
                                              │ (zero LLM,   │        │
                                              │  ~2ms)       │        │
                                              └──────┬──────┘        │
                                                     │               │
                                                     ▼               │
                                              ┌─────────────┐        │
                                              │ tasks.       │───────┘
                                              │ completed    │  audit event
                                              └─────────────┘

Total LLM calls: 0
Total time: ~5ms (Kafka → Cascade → VERGE → execute)
```

## The learning path (cache miss — novel task)

```
┌──────────┐     ┌─────────┐     ┌──────────┐     ┌─────────┐
│  Agent /  │     │  MCP    │     │  VERGE   │     │  LLM    │
│  arf     │────▶│ Gateway │────▶│ Registry │────▶│ (propose)│
│          │     │         │     │  MISS    │     │         │
└──────────┘     └─────────┘     └────┬────┘     └────┬────┘
                                      │               │
                                      │    ┌──────────┘
                                      │    │ structured trace
                                      ▼    ▼
                                 ┌─────────────┐
                                 │   VERGE     │
                                 │  Compile    │
                                 │  trace →    │
                                 │  graph      │
                                 └──────┬──────┘
                                        │
                                        ▼
                                 ┌─────────────┐
                                 │  VERGE      │
                                 │  Verify:    │
                                 │  - static   │
                                 │  - property │
                                 │  - differential│
                                 │  - adversarial│
                                 │  - consensus│
                                 └──────┬──────┘
                                        │
                          ┌─────────────┴─────────────┐
                          ▼                           ▼
                   ┌─────────────┐           ┌─────────────┐
                   │   PASS      │           │    FAIL     │
                   │ admit as    │           │ negative    │
                   │ candidate   │           │ evidence    │
                   │ → qualify → │           │ retained    │
                   │ authorize   │           │ → repair    │
                   └──────┬──────┘           │ → retry     │
                          │                  └─────────────┘
                          ▼
                   ┌─────────────┐
                   │ Execute for │
                   │ this task   │
                   │ (now cached │
                   │ forever)    │
                   └─────────────┘
```

## The bootstrap path (historical replay)

```
┌──────────────────┐     ┌─────────────┐     ┌──────────────┐
│ Historical data  │     │   Kafka     │     │   Cascade    │
│ (JSONL files,    │────▶│ signals.raw │────▶│ --from-      │
│ DB export, etc.) │     │             │     │ beginning    │
└──────────────────┘     └─────────────┘     └──────┬───────┘
                                                     │
                                                     ▼
                                             ┌──────────────┐
                                             │ Memory built │
                                             │ Rules formed │
                                             │ ORACLE READY │
                                             └──────────────┘
                                                     │
                                                     ▼
                                             ┌──────────────┐
                                             │    VERGE     │
                                             │ starts,      │
                                             │ queries      │
                                             │ Cascade      │
                                             │ memory for   │
                                             │ verification │
                                             └──────────────┘

Time: minutes (not weeks of live operation)
```

## Event lifecycle detail

### 1. Signal arrives

```
Topic: signals.raw
{
  "event_id": "uuid-...",
  "event_type": "signal.raw",
  "source": "collector.kubernetes",
  "timestamp": "2026-10-21T10:30:00Z",
  "tenant": "acme",
  "payload": {
    "signal_type": "event_unhealthy",
    "severity": "medium",
    "source": "kube-api",
    "content": {"pod": "api-7f8b9", "namespace": "prod"},
    "labels": {"domain": "kubernetes"}
  }
}
```

### 2. Cascade classifies

```
Consumer group: cascade
Output topic: signals.classified
{
  "event_type": "signal.classified",
  "source": "cascade",
  "payload": {
    "signal_id": "...",
    "classification": "needs_attention",
    "confidence": 0.85,
    "outcome": "classified",
    "agent": "PatternClassifier"
  }
}
```

### 3. Agent submits task (if enrichment needed)

```
MCP call: submit_task
Topic: tasks.submitted
{
  "event_type": "task.submitted",
  "source": "agent.mcp",
  "payload": {
    "task_type": "observability.enrich_pod_alert",
    "input": {...},
    "input_schema": {...},
    "output_schema": {...},
    "example_input": {...},
    "example_expected": {...}
  }
}
```

### 4. VERGE executes (cache hit)

```
Topic: tasks.completed
{
  "event_type": "task.completed",
  "source": "verge",
  "payload": {
    "capability_id": "sha256:...",
    "status": "ok",
    "output": {...},
    "duration_ms": 2.1,
    "cost": {"llm_calls": 0, "llm_tokens": 0, "cpu_ms": 2.1}
  }
}
```

### 5. Capability lifecycle events (cache miss → admission)

```
Topic: capabilities.lifecycle
{ "event_type": "capability.admitted", "source": "verge", ... }
{ "event_type": "capability.qualified", "source": "verge", ... }
{ "event_type": "capability.authorized", "source": "verge", ... }
```

### 6. Evidence produced

```
Topic: evidence.produced
{
  "event_type": "evidence.verification",
  "source": "verge",
  "payload": {
    "evidence_id": "sha256:...",
    "kind": "property",
    "verdict": "pass",
    "claim_scope": "Empirical agreement on 51 cases; not a proof"
  }
}
```

### 7. Audit trail (mirror to ARE)

```
Topic: audit.events
{
  "event_type": "audit.entry",
  "source": "are-adapter",
  "payload": {
    "entry_type": "verge.promotion",
    "chain_hash": "...",
    "prev_hash": "..."
  }
}
```

## Component subscription map

| Component | Subscribes to | Publishes to |
|---|---|---|
| Cascade | signals.raw | signals.classified |
| VERGE | signals.classified, tasks.submitted | tasks.completed, capabilities.lifecycle |
| eval-cards | tasks.completed, model.performance | model.performance, evidence.produced |
| ARE adapter | all topics | audit.events |
| Guardian | tasks.submitted, capabilities.lifecycle | (validation results inline) |
| EAK | authority.changed | authority.changed |
| VEF | tasks.completed, capabilities.lifecycle | (value reports on demand) |
| SDAVL | model.performance, signals.classified | (routing decisions inline) |

## Failure and recovery

| Failure | System behavior | Recovery |
|---|---|---|
| LLM gateway down | Cache hits still work; novel tasks queue | Retry on gateway recovery |
| Cascade down | VERGE defaults to "invoke LLM" (fail-open for routing) | Cascade restarts, catches up from Kafka offset |
| ARE down | Audit events buffer in Kafka | ARE restarts, consumes from last offset |
| VERGE down | Tasks buffer in Kafka | VERGE restarts, processes backlog |
| Kafka down | System halts (all communication stops) | Kafka cluster recovery; consumers resume from last committed offset |
| PostgreSQL (ARE) down | ARE stops writing; audit gap | Point-in-time recovery; chain verification detects gap |
