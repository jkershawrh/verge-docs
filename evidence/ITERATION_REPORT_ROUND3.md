# VERGE Iteration Report — Round 3 (R-17 hardening + Phase 5a real models)

Charter §13 format. Evidence-backed; scope stated per claim.

## Hypothesis

(a) R-17: distribution-hostile verification closes the known
slice-bypass class. (b) Phase 5a / H0 core: REAL, architecturally
independent models can propose verified capabilities, and a capability
qualified by one model is reused by another with zero re-reasoning.

## Implementation

- Adversarial case generation (`verge.verification.adversarial`) wired into
  property + differential verification; the `string.slice(0,12)` bypass
  reproduced red end-to-end, then failed closed.
- `OllamaBackend` + `OpenAICompatBackend` (a LiteLLM API gateway/LiteLLM/vLLM/KServe
  endpoint-agnostic), real server-side token accounting, bounded
  retry-with-feedback, no self-oracle; engine oracle seam injects
  task-level ground truth (D-013/D-015).
- Tier ladder executed: claude-haiku-4-5 (a LiteLLM API gateway, owner-authorized bounded
  use), granite3.3:8b and qwen2.5:7b (local).

## Verification (what actually ran)

- 807+ pytest (unit/contracts/EDD/CBT) + 8 BDD scenarios green after all
  changes; every closed adversarial vector pinned by red-first regression.
- e1-real run-2-a LiteLLM API gateway, e1-real run-1 (local), e8-real run-1 (local),
  e8-real run-2-a LiteLLM API gateway — all with EDD-frozen manifests and result
  manifests on disk.

## Performance (measured, frozen datasets)

| Proposer | Admission | Qualification cost | Reuse correctness |
|---|---|---|---|
| claude-haiku-4-5 | **5/5** | 6 calls, 11,322 tokens, 2.4–11.8s/family | 50/50, 0 LLM |
| granite3.3:8b | 3/5 | 5 calls, 9,050 tokens | 30/30, 0 LLM |
| qwen2.5:7b | 3/5 | 4 calls, 6,001 tokens | 30/30, 0 LLM |

- Local-tier failures are COMPLEMENTARY (granite: sort/normalize; qwen:
  reverse/normalize) — a two-proposer local pool covers 4/5 families.
- Cross-model behavioral agreement on jointly-solved families: **10/10**
  (independent architectures, equivalent verified programs).
- Both local models failed `normalize` (2-op composition) — the
  composition threshold for this prompt template sits between 8b-local and
  frontier tier.
- **Transfer**: claude→qwen 50/50 correct at ZERO transferer LLM calls
  (qwen cannot compose these tasks itself); granite→qwen 30/30 at zero.

## Economics

Qualifying all 5 families cost ~11.3k haiku tokens once; every subsequent
execution across every model is LLM-free. The cheap-proposer safety result:
wrong local proposals cost only their tokens (~6–9k across the run) and
were all discarded by verification.

## Failures and uncertainties

- 5 synthetic data-transform families; one frontier proposer (haiku;
  sonnet untested); opset v1.1.0 pure ops only (R-14 bounds
  generalizability).
- Local tier admission is prompt-template-dependent (v3 frozen); engine
  -level verify-feedback repair (bounded re-solve on semantic failure) is
  unbuilt and would likely raise local admission rates.
- Hardware nondeterminism: rates, not capability ids, are comparable
  across runs.
- Operational incident (transparency): concurrent e8-real runs shared the
  run-1 label and one overwrote the other; fixed by run labels
  (`--run=`), both results restored on disk with distinct labels.
- 5 adversarial vectors remain open (R-07 ledger anchoring highest).

## Decision

**Retain; H0 is supported with real models within scope.** The mechanism
chain — real proposal → verification → admission → zero-LLM cross-model
reuse — held at every tier tested, with wrong proposals failing closed.
Phase 5b candidates: engine-level bounded repair for weak proposers;
multi-proposer registry pooling; a remote GPU cluster Gaudi tier (blocked on VPN +
kubeconfig); sonnet/stronger proposers; broader task corpora against R-14.

## Next experiment

Multi-proposer pooling is the cheapest high-value step: one registry fed
by BOTH local models plus claude — measure union coverage (predicted 5/5)
and cross-proposer equivalence at scale, all with existing components and
no new infrastructure.
