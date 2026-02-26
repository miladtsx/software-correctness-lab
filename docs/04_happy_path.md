# Baseline Happy Path

Concise reference for the non-failure flow. Rules live in `docs/01_invariants.md`; state transitions live in `docs/02_state_model.md`. This page only pins the happy-path ordering and where we check the boundaries.

---

## Canonical sequence

```mermaid
flowchart TD
  %% agenda helps readers orient the boundary checks


  A["Create job<br/>(PENDING)"] --> B["Lease to worker W1<br/>(LEASED)<br/>set <i>lease_expires_at</i>"]
  B --> C[Mark IN_PROGRESS]
  C --> D{Prior COMMITTED?}
  D -->|no| E["Apply effect once<br/>+ mark COMMITTED"]
  D -->|yes| E2[Short-circuit, no effect]
  E --> F[Mark DONE]
  E2 --> F
  F --> G[Derive job SUCCEEDED]
  G --> H[Record audit: effects_count=1]
  H --> I[Release lease]
  I --> J[No retries issued]

  %% boundary highlighting (opaque group boxes)
  subgraph LeaseBoundary[Lease boundary]
    B
  end
  subgraph CommitBoundary[Commit boundary]
    D
    E
    E2
  end
  subgraph AuditBoundary[Audit & release]
    H
    I
  end

  classDef boundary stroke:#d97706,stroke-width:3px,color:#1f2937,fill:#fef3c7;
  classDef audit stroke:#2563eb,stroke-width:3px,color:#0f172a,fill:#e0f2fe;
  class LeaseBoundary boundary;
  class CommitBoundary boundary;
  class AuditBoundary audit;

  %% legend (agenda already shows order of boundary checks)
```

- Output constraints:
  - one logical effect
  - one COMMITTED execution per job_id
  - zero retries.
- Idempotency boundary
  - sits at `COMMITTED`
  - side effects are illegal before it ([INV_001](./01_invariants.md#inv_001----job-execution-is-logically-idempotent)/[INV_002](./01_invariants.md#inv_002----partial-execution-must-not-leave-irreversible-damage)).
- Transition order mirrors the execution machine from [`state_model`](./02_state_model.md)
  - jobs stay derived (see [`state_model`](./02_state_model.md)).

---

## Lease eligibility (summary)

- **Leasable**: never leased, or lease expired with no terminal success.
- **Not leasable**: latest execution `DONE`, or an unexpired lease exists.

This is the minimum needed for `at-least-once` delivery while avoiding duplicate effects.

---

## Invariant touchpoints

- Lease grant asserts INV_003 (explicit states) and sets up INV_004 (recoverable retries).
- Commit boundary enforces INV_001 / INV_002 (single effect, crash-safe).
- Audit after `DONE` keeps INV_005 observable.

Details stay centralized in `docs/01_invariants.md` to avoid drift.

---

## Failure hook for FM_001

Injection point: between `IN_PROGRESS` and `COMMITTED` (see [`FM_001 spec`](../failure_modes/FM_001_duplicate_retry/spec.md)).  
Scenario: 
  - Worker A times out before commit; 
  - Worker B retries and would double-apply unless the commit boundary short-circuits (policy in [`policies`](./03_policies.md#commit-boundary-idempotency)).
