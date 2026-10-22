# E10 Analysis — opset expressiveness, measured

Raw result: `experiments/e10_opset_expressiveness/results/run-1/result.json`
(coverage 2/15). This note classifies every failure precisely — the naive
read ("13% coverage") is not the finding.

## Failure classification (from recorded error strings)

| Task | Predicted | Outcome | Classification |
|---|---|---|---|
| t01 nested extraction | expressible | compile fail: config nesting (`key` at top level instead of under `config`) | **proposer format-failure** — task IS expressible (sequential list.maps) |
| t02 case-preserving dedup | expressible | verification fail (wrong output) | **GENUINE GAP (discovered)** — needs paired/multi-value element access |
| t03 CSV cleanup | expressible | compile fail (malformed filter config) | **proposer failure** — expressible via `filter(compare.ne, config b="")` |
| t04 filter+sort desc | expressible | compile fail (config `threshold` not op-form) | **proposer format-failure** — expressible (`filter(compare.gt b=50)` → sort → reverse) |
| t05 conditional mapping | expressible | verification fail (wrong output) | **GENUINE GAP (discovered)** — nested conditionals need composed element ops |
| t07–t15 (8 tasks) | gap | all failed | **genuine gaps, confirmed 8/8** (regex, dates, group-by, tostring+pad, stateful scan, zip, predicate composition, multi-field templating) |

## The measured picture

- **As-is coverage: 2/15** (upper-join, flatten — single-op-chain shapes).
- **With engine-level bounded repair** (feeding compile/verification errors
  back to the proposer for one re-solve): predicted **~5/15** — the three
  proposer format-failures are config-nesting errors the format-feedback
  loop would almost certainly fix (the same loop already self-repaired
  granite in e1-real).
- **Genuine opset gaps: 10/15**, in two classes:
  1. *Missing op families* (8): tostring+padding, group-by/count, zip,
     regex, dates, stateful scan.
  2. *Element-op composition limits* (2 discovered + predicate composition
     predicted): the higher-order forms bind ONE op per element with
     config-bound extra ports — no predicate composition, no nested
     conditionals, no per-stage configs in chains, no paired field access.

## Ranked opset v1.2 agenda (by unlock breadth)

1. **Composed element ops** (predicate lambdas / op-pipelines with per-stage
   configs) — unlocks t03/t05/t13/t15 class = 4 corpus tasks.
2. **tostring + pad** — t10 class.
3. **zip / paired access** — t12, t02 class.
4. **group-by/count** — t09 class.
5. **regex, dates** — t07/t08 (domain ops; consider as capability-level
   extensions with their own verification story rather than core ops).
6. **Stateful scans (t11)** — recommend AGAINST in v1.2: state breaks the
   pure-graph verification model; belongs in a v2 side-effect-class design
   with its own permissions contract major version.

## Update: run-2-with-repair (engine bounded repair, commit `e119717`)

Re-running the frozen corpus with the engine-level repair loop active:

- **Coverage 4/15** (from 2/15): t03 (CSV cleanup) and t04 (filter+sort)
  converted — the predicted format-failure class responds to feedback.
- **t01 did NOT convert**: the repair round fixed its config-nesting format
  error (compile passed) but the re-proposal was semantically wrong
  (property verification failed). Honest distinction: **repair converts
  format errors; semantic errors on expressible tasks can persist** —
  t01 reclassifies as proposer-semantics-limited, not opset gap.
- All 10 genuine opset gaps unchanged (correctly: no code path exists).

| Run | Coverage | Notes |
|---|---|---|
| run-1 (baseline) | 2/15 | 3 format-failures + 10 gaps |
| run-2-with-repair | 4/15 | 2 format-failures converted; t01 semantics-limited; 10 gaps |

Baseline result restored from git history after a run-label overwrite
(the same operational mistake as e8-real earlier — run labels now
supported on all runners).

- **Fail-closed held perfectly**: 13 failures, zero wrong admissions, every
  proposer error caught at compile or verification.
- **The bounded-repair gap is engine-level**: the backend's retry loop only
  sees format errors it can detect pre-compile; compile and verification
  failures are terminal today. Feeding them back is the single
  highest-leverage robustness fix (also predicted to raise local-tier
  admission in e1-real).

## System-perspective findings (updated for run-2)
