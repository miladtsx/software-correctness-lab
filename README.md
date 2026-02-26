# Software Correctness Lab

A deterministic reliability lab for exploring failure modes, invariants, and recovery boundaries in infrastructure systems.

## Methodology

This repository exists to answer one question repeatedly:

> Can we reproduce a failure mode, detect it, prevent/recover it, and prove the invariant holds?

Current implementation artifact:

- **Experiment 01: Predictable Job Queue**

<details><summary>Potential next experiments</summary>

- Liveness under partial partition
- Retry amplification loops
- State divergence drills
- Control-plane starvation
- Idempotency boundary violations
- Backpressure collapse

</details>

---

## Experiment 01: Predictable Job Queue (correctness under stress)

A minimal job queue built to demonstrate **fast + reliable** backends: explicit invariants, disciplined retries, and recovery paths that preserve correctness under stress.

Avoid common mistakes: duplicate execution, partial writes, stuck queues.

## Guarantees

- **Delivery semantics:** we target **at-least-once delivery** with an idempotency boundary that delivers **exactly-once external effects** when handlers honor the contract.
- **Handler contract:** handlers must be idempotent by stable job key (safe to re-run without changing the final correct state).
- **Retries & dead letters:** retries are bounded by policy budget; once exhausted, jobs are moved to dead-letter for explicit operator/actionable recovery.
- **Recovery outcome:** “correctness restored” means all protected invariants are re-established (`INV_XXX`), with no duplicate committed effects and reconciled state.

## Why this matters

- Protect money & trust (no duplicate charges, no silent drift)
- Move faster safely (idempotency, retries, replay, reconciliation)
- Debug in minutes (deterministic scenarios + tests)

## Design principles

- Invariants first: correctness is defined explicitly
- Reliability is a feature: recovery restores correctness, not just uptime
- Small scope on purpose: easy to reason about / fork

## Repo map

- Scope: [`docs/00_scope.md`](./docs/00_scope.md)
- Invariants: [`docs/01_invariants.md`](./docs/01_invariants.md)
- Failure-mode methodology: [`docs/03_failure_modes.md`](./docs/03_failure_modes.md)
- Failure-mode bundles: `failure_modes/FM_XXX_*`
  - `spec.md` (trigger, symptoms, invariant, detection, recovery/prevention, acceptance)
  - `scenario.py` (how to trigger)
  - tests: repro + prevent

## System (Experiment 01)

- Minimal job processor: queue + worker + store + deterministic clock
- Policies (commit/reconcile/budgets) exist only to protect invariants

## Baseline

- Happy path story: [`docs/04_happy_path.md`](./docs/04_happy_path.md)
- Proof: [`tests/test_happy_path.py`](./tests/test_happy_path.py)

## Run

```bash
make sync
make test
```

## Current focus

**FM_001 — retry duplication → double execution**

```bash
make test-fm1       # reproduces
make test-fm1-fix   # proves prevention boundary
```

Index: [`docs/failure_mode_index.md`](./docs/failure_mode_index.md)

## When to add a new runtime/experiment

Stay queue-first until a failure mode can’t be represented faithfully in this model. Add a new runtime only if:

- The failure mode requires independent system boundaries not modelable here

- The invariant depends on protocol properties absent from this model (e.g., quorum/consensus)

- Cross-system correctness can’t be expressed as queue + policy + deterministic fault injection

## Contributing: add a failure-mode exercise

1. Create `failure_modes/FM_XXX_name/`
2. Write `spec.md` (trigger, symptoms, violated `INV_XXX`, detection, recovery/prevention, acceptance)
3. Add two tests: `test_repro_fmxxx.py` and `test_prevent_fmxxx.py`
4. Implement the smallest change that satisfies the prevention test
5. Keep it deterministic (use `runtime/clock.py`), one failure mode per change set
