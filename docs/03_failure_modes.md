# Failure Modes Methodology

> This document defines the required structure for adding a failure mode (FM) to this repository.

## Single source of truth

- Invariants: [docs/01_invariants.md](./01_invariants.md)
- State model: [docs/02_state_model.md](./02_state_model.md)
- Policies: [docs/03_policies.md](./03_policies.md)
- Failure Mode Index: [docs/failure_mode_index.md](./failure_mode_index.md)
- FM-specific truth: [failure*modes/FM_XXX*\*/spec.md](../failure_modes/)

## FM bundle structure

Each failure mode lives in its own directory:

```text
failure_modes/FM_XXX_short_name/
  spec.md
  scenario.py
  tests/
    test_repro_fmxxx.py
    test_prevent_fmxxx.py
    test_recover_fmxxx.py   # optional
```

Required bundle artifacts:

- `spec.md`: single-source declaration of the failure mode, violated invariants, and acceptance criteria.
- `scenario.py`: deterministic setup/driver that reproduces the condition described in `spec.md`.
- `tests/`: executable proofs that the failure is reproducible and preventable.

## Required fields in `spec.md`

Every FM `spec.md` must include these sections (exact heading text may vary slightly, but content is required):

1. **Description**
   - One-paragraph statement of what fails and why it matters.
2. **Trigger**
   - Precise preconditions + event sequence needed to activate the failure mode.
3. **Symptoms**
   - Observable incorrect outcomes (state, side effects, signals, logs, counters).
4. **Violated invariants**
   - Explicit references to `INV_XXX` from [`docs/01_invariants.md`](./01_invariants.md).
5. **Detection**
   - How the system/test harness can detect the failure unambiguously.
6. **Recovery / prevention strategy**
   - Minimal policy or runtime mechanism that restores or preserves correctness.
7. **Acceptance criteria**
   - Concrete, testable statements that the repro/prevent(/recover) tests must prove.

## Required tests

Each FM must include:

1. **Repro test** (`test_repro_fmxxx.py`) — required
   - Demonstrates the failure mode is real and reproducible under controlled conditions.- Must demonstrate the broken outcome **without** the prevention mechanism enabled.

2. **Prevent test** (`test_prevent_fmxxx.py`) — required
   - Proves the chosen prevention mechanism preserves the referenced invariants.
   - Must prove the invariant holds **with** the prevention mechanism enabled.

3. **Recover test** (`test_recover_fmxxx.py`) — optional but recommended
   - Use when recovery is distinct from prevention (e.g., reconcile/repair after partial failure).
   - Proves correctness is restored, not just availability.

## How to name `INV_XXX` references

- Use the exact invariant ID format: `INV_001`, `INV_002`, etc.
- In prose, reference the invariant ID and point to the canonical invariant doc [`docs/01_invariants.md`](./01_invariants.md).
- Do not invent local invariant aliases per FM; IDs are global and stable.

## Determinism requirements in this repository

“Deterministic” in this repo means a test/scenario is reproducible with the same result on repeated runs, independent of wall-clock timing or external state.

Required practices:

- Use test-controlled time (e.g., `runtime/clock.py`), not real `time.sleep`.
- Use isolated, fresh in-memory SQLite state per test.
- Inject failures explicitly via fault injectors; do not depend on random or ambient failures.
- Avoid race-sensitive assertions that rely on scheduler luck.
- Assert invariant outcomes directly (state/effects) rather than only incidental signals.

## Catalog linkage

This methodology defines **how** failure modes are authored and proven.
The current FM catalog and progress ledger live in:

- [`Failure Mode Index`](./failure_mode_index.md)

## Content placement rule

- The index is a catalog (title/status/invariants/links) only.
- This doc defines authoring rules only.
- All FM details live in `failure_modes/FM_XXX_*/spec.md`.
- Avoid deep dives in the index to prevent duplication
