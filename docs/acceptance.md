# VERGE acceptance method

This document is normative for every VERGE component, experiment and report.
The rubric is the house standard used across this ecosystem (cascade-compression,
tca, arf!) — adopted verbatim, not reinvented.

## The five methods

- **TDD** — write and observe failing tests before implementing behavior. A
  behavior is implemented only after its test has been observed red.
- **EDD** — evaluation-driven development: freeze outcomes, baselines, cases and
  thresholds **before** evaluating. Evaluation datasets, success thresholds and
  baselines are committed and hashed before any candidate is measured against
  them.
- **CDD** — contract-driven development: validate every boundary through
  checked, versioned contracts. VERGE boundaries: task contract, capability
  manifest, graph IR, verification-evidence record, promotion record,
  cascade-adapter signal/decision mapping.
- **BDD** — given/when/then scenarios for complete workflows **and failure
  paths** (revocation, failed verification, invalid interfaces, drift).
- **CBT** — capability-based testing across the surface: every capability is
  exercised through its full journey (classify → discover → execute → verify →
  qualify → promote → reuse → revoke), not just its unit internals.

Ordering follows the Cascade house methodology: **CDD → TDD → EDD → BDD**,
with CBT closing the loop across the surface.

## Red / amber / green

- **GREEN** — tests, contracts, builds and independent checks all passing for
  the stated scope. A passing unit test alone never turns a capability green.
- **RED** — a measured condition failed.
- **AMBER** — evidence is missing (not measured, not run, or measured outside
  the declared scope).

Statuses are never averaged into a production claim. Every green is scoped to
its evidence column. A synthetic-workload green confers no authority over real
workloads. Verified capability never implies authorized action.

## Suite layout (mirrors tca)

| Method | Location | Runner |
|---|---|---|
| TDD | `tests/unit` (`-m unit`) | pytest |
| BDD | `tests/bdd/features` | behave |
| EDD | `tests/examples` (`-m edd`) | pytest |
| CDD | `tests/contracts` (`-m cdd`) | pytest |
| CBT | `tests/integration` (`-m cbt`) | pytest |

Run the five-part matrix:

```bash
uv run python rubric/score.py
```

The gate passes only when all five suites executed and passed. Missing suites
are AMBER and fail the gate.
