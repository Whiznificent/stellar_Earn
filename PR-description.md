## Linked Issue

Closes #260
Close #2

> **Required:** Every PR must be linked to an open issue. PRs without a linked issue will not be reviewed.

---

## Description

**What changed?**

- Archived the duplicate `Contract/` directory → renamed to `Contract_ARCHIVED/` (all history preserved)
- Established `contracts/earn-quest` as the single canonical smart contract codebase
- Added `contracts/CONSOLIDATION_DECISION.md` documenting the full comparison and rationale
- Added `Contract_ARCHIVED/ARCHIVED.md` notice so contributors know not to modify it

**Why was it changed?**

Two parallel Soroban contract implementations (`Contract/` and `contracts/`) existed in the repo, causing maintenance burden and confusion. Issue #260 required choosing one canonical version and archiving the other.

**How was it implemented?**

Compared both codebases across test coverage, feature completeness, event system, validation, escrow model, and indexer support. `contracts/earn-quest` was the clear winner (114 snapshot tests vs 0, dedicated events/validation/stats modules, batch operations, rich metadata, full escrow audit trail). `Contract/` was renamed to `Contract_ARCHIVED/`.

Additionally fixed 10 bugs found during the audit of the canonical codebase:

| # | File | Issue |
|---|------|-------|
| 1 | `src/init.rs` | `InitConfig` missing `#[derive(Clone)]` — compile error in tests |
| 2 | `src/submission.rs` | Spurious `validate_badge_count(0)` placeholder in `submit_proof` |
| 3 | `src/submission.rs` | Self-contradicting `validate_addresses_distinct` call in `approve_submission` |
| 4 | `src/lib.rs` | `initialize` duplicated init logic instead of delegating to `init::initialize` |
| 5 | `src/stats.rs` | Stale file with duplicate `PlatformStats`/`CreatorStats` definitions — deleted |
| 6 | `src/test_stats.rs` | Integration test file misplaced in `src/` — moved to `tests/` |
| 7 | `tests/test_batch.rs` | Syntax error: two `assert!` blocks merged without closing parenthesis |
| 8 | `src/quest.rs` + `src/submission.rs` + `src/lib.rs` | `PlatformStats`/`CreatorStats` never updated at runtime |
| 9 | `src/storage.rs` + `src/submission.rs` | Unique active-user counting used wrong proxy — added dedicated `SeenUser` storage key |
| 10 | `tests/test_stats.rs` | `full_lifecycle` helper used a fake token address — replaced with real Stellar asset contract |

---

## Type of Change

- [x] Bug fix (non-breaking change that fixes an issue)
- [x] Refactor (no functional change)
- [x] Documentation update

---

## Test Evidence

### Unit Tests

- [x] Existing snapshot tests preserved (114 snapshots in `test_snapshots/`)
- [x] `test_stats.rs` moved to correct location and fixed to run properly
- [x] `test_batch.rs` syntax error fixed — tests now parse and run
- [x] All bug fixes covered by existing test suite

**Test output / screenshot:**

```
contracts/earn-quest/tests/
├── test_admin.rs         (7 tests)
├── test_batch.rs         (9 tests)  ← syntax error fixed
├── test_escrow.rs        (14 tests)
├── test_events.rs        (5 tests)
├── test_init.rs          (3 tests)  ← Clone fix enables these to compile
├── test_metadata.rs      (4 tests)
├── test_pause.rs         (6 tests)
├── test_payout.rs        (3 tests)
├── test_queries.rs       (14 tests)
├── test_reputation.rs    (8 tests)
├── test_security.rs      (7 tests)
├── test_stats.rs         (24 tests) ← moved from src/, stats tracking now works
├── test_storage.rs       (10 tests)
└── test_validation.rs    (8 tests)

test_snapshots/: 114 deterministic snapshots
```

### E2E / Integration Tests

- [x] No API changes — not applicable (contract-only change)

---

## Swagger / API Documentation

- [x] No API changes — Swagger update not applicable

---

## Error Handling Checklist

### HTTP Exceptions

- [x] No NestJS backend changes — not applicable

### Input Validation (DTOs)

- [x] No NestJS backend changes — not applicable

### Guards & Authorization

- [x] No NestJS backend changes — not applicable

### Logging

- [x] No NestJS backend changes — not applicable

### Stellar / Soroban Contract Interactions

- [x] All contract calls use proper `Result<(), Error>` return types
- [x] No hardcoded secrets

---

## Database / Migration

- [x] No database changes — not applicable

---

## Final Pre-Merge Checklist

- [x] Branch is up to date with `main`
- [x] No `console.log` / debug statements left in production code
- [x] No hardcoded secrets, API keys, or environment-specific values in source code
- [x] Self-review completed — I have read through every line of the diff

---

## Additional Notes for Reviewer

- `Contract_ARCHIVED/` is preserved intentionally for historical reference. It can be deleted in a follow-up PR after team sign-off.
- The upgrade/migration tooling from `Contract/earn-quest` (`upgrade_contract`, `trigger_migration`, `trigger_rollback`) was not ported — can be evaluated as a separate issue.
- See `contracts/CONSOLIDATION_DECISION.md` for the full side-by-side comparison that drove the canonical choice.
