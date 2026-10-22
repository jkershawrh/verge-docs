# Workflows — The Governed AI Agent Platform

User-visible workflows: how different personas interact with the system.

## Workflow 1: Agent submits a task (the core loop)

**Persona**: Any AI agent or application

**Trigger**: A task needs to be done (enrichment, routing, transformation)

```
Step 1: Agent calls the MCP gateway
  POST /mcp {"method": "tools/call", "params": {
    "name": "submit_task",
    "arguments": {
      "task_type": "observability.enrich_pod_alert",
      "input": {"signal_type": "event_unhealthy", "content": {...}},
      "input_schema": {...},
      "output_schema": {...},
      "example_input": {...},
      "example_expected": {...}
    }
  }}

Step 2: Gateway authenticates and routes
  - Bearer token verified
  - Task contract validated (CDD)
  - Input canonicalized and schema-checked

Step 3: System routes the task
  ├── Cache hit (99%+ after bootstrap):
  │   → VERGE registry lookup → execute graph → return result
  │   → Cost: 0 LLM calls, ~2ms
  │
  └── Cache miss (novel task):
      → LLM proposes solution (structured trace)
      → VERGE compiles to graph
      → Verification: static + property + differential + adversarial
      ├── Pass → admit → qualify → authorize → execute → return result
      └── Fail → negative evidence → bounded repair → retry or reject

Step 4: Result returned
  {
    "status": "ok",
    "output": {...},
    "lane": "registry" | "generative",
    "capability_id": "sha256:...",
    "llm_calls": 0 | 1,
    "created_capability": false | true
  }
```

**What the agent sees**: a correct result, fast. Whether it came from cache
or was just created is transparent.

---

## Workflow 2: Domain onboarding (provisioning a new vertical)

**Persona**: Platform engineer or domain expert

**Trigger**: A new domain (observability, compliance, security, etc.) needs
to be onboarded

```
Step 1: Write the domain kit
  cat > domain-kit.yaml <<EOF
  apiVersion: verge.dev/v1
  kind: DomainKit
  metadata:
    name: compliance-financial
  spec:
    collectors:
      - type: kafka-connect-http
        config: {url: "https://bank.example.com/transactions"}
    domain_pack:
      signal_types: [wire_transfer, account_change, login_attempt]
    prompt_template: |
      You are analyzing financial transactions for compliance...
    oracle:
      source: cascade-memory
      consensus_threshold: 0.7
    bootstrap:
      kafka_topic: signals.raw
      replay_from: beginning
  EOF

Step 2: Apply the kit
  MCP call: provision_domain {"kit": <kit YAML>}

Step 3: Load historical data (bootstrap)
  kafka-console-producer --topic signals.raw \
    --from-file historical-transactions.jsonl

Step 4: Wait for bootstrap (minutes)
  Cascade processes all historical data
  → Memory built
  → Rules promoted
  → Oracle ready

Step 5: System is operational
  Agents can submit tasks in this domain
  LLM calls happen only for genuinely novel patterns
  Everything else is cached and free
```

**Time to value**: minutes for bootstrap, not weeks of engineering.

---

## Workflow 3: Model qualification (eval-cards)

**Persona**: ML engineer or platform team

**Trigger**: A new model needs to be qualified for a task class

```
Step 1: Author the construct statement
  {
    "statement": "The model can correctly enrich Kubernetes 
                  event_unhealthy signals with pod context and severity",
    "metric": "classification accuracy on held-out signal set",
    "gaming_analysis": "A model could achieve high accuracy by always 
                        predicting 'needs_attention' for medium-severity 
                        signals without actually reading the content..."
  }

Step 2: Run the qualification (k trials per case, k ≥ 2)
  For each test case:
    → Submit the signal to the model
    → Record the output
    → Classify the outcome (completed/incomplete)
    → Grade the quality (GREEN/AMBER/RED)
  Repeat k times per case

Step 3: Compute metrics
  single_attempt_rate = successful trials / total trials
  pass_at_k = cases with ≥1 success / cases
  pass_all_k = cases where all k trials succeeded / cases
  Wilson 95% intervals on each

Step 4: Build the evaluation card
  {
    "construct": <construct statement>,
    "reliability": {pass_at_k, pass_all_k, Wilson intervals},
    "evidence_label": "LIVE" | "REHEARSAL" | "OFFLINE",
    "card_sha256": "<digest>"
  }

Step 5: Notarize
  MCP call: notarize_evaluation_card(card)
  → Card's exact bytes chained into hash chain
  → Proof receipt returned

Step 6: Consume
  Any agent can query: "Is model X qualified for task class Y?"
  MCP call: summarize_evaluation_card(card)
```

**What the consumer learns**: whether the model consistently produces
correct results, with what statistical confidence, and whether they can
prove the measurement wasn't edited.

---

## Workflow 4: Authority management (EAK)

**Persona**: Security officer or platform admin

**Trigger**: An agent needs authority to perform actions

```
Step 1: Agent requests authority
  "I need to execute enrichment tasks in the compliance domain"

Step 2: EAK evaluates the request
  → Check: does the agent have qualification evidence?
  → Check: has the agent demonstrated competence?
  → Check: are there any prior revocations?

Step 3: Authority granted (or denied)
  ├── Granted: agent receives scoped, time-bound authority
  │   → Recorded in ARE (immutable)
  │   → Agent can now perform the authorized actions
  │
  └── Denied: reason recorded
      → Agent can reapply after meeting qualification requirements

Step 4: Ongoing monitoring
  → Agent exercises authority
  → Outcomes recorded
  ├── Good outcomes: authority reinforced
  └── Bad outcomes (falsified actions): 
      → Immediate revocation
      → Authority permanently revoked
      → Recorded in ARE

Step 5: Requalification (if authority expired or was revoked)
  → Agent must demonstrate competence again
  → Fresh evidence required
  → New authority grant (if qualified)
```

**Key property**: authority is earned, not given. It can be revoked
instantly on evidence of misuse.

---

## Workflow 5: Audit and compliance review

**Persona**: Auditor, risk officer, or regulator

**Trigger**: "Prove that this AI-driven decision was correct"

```
Step 1: Query the audit trail
  MCP call: verify_receipt(receipt)
  → Receipt verified against ARE hash chain
  → Chain integrity confirmed

Step 2: Trace the evidence chain
  For a specific decision:
  ├── Proposal: what the LLM suggested (trace, model identity)
  ├── Verification: what evidence was checked (evidence records)
  ├── Authority: who authorized the capability (promotion records)
  ├── Execution: what actually ran (graph digest, execution record)
  └── Outcome: what was produced (output digest, timestamp)

Step 3: Cross-reference by content digest
  Every artifact in the chain is content-addressed:
  → proposal digest → verification evidence digest → 
    capability manifest digest → execution record digest →
    ARE entry hash

Step 4: Generate compliance report
  "Decision X was proposed by model Y, verified against Z evidence,
   authorized by policy P, executed as graph G, and recorded in ARE
   entry E at timestamp T. The chain from E to the genesis block is
   intact. No tampering detected."
```

**What the auditor gets**: cryptographic proof of what happened, not
"trust us."

---

## Workflow 6: Progressive cost optimization

**Persona**: CTO or cost-conscious engineering lead

**Trigger**: "AI costs are too high; can we reduce them?"

```
Phase 1: Deploy with expensive models
  → All novel tasks solved by frontier model
  → Cost: high per-call, but capabilities start accumulating

Phase 2: Cache hit rate rises
  → Repetitive task types hit the registry
  → LLM calls decline monotonically
  → Cost: declining per-task average

Phase 3: Pool cheap proposers
  → Add local models (granite, qwen) as proposers
  → They handle tasks they can solve (complementary coverage)
  → Frontier model invoked only for the hardest tasks
  → Cost: approaching zero for established task families

Phase 4: Measure the savings
  VEF call: score_value(outcome)
  → "Your agents executed 2.3M verified tasks.
     99.7% served from cache.
     LLM cost: $X (vs $Y without the platform).
     Savings: $Y - $X over N months."
```

**The cost curve**: starts high (paying for qualification), declines
monotonically (capabilities accumulate), approaches zero for established
domains.

---

## Workflow 7: Disaster recovery

**Persona**: SRE or platform operator

**Trigger**: "The cluster died. Rebuild everything."

```
Step 1: Provision new infrastructure
  → OpenShift cluster
  → Kafka (AMQ Streams)
  → PostgreSQL (for ARE)

Step 2: Deploy components
  → ARE (are-immutable-ledger)
  → Cascade (cascade-compression)
  → VERGE
  → eval-cards
  → MCP gateway

Step 3: Replay Kafka history
  → Historical signals replayed through Cascade
  → Memory rebuilt
  → Rules re-promoted
  → Oracle restored

Step 4: Verify integrity
  → ARE chain verification (VerifyChain)
  → All entries present and intact
  → Capability registry cross-checked against ARE

Step 5: Resume operations
  → Agents reconnect to MCP gateway
  → Cache hit rate restored (capabilities were rebuilt from Kafka replay)
  → Zero capability loss
```

**Recovery time**: proportional to Kafka retention period and replay speed.
With 30-day retention and batch replay: minutes.

---

## MCP tool reference

All workflows are driven through MCP tool calls:

| Tool | Component | Purpose |
|---|---|---|
| `submit_task` | VERGE | Execute a task (cached or new) |
| `list_capabilities` | VERGE | List registry contents |
| `get_capability` | VERGE | Inspect a capability's evidence |
| `verge_stats` | VERGE | Registry and cost statistics |
| `qualify_model` | eval-cards | Check model qualification |
| `select_model` | SDAVL | Select best model for task class |
| `classify_signal` | Cascade | Classify a signal |
| `recall_similar` | Cascade | Find similar signals in memory |
| `should_invoke_llm` | Cascade | Check if LLM invocation is needed |
| `check_authority` | EAK | Check agent authorization |
| `validate_action` | Guardian | Validate action safety |
| `verify_receipt` | ARE | Verify proof receipt |
| `chain_status` | ARE | Verify ledger chain integrity |
| `system_status` | Gateway | Health of all components |
| `provision_domain` | Domain kit | Onboard a new domain |
| `list_domains` | Domain kit | List provisioned domains |
