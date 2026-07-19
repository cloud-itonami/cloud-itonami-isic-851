# cloud-itonami-isic-851

**Pre-primary and Primary Education** — ISIC Rev.4 class 851.

A coordination-only actor for primary/pre-primary school back-office administration, behind an independent Governor that earns advisor trust through structured oversight: proposal → advise → govern → decide → commit|hold|escalate.

## Features

- **Closed proposal-op allowlist**: `log-attendance-note`, `schedule-parent-guardian-meeting`, `coordinate-supply-request`, `schedule-staff-shift-proposal`, `flag-safety-concern` (all `:effect :propose`).
- **Three HARD governor checks** (permanent, un-overridable):
  1. **Student verified** — target must exist AND be registered/verified in the store.
  2. **Effect is :propose** — any other `:effect` value is rejected.
  3. **Scope exclusion** — grading/academic assessment, curriculum/pedagogical decisions, disciplinary action (suspension/expulsion), special-education (IEP) determinations, custody/guardianship decisions, and safety-authority overrides are permanently blocked.
- **Staged rollout** (Phase 0→3):
  - Phase 0: read-only
  - Phase 1: attendance-note logging only (approval-gated)
  - Phase 2: + parent/guardian meeting, supply, staff-shift proposals (approval-gated)
  - Phase 3: auto-commits clean, high-confidence proposals (safety concerns always escalate)
- **Append-only audit ledger** — every decision is an immutable log entry.
- **langgraph-clj StateGraph** — one request = one supervised run; human-in-the-loop via `interrupt-before`.

## Development

```bash
# Install dependencies (if inside the superproject, use :dev alias for local overrides)
clojure -M:dev -P

# Run tests
clojure -M:dev:test

# Run linter
clojure -M:lint

# Run demo
clojure -M:dev:run
```

## Test suite

- `test/schoolops/governor_test.clj` — unit tests of governor hard checks and scope exclusion
- `test/schoolops/advisor_test.clj` — advisor proposal shape and consistency
- `test/schoolops/phase_test.clj` — rollout phase logic
- `test/schoolops/governor_contract_test.clj` — full graph integration, audit trail
- `test/schoolops/store_contract_test.clj` — Store protocol and MemStore implementation

## Modules

- `schoolops.store` — SSoT (MemStore, String-keyed student directory, append-only ledger)
- `schoolops.advisor` — contained intelligence node (mock + real-LLM seam)
- `schoolops.governor` — independent compliance layer
- `schoolops.phase` — staged rollout (0→3)
- `schoolops.operation` — langgraph-clj StateGraph
- `schoolops.sim` — demo driver
- `schoolops.render-html` — build-time generator for `docs/samples/operator-console.html` (`clojure -M:dev:render-html`)

## License

AGPL-3.0-or-later. See LICENSE file.

## Governance

This actor is part of the cloud-itonami Wave 4 (human-services) fleet, and is the
first actor in the ISIC-85 education cluster. See ADR-2607152900 (this repo's own
decision record), ADR-2607121000 (Wave definition), and ADR-2607152700 (ISIC-873
elderly residential care — module-shape precedent this repo mirrors) for design
decisions.
