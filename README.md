# VERGE — Verified Execution & Recursive Graph Evolution

**A model-agnostic computational substrate that accumulates verified
executable capabilities.**

This repository contains the documentation, research report, experiment
results, and architecture for VERGE — a platform that compiles LLM
proposals into typed dataflow graphs, verifies them through multi-modal
testing, and serves them from a content-addressed registry at zero LLM
cost.

> The source code lives in a private repository. This public repo contains
> the sanitized documentation, research findings, and product roadmap.

## Key results

| Result | Number |
|---|---|
| Real-model cross-family transfer | claude→qwen: 50/50 correct, 0 LLM calls |
| Multi-proposer pooling | 5/5 coverage with ONE frontier call (1,841 tokens) |
| Consensus oracle precision | **1.0** — zero false agreements |
| Production repetition rate | 17.6M signals / 171 families (103,051×) |
| LLM-call avoidance (simulated) | 90 → 5 calls (94%) at equal correctness |

## Documentation

| Document | Purpose |
|---|---|
| [docs/RESEARCH_REPORT.md](docs/RESEARCH_REPORT.md) | Full research write-up |
| [docs/PRODUCT_ROADMAP.md](docs/PRODUCT_ROADMAP.md) | SaaS product roadmap (5 phases) |
| [docs/USE_CASES.md](docs/USE_CASES.md) | 12 use cases across 3 tiers |
| [docs/acceptance.md](docs/acceptance.md) | Testing methodology (TDD/EDD/CDD/BDD/CBT) |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Nine-layer architecture |
| [RESEARCH_HYPOTHESES.md](RESEARCH_HYPOTHESES.md) | Falsifiable hypotheses with results |
| [EXPERIMENTS.md](EXPERIMENTS.md) | Experiment definitions |
| [EVIDENCE_REGISTER.md](EVIDENCE_REGISTER.md) | Every claim bound to evidence |
| [DECISION_LOG.md](DECISION_LOG.md) | 16 decision records |

## Evidence reports

| Report | Round |
|---|---|
| [evidence/ITERATION_REPORT_ROUND1.md](evidence/ITERATION_REPORT_ROUND1.md) | MVP (Phases 0-3) |
| [evidence/ITERATION_REPORT_ROUND2.md](evidence/ITERATION_REPORT_ROUND2.md) | Governance + evolution (Phase 4) |
| [evidence/ITERATION_REPORT_ROUND3.md](evidence/ITERATION_REPORT_ROUND3.md) | Real models + hardening (Phase 5) |

## License

TBD
