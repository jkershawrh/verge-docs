# The Governed AI Agent Platform

**A complete infrastructure for deploying, governing, and auditing AI agents
that act — not just decide.**

This repository documents the full ecosystem: every component, how they
connect, what problems they solve, and how to think about deploying them
as a unified platform.

---

## What this is

A **governed AI agent platform** where:

- **Models are qualified** before they're trusted (eval-cards)
- **Decisions are compressed** when patterns repeat (Cascade)
- **Novel work is verified** before it executes (VERGE)
- **Authority is earned** through evidence, not granted by default (EAK)
- **Every action is audited** in an immutable ledger (ARE)
- **Safety is enforced** in real-time by a coprocessor (Guardian)
- **Identity is cryptographic** — scoped, time-bound, revocable (agent-passport)
- **Capabilities accumulate** as reusable, model-independent assets (VERGE registry)
- **The entire system communicates** via MCP tools and Kafka events
- **New domains onboard** through configuration, not code (domain kits)

## The problem

Enterprises are deploying AI agents that make decisions and take actions.
But the current tooling addresses only fragments of the problem:

| Problem | Current state | What's needed |
|---|---|---|
| "Can this model do this task?" | Ad-hoc benchmarks, vibes | Eval-cards: qualified with pass-all@k and Wilson intervals |
| "Is this output correct?" | Trust the model | VERGE: verified against evidence before execution |
| "Who is allowed to do what?" | Role-based access control | EAK: authority earned through evidence, revoked on failure |
| "What actually happened?" | Application logs | ARE: immutable hash-chained audit trail |
| "Is this action safe?" | Human review | Guardian: real-time safety coprocessor |
| "Can I reuse this work?" | No — every call is from scratch | VERGE: content-addressed capability registry |
| "What if I switch models?" | Rewrite everything | VERGE: capabilities are model-independent graphs |
| "How much did this cost?" | Token counts, maybe | VEF: capability → outcome → value → confidence |
| "What should this agent do next?" | Hardcoded workflows | SDAVL: evidence-based capability selection |

No existing platform addresses all of these together. This ecosystem does.

## The architecture (one paragraph)

An agent submits work through an MCP gateway (Praxis). The system checks
whether the agent is qualified (eval-cards), authorized (EAK +
agent-passport), and safe (Guardian). If the task matches a known pattern,
Cascade suppresses the LLM call and a cached capability from VERGE's
registry executes it at zero cost. If the task is novel, an LLM proposes a
solution which VERGE compiles, verifies, and — if correct — admits to the
registry for permanent free reuse. Every decision, verification, and
execution flows through Kafka to ARE's immutable ledger. The result: AI
work that is **qualified, verified, governed, audited, and increasingly
free**.

## Repository map

| Component | Repository | Language | What it does |
|---|---|---|---|
| **VERGE** | `verge` (private) | Python | Capability registry, verification, sandboxed execution |
| **Cascade** | `cascade-compression` | Python | Signal classification, inference avoidance, rule promotion |
| **eval-cards** | `eval-cards` | Python | Model qualification cards, notarization, proof receipts |
| **ARE** | `are-immutable-ledger` | Rust | Immutable hash-chain ledger, Postgres-backed |
| **EAK** | `earned-authority-kit` | Python | Earned authority lifecycle (qualify→grant→revoke) |
| **GCL** | `gcl-oss` | Python | Governed decision synthesis, falsification gates |
| **arf** | `arf-metadata-learning` | Python | Work orchestration, specialist routing (121 modules) |
| **Guardian** | `guardian` | Rust | Real-time safety coprocessor (28 modules) |
| **agent-passport** | `agent-passport` | Go | Signed, scoped, revocable agent credentials |
| **SDAVL** | `sdavl-gate4` | Python | Sense→Decide→Act→Verify→Learn capability selection |
| **Praxis** | `praxis-lab` | Mixed | AI gateway for application teams |
| **VEF** | `value-evidence-framework` | Python | ROI scorecards: capability→outcome→value→confidence |
| **llm-d-fleet** | `llm-d-fleet` | Go+Rust | Model placement, routing, metering, chargeback |
| **NovaScan** | `novascan` | — | Deployment planning and provisioning |

## Key results (measured, evidence-linked)

| Finding | Number | Source |
|---|---|---|
| LLM-call avoidance through capability caching | 94% (90→5 calls) | VERGE E5 |
| Cross-model capability transfer (zero LLM) | 50/50 correct, 0 calls | VERGE E8-real |
| Multi-proposer pooling (one frontier call) | 5/5 coverage, 1,841 tokens | VERGE E9 |
| Consensus oracle precision | **1.0** (zero false agreements) | VERGE E12 |
| Production signal repetition rate | 17.6M / 171 families (103,051×) | VERGE E11 |
| Signal compression (inference avoidance) | 85-99% suppressed without LLM | Cascade |
| Adversarial hardening | 24 probes, 8 vectors closed | VERGE security |
| Ledger integrity | Role-verified no-UPDATE/DELETE | ARE |

## Documentation

| Document | Audience |
|---|---|
| [WHITEPAPER.md](WHITEPAPER.md) | Leadership, external readers |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Engineers, architects |
| [EVENT_FLOW.md](EVENT_FLOW.md) | Engineers (how events move through the system) |
| [WORKFLOWS.md](WORKFLOWS.md) | Users, operators (how to use the system) |
| [USE_CASES.md](USE_CASES.md) | Buyers, product managers |
| [docs/RESEARCH_REPORT.md](docs/RESEARCH_REPORT.md) | Researchers (VERGE-specific) |
| [docs/PRODUCT_ROADMAP.md](docs/PRODUCT_ROADMAP.md) | Product team |
