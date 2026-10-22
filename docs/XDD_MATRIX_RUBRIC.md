# The XDD Method Matrix Rubric

**The definitive testing standard for the governed AI agent platform.**

This document is normative. Every component, capability, experiment, and
claim on this platform is tested according to this rubric. A result that
doesn't follow this rubric is not evidence. A claim without evidence is
false by definition.

---

## The five methods

| Method | Full name | What it tests | When it runs |
|---|---|---|---|
| **TDD** | Test-Driven Development | Individual units of behavior | Before implementation |
| **EDD** | Evaluation-Driven Development | Outcomes against frozen baselines | Before candidate measurement |
| **CDD** | Contract-Driven Development | Boundaries between components | At every interface |
| **BDD** | Behavior-Driven Development | Complete workflows and failure paths | After integration |
| **CBT** | Capability-Based Testing | End-to-end capability journeys | At the surface |

**Ordering**: CDD → TDD → EDD → BDD, closed by CBT.

```
CDD  contracts frozen
 ↓
TDD  units written red, implemented green
 ↓
EDD  outcomes frozen, candidates measured against baselines
 ↓
BDD  scenarios exercised (happy path + failure paths)
 ↓
CBT  capabilities tested across the full surface
 ↓ (feedback loop)
CDD  contracts revised based on discovered gaps
```

---

## Red / Amber / Green definitions

### Universal rules

1. **GREEN** requires ALL of: tests executed AND passed AND evidence on
   disk AND independently verifiable. A passing unit test alone NEVER
   turns a capability green.
2. **RED** means a measured condition failed. Not "we think it would
   fail" — it failed and the failure is on disk.
3. **AMBER** means evidence is missing. Not yet run, not yet measured,
   or measured outside the declared scope.
4. **Statuses are NEVER averaged.** A capability with 99 green tests and
   1 red test is RED. A suite with 100 passing tests and 1 missing suite
   is AMBER.
5. **Every GREEN is scoped** to its evidence column. "Green on synthetic
   data" is not "green in production."
6. **GREEN never implies authorization.** A verified capability is not
   an authorized capability. Authorization is a separate governance step.

---

## TDD — Test-Driven Development

### What it means

Write a test. Observe it fail. Implement the behavior. Observe it pass.
The test IS the specification; the implementation is verified against it.

### Red / Amber / Green

| Status | Definition | Evidence required |
|---|---|---|
| **RED** | Test written and observed failing for the CORRECT reason (the behavior doesn't exist yet) | Test output showing the failure |
| **AMBER** | Test written but NOT observed failing before implementation, OR observed failing for the wrong reason (e.g., import error instead of assertion failure), OR test not written | None (evidence missing) |
| **GREEN** | Test written, observed failing (RED), then observed passing (GREEN) after implementation, AND the test tests meaningful behavior | Both the red AND green outputs, plus the diff |

### Scoring

| Score | Criteria |
|---|---|
| 0 | No tests, or tests that can't run |
| 1 | Tests exist but were never observed red, OR all tests fail |
| 2 | 5-9 tests, all passing, red-first observed |
| 3 | 10+ tests, all passing, red-first observed, covering edge cases |

### What TDD does NOT prove

- That the tested behavior is correct in production (only that it matches
  the test's expectations)
- That the tested behavior integrates correctly with other components
- That the tested behavior is safe, governed, or authorized
- That the test itself is meaningful (a test that always passes is worse
  than no test)

### Anti-patterns (automatically AMBER)

- Test written AFTER implementation (not test-driven)
- Test modified to make it pass (test weakened)
- Test skipped or marked xfail without documented reason
- Assertion removed or weakened to achieve green
- Mock objects that test the mock, not the behavior

---

## EDD — Evaluation-Driven Development

### What it means

Freeze the outcomes, baselines, cases, and thresholds BEFORE evaluating
any candidate. Then measure the candidate against the frozen criteria.
The dataset and thresholds are committed and hashed before any run.

### Red / Amber / Green

| Status | Definition | Evidence required |
|---|---|---|
| **RED** | Evaluation run on frozen dataset; candidate did NOT meet the pre-declared threshold | Result manifest showing the measurement AND the frozen threshold |
| **AMBER** | Dataset not frozen before the run, OR thresholds not declared before the run, OR baseline not established, OR evaluation not run | None (evidence missing) |
| **GREEN** | Evaluation run on frozen dataset; candidate met or exceeded the pre-declared threshold; baseline was beaten; dataset digest matches the frozen manifest | Frozen manifest + result manifest + digest match |

### Scoring

| Score | Criteria |
|---|---|
| 0 | No evaluation, or evaluation on non-frozen data |
| 1 | Evaluation attempted but thresholds not pre-declared |
| 2 | Evaluation run on frozen data with declared thresholds |
| 3 | Evaluation run on frozen data, thresholds met, baseline beaten, held-out set also evaluated |

### What EDD does NOT prove

- That the frozen dataset is representative of production (it's frozen,
  not necessarily representative)
- That the thresholds were well-chosen (they were pre-declared, not
  necessarily optimal)
- That the baseline was fair (it was frozen, not necessarily strong)
- That the result generalizes beyond the dataset

### Anti-patterns (automatically AMBER)

- Dataset modified after evaluation started
- Thresholds adjusted after seeing results
- Baseline cherry-picked to make the candidate look good
- Held-out set used during development
- Evaluation-set contamination (test data leaked into training)

---

## CDD — Contract-Driven Development

### What it means

Every boundary between components is defined by a versioned contract
(JSON Schema, OpenAPI, proto). Both sides validate against the contract.
Invalid data is rejected at the door, never propagated.

### Red / Amber / Green

| Status | Definition | Evidence required |
|---|---|---|
| **RED** | Contract violation detected: valid data rejected (false negative) OR invalid data accepted (false positive) | Contract violation report showing the specific input and the schema rule violated |
| **AMBER** | Contract exists but no validation tests, OR golden examples missing, OR contract not versioned | None (evidence missing) |
| **GREEN** | All contracts validated: every golden valid example passes, every golden invalid example is rejected, contract version is pinned | Test output showing all golden examples pass/fail correctly |

### Scoring

| Score | Criteria |
|---|---|
| 0 | No contracts, or contracts exist but are never validated |
| 1 | Contracts exist, some validated, golden examples incomplete |
| 2 | All contracts validated, golden examples complete, versioned |
| 3 | All contracts validated + round-trip tests (serialize → deserialize → validate) + backward compatibility verified |

### What CDD does NOT prove

- That the contract is semantically correct (only that data conforms to it)
- That all possible invalid inputs are rejected (only that the golden
  invalid examples are)
- That the contract won't need to change (it will; that's why it's versioned)
- That two components that both validate against the same contract are
  actually compatible (they agree on the contract, not on semantics)

### Anti-patterns (automatically AMBER)

- Contract modified to make invalid data pass
- Golden examples updated to match buggy behavior
- Contract version bumped without a migration note
- additionalProperties: true (lazy schema) without documented reason

---

## BDD — Behavior-Driven Development

### What it means

Given/When/Then scenarios for complete user-visible workflows AND their
failure paths. The scenario describes what a user (human or agent)
experiences, not how the system works internally.

### Red / Amber / Green

| Status | Definition | Evidence required |
|---|---|---|
| **RED** | A scenario fails: the given/when/then does not produce the expected outcome | Scenario execution output showing which step failed and why |
| **AMBER** | Happy-path scenarios exist but failure-path scenarios are missing, OR scenarios exist but are not automated, OR failure paths exist but are not tested | None (evidence missing) |
| **GREEN** | All scenarios pass: happy paths AND failure paths (revocation, rejection, timeout, error, drift, tampering) | Full scenario execution output showing all steps |

### Scoring

| Score | Criteria |
|---|---|
| 0 | No scenarios, or scenarios exist but are manual |
| 1 | Happy-path scenarios automated, failure paths missing |
| 2 | Happy path + at least 2 failure paths automated |
| 3 | Happy path + comprehensive failure paths (rejection, revocation, timeout, drift, tampering, concurrent access) |

### Mandatory failure paths

Every BDD suite MUST include scenarios for:

| Failure path | What it tests |
|---|---|
| Invalid input rejected | The system refuses bad data at the door |
| Verification failure | Wrong proposals are rejected, negative evidence retained |
| Revocation enforcement | Revoked capabilities cannot execute |
| Authorization refusal | Unauthorized actions are blocked |
| Timeout / resource exhaustion | The system degrades gracefully |
| Tampering detection | Modified data is detected and refused |
| Concurrent access | Multiple simultaneous operations don't corrupt state |

Missing any of these → the suite is AMBER regardless of what passes.

### What BDD does NOT prove

- That the scenarios cover all possible user behaviors (they cover the
  described ones)
- That the internal implementation is correct (only that the externally
  visible behavior is)
- That performance is adequate (correctness, not speed)
- That security is complete (the tested attack vectors, not all of them)

---

## CBT — Capability-Based Testing

### What it means

Every capability is exercised through its FULL journey across the
platform surface: classify → discover → execute → verify → qualify →
promote → reuse → revoke. Not just the unit internals — the complete
lifecycle.

### Red / Amber / Green

| Status | Definition | Evidence required |
|---|---|---|
| **RED** | Any step in the capability journey fails: can't classify, can't discover, executes wrongly, fails verification, can't be promoted, can't be reused, or can't be revoked | Execution trace showing which journey step failed |
| **AMBER** | Capability exists and passes some journey steps, but the full journey hasn't been tested end-to-end, OR the journey was tested with mocks/stubs instead of real components | None (evidence missing) |
| **GREEN** | Capability passes ALL journey steps with REAL components: classified correctly, discovered by exact signature, executed with correct output, verified against evidence, qualified through policy, promoted through governance, reused at zero LLM cost, and revoked cleanly |

### The capability journey (all steps must pass)

```
 1. CLASSIFY     Signal arrives → Cascade classifies → routed to correct lane
 2. DISCOVER     VERGE registry lookup → exact signature match found
 3. EXECUTE      Sandboxed runtime executes the graph → correct output
 4. VERIFY       Static + property + differential + adversarial checks pass
 5. QUALIFY      Policy gates satisfied → evidence bound to capability
 6. AUTHORIZE    Promotion record written to ledger → status = authorized
 7. REUSE        Second task of same type → registry hit → zero LLM → correct
 8. TRANSFER     Different model executes the capability → correct output
 9. REVOKE       Revocation issued → capability cannot execute → clean fallback
10. AUDIT        ARE ledger entry exists → chain verified → receipt valid
```

Any step failing → the capability is RED.
Any step untested → the capability is AMBER.
All steps passing with real components → GREEN.

### Scoring

| Score | Criteria |
|---|---|
| 0 | No capability journey tested |
| 1 | Some journey steps tested, others mocked or missing |
| 2 | Full journey tested with real components for ≥1 capability |
| 3 | Full journey tested for ≥5 capabilities across ≥2 domains |

### What CBT does NOT prove

- That the capability is correct for inputs outside the tested distribution
- That the capability will remain correct as the domain evolves
- That the capability is authorized for production use (CBT tests the
  mechanism, not the authorization decision)
- That all possible capabilities would pass (only the tested ones)

---

## The full matrix

### Per-component scoring

| Method | Weight | What it gates |
|---|---|---|
| TDD | 20% | Unit correctness |
| EDD | 20% | Outcome quality vs. baseline |
| CDD | 20% | Interface integrity |
| BDD | 20% | Workflow correctness (including failure paths) |
| CBT | 20% | End-to-end capability journeys |

### Overall status determination

```
IF any method is RED:
    OVERALL = RED (a measured condition failed)
ELIF any method is AMBER:
    OVERALL = AMBER (evidence is missing)
ELIF all methods are GREEN:
    OVERALL = GREEN (all evidence on disk, all passing)
```

**Statuses are NEVER averaged.** One RED makes the whole thing RED.
One AMBER makes the whole thing AMBER. Only ALL GREEN is GREEN.

---

## Evidence requirements (what "on disk" means)

Every GREEN claim must have ALL of:

| Evidence type | Format | Location |
|---|---|---|
| Test execution output | Text/JSON | CI logs or test runner output |
| Dataset digest | SHA-256 hex | Experiment manifest |
| Result manifest | JSON | `experiments/<id>/results/<run>/result.json` |
| Environment stamp | JSON | Python version, verge version, git commit, timestamp |
| Contract validation | Test output | CDD test suite output |
| Scenario execution | BDD output | Behave/jest/playwright output |
| Journey trace | Execution log | CBT test output showing all 10 steps |

**If the evidence is not on disk, the claim is AMBER (at best).**

---

## Honest labeling requirements

Every test result and evaluation must carry honest labels:

| Label | Meaning | Requirement |
|---|---|---|
| `simulated` | Used a test double, not a real model | Cannot support claims about real-model behavior |
| `real-api` | Used a real model via API | Can support claims about that specific model |
| `LIVE` | Real invocation at evaluation time | Strongest evidence label |
| `REHEARSAL` | Retained response replay | Cannot support claims about current behavior |
| `OFFLINE` | Deterministic computation | Cannot support claims about model reasoning |
| `empirical` | Finite-sample agreement | NEVER proof; always carries a claim scope |
| `proven` | Formal verification (SMT, Coq, etc.) | The only "proof" label; extremely rare |

**Mixing labels dishonestly is the worst violation of this rubric.**
Labeling simulated results as real, or rehearsal as live, invalidates
every claim in the associated artifact.

---

## The claim scope (mandatory honesty field)

Every GREEN status carries a claim scope — a mandatory statement of
what the evidence does and does NOT establish:

```json
{
  "claim_scope": "Empirical property agreement on 51 cases (1 declared + 50 generated)
                  within the seeded distribution. Not a formal proof. No claims
                  about inputs outside the generator distribution. No claims about
                  production behavior. Cross-proposer consensus on 3 architectures
                  at temperature 0; hardware-nondeterministic; rates are point
                  estimates with n=20, not bounds."
}
```

**A GREEN without a claim scope is AMBER.** The claim scope is what
prevents "tests passed" from being read as "it works in production."

---

## Running the matrix

```bash
# Full five-part matrix (the quality gate)
uv run python rubric/score.py

# Individual suites
uv run python -m pytest tests/unit -m unit -q          # TDD
uv run python -m pytest tests/contracts -m cdd -q      # CDD
uv run python -m pytest tests/examples -m edd -q       # EDD
uv run python -m behave tests/bdd/features -q          # BDD
uv run python -m pytest tests/integration -m cbt -q    # CBT
```

**The gate passes only when ALL FIVE suites are GREEN.**

---

## This rubric governs

| Artifact | Must follow this rubric |
|---|---|
| Source code changes | TDD + CDD |
| Experiment runs | EDD |
| New capabilities | CBT |
| User-facing features | BDD |
| Performance claims | EDD + CBT |
| Security claims | BDD (failure paths) + adversarial testing |
| Production readiness | ALL FIVE GREEN + claim scopes reviewed |

---

## Common misreadings (and corrections)

| Misreading | Correction |
|---|---|
| "All tests pass, so it works" | Tests passing ≠ correctness. The tests test what they test. |
| "94% cache hit rate means we save 94%" | Only on the tested distribution with the tested workload. |
| "The consensus oracle has precision 1.0" | On n=20 tasks. One more data point could change it. |
| "It's green, so it's production-ready" | GREEN means "evidence exists and passes." Production readiness requires additional criteria (HA, monitoring, load testing, security review). |
| "We proved the model is correct" | We showed empirical agreement on finite cases. Proof requires formal methods. |
| "The capability is verified, so it's authorized" | Verification and authorization are SEPARATE. Verified ≠ authorized. |
| "Simulated results tell us about real models" | They tell us about the mechanism, not about real-model behavior. |
| "The evaluation was fair" | The evaluation was frozen before measurement. Fairness of the dataset itself is a separate question. |
