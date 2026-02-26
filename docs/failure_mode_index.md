# Failure Mode Index

Map of failure modes to the invariants they threaten, the proving tests, and the recovery lever we exercise.
Runtime under test today: **Experiment 01 (Job Queue)**.
This index is the primary progress ledger for the lab methodology.
New experiments/runtimes are introduced only when an FM cannot be represented in Experiment 01 without violating modeling fidelity.

| Failure Mode | Description | Violated Invariants | Repro Test | Prevent Test | Recovery / Policy Lever |
| --- | --- | --- | --- | --- | --- |
| `FM_001` Duplicate Retry | Lease timeout causes duplicate execution of the same job. | [`INV_001`](./01_invariants.md#inv_001----job-execution-is-logically-idempotent), [`INV_002`](./01_invariants.md#inv_002----partial-execution-must-not-leave-irreversible-damage), [`INV_004`](./01_invariants.md#inv_004----recovery-restores-correctness-not-just-availability) | [`test_repro_fm001`](../failure_modes/FM_001_duplicate_retry/tests/test_repro_fm001.py) | [`test_prevent_fm001`](../failure_modes/FM_001_duplicate_retry/tests/test_prevent_fm001.py), [`test_recover_fm001`](../failure_modes/FM_001_duplicate_retry/tests/test_recover_fm001.py) | Commit boundary (`policies/commit.py`) + reconcile finalization (`policies/reconcile.py`) |

FM details live in each FM’s `spec.md`.
