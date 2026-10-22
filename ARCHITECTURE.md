# VERGE Architecture

Nine logical layers (responsibilities, not microservices). Module map reflects
what is actually built in this repository; every claim is scoped to the
EVIDENCE_REGISTER.md.

```
                     ┌─────────────────────────────────────────────┐
                     │ L1 Input & Intent      verge.task contract  │
                     └───────────────┬─────────────────────────────┘
                                     ▼
                     ┌─────────────────────────────────────────────┐
   task ────────────▶│ L2 Cascade adapter     verge.cascade        │
                     │ (domain consumer of Cascade pipeline        │
                     │  semantics; llm-d-sc lane later)            │
                     └───────────────┬─────────────────────────────┘
                          known pattern │ needs generation
                                     ▼
                     ┌─────────────────────────────────────────────┐
                     │ L3 Capability Registry  verge.registry      │
                     │ content-addressed, ledger-backed            │
                     └───────┬─────────────────────┬───────────────┘
                    hit      │                     │ miss
                             ▼                     ▼
             ┌───────────────────────┐   ┌─────────────────────────┐
             │ L8 Execution Runtime  │   │ L4 Reasoning: backends  │
             │ verge.runtime         │   │ verge.backends (LLM     │
             │ sandboxed interpreter │   │ adapters, simulated +   │
             └───────────┬───────────┘   │ real-API-gated)         │
                         │               └────────────┬────────────┘
                         │                            ▼ trace
                         │               ┌─────────────────────────┐
                         │               │ L5 Trace→Graph Compiler │
                         │               │ verge.compiler + ir     │
                         │               └────────────┬────────────┘
                         │                            ▼ graph
                         │               ┌─────────────────────────┐
                         │               │ L6 Verification         │
                         │               │ verge.verification      │
                         │               └────────────┬────────────┘
                         │                            ▼ evidence
                         │               ┌─────────────────────────┐
                         │               │ L7 Authority & Promotion│
                         │               │ verge.registry (policy) │
                         │               │ + hash-chain ledger     │
                         │               └────────────┬────────────┘
                         │                            ▼ authorized
                         ▼                            │
             ┌───────────────────────────────────────────────────┐
             │ L9 Computational Memory  verge.registry + ledger  │
             │ capabilities, graphs, evidence, executions,       │
             │ equivalence history (all content-addressed)       │
             └───────────────────────────────────────────────────┘
```

## Module map and ownership

| Layer | Module | Status | Owner |
|---|---|---|---|
| L1 | `contracts/schemas/task.json` + engine intake | built | principal |
| L2 | `src/verge/cascade/` — adapter: task→Signal, VERGE domain agent, documented compatible lane | MVP | principal |
| L3 | `src/verge/registry/` — content-addressed store, exact signature lookup, advisory semantic search | MVP | Agent F |
| L4 | `src/verge/backends/` — LLMBackend protocol; simulated deterministic backends; real-API adapters (budget-gated, not called by default) | MVP | principal |
| L5 | `src/verge/ir/`, `src/verge/compiler/` — graph IR, canonicalization, structural hashing, trace→graph | MVP | Agent C |
| L6 | `src/verge/verification/` — static/property/differential engines, evidence records, policy gates | MVP | Agent D |
| L7 | `src/verge/registry/` (promotion policy, legal transitions, evidence-required authorization) + `src/verge/ledger/` | MVP | Agent F |
| L8 | `src/verge/runtime/` — closed opset v1.2.0 (53 pure ops incl. logic.case, per-op config chains, composed filter predicates), sandboxed interpreter with budgets/timeouts | shipped | Agent E + v1.2 agent |
| L9 | `src/verge/registry/` + `src/verge/ledger/` + execution records | MVP | Agent F |
| — | `src/verge/engine/` — the vertical slice orchestrating L1→L9 | MVP | principal |
| — | `src/verge/economics/` — total-cost-per-verified-task ledger | MVP | principal |

## Structural rules

1. **Modules exchange contract-validated dicts**, not live objects, at every
   layer boundary (CDD). The JSON Schemas in `contracts/` are the only
   interface definitions.
2. **Dependency injection at the verification seam**: the verification engine
   receives `graph_validator` and `executor` callables; it never imports the
   IR or runtime directly. This keeps verification independently testable and
   independently replaceable.
3. **Ledger is the source of truth for authority**; capability manifests are
   rebuildable projections (shared-enterprise-memory pattern). Local MVP
   ledger = append-only hash-chained JSONL; production target =
   are-immutable-ledger (same pattern, Postgres-backed).
4. **Canonicalization before hashing**: all content addressing digests the
   canonical form (`verge.hashing`), so node naming/order never affects
   identity. Structural identity of graphs from different producers is the
   basis of cross-model agreement measurement (E1).
5. **Verification ≠ authorization**: the verification engine produces
   evidence; the registry's promotion policy decides authority. Authorization
   requires existing passing evidence, but evidence alone never authorizes.
6. **Simulated backends are labeled simulated** in every trace
   (`model.kind: "simulated"`). Cross-model experiments using them
   demonstrate mechanism, not model independence — stated in every
   experiment's claim scope.

## Key design decisions (details in DECISION_LOG.md)

- **D-001 Runtime: pure-Python interpreter, not WebAssembly, for MVP.** The
  VERGE spec requires evaluating WASM on measured suitability; with a closed
  52-op pure opset, a Python interpreter achieves the same sandbox guarantee
  (no eval/exec/import/io, enforced by test) with zero toolchain cost. WASM
  revisited when side-effecting ops or cross-language portability arrive.
- **D-002 Ledger: local hash-chain JSONL stand-in** for are-immutable-ledger
  (Postgres) — identical pattern (append-only, chained, verifiable), no
  infrastructure dependency for hermetic experiments.
- **D-003 Backends: simulated deterministic test doubles** for all
  experiments; real-API adapters exist behind an explicit spending-limit
  config and are never invoked by default.
- **D-004 Cascade integration: domain consumer.** VERGE registers its own
  CascadeAgent (task-pattern recognition) against Cascade's published
  protocol; Cascade's classification semantics are never modified.
- **D-005 MVP scope: pure data transformations only** — permissions contract
  const-locks side_effects to "none".

## Vertical slice (MVP acceptance path)

1. Typed task in (L1, contract-validated).
2. Cascade-lane classification (L2): deterministic pattern recognition →
   task_type + known-pattern verdict.
3. Registry exact-signature lookup (L3).
4. Authorized hit → sandboxed execution (L8) → execution record.
5. Miss → backend proposal (L4, trace) → compile to graph (L5) →
   verify: static + property + differential (L6) → admit as candidate
   (L3/L9) → qualify + authorize under policy (L7, ledger-recorded).
6. Subsequent same-signature task → step 4 (no LLM).
7. Every step's cost lands in the cost ledger (economics module).
