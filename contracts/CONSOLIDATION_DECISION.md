# Contract Consolidation Decision — Issue #260

## Decision: `contracts/earn-quest` is the canonical implementation

**Date:** 2026-04-24  
**Issue:** #260 — Contract Codebase Consolidation  
**Status:** ✅ Resolved

---

## Comparison Summary

| Dimension | `contracts/earn-quest` ✅ | `Contract/earn-quest` ❌ |
|---|---|---|
| **Test files** | 14 integration test files | 13 test files |
| **Test snapshots** | 114 deterministic snapshots | None |
| **Source modules** | 16 (incl. events, security, validation, stats) | 17 (incl. dispute, templates, upgrade, pausable) |
| **Pause mechanism** | Simple admin-controlled multi-sig + timelock | Complex multi-sig with grace period, separate `pausable` module |
| **Event system** | Full indexed event module + `EVENT_SPECIFICATION.md` | No dedicated event module |
| **Validation** | Dedicated `validation.rs` with constants and helpers | Inline, scattered |
| **Batch operations** | `register_quests_batch`, `approve_submissions_batch` | None |
| **Quest metadata** | `QuestMetadata` with title, description, tags | None |
| **Platform/creator stats** | `PlatformStats`, `CreatorStats` | None |
| **Quest queries** | Filter by status, creator, reward range, active | None |
| **Escrow model** | `EscrowInfo` struct with full audit trail | Simple balance only |
| **`register_quest` signature** | No `max_participants` (open quests) | Requires `max_participants` |
| **Indexer support** | `examples/indexer-example.ts` + event spec doc | None |
| **Workspace structure** | Single crate, clean | Workspace with `stellar_earn` scaffold |
| **Crate name** | `earn_quest` | `earn-quest` |

---

## Rationale

`contracts/earn-quest` is the clear canonical choice for the following reasons:

1. **More complete feature set.** It implements batch operations, rich quest metadata, platform/creator analytics, and advanced query functions that `Contract/earn-quest` lacks entirely.

2. **Superior test coverage.** 114 deterministic test snapshots provide regression safety that `Contract/earn-quest` cannot match. Snapshot tests catch subtle storage and event changes automatically.

3. **Dedicated event system.** The `events.rs` module with indexed topics and the accompanying `EVENT_SPECIFICATION.md` are production-ready for off-chain indexers and subgraph integration. `Contract/earn-quest` has no event module.

4. **Explicit validation layer.** `validation.rs` centralises all input constraints (reward bounds, deadline checks, status transitions, batch limits) with documented constants. `Contract/earn-quest` scatters validation inline.

5. **Richer escrow model.** `EscrowInfo` tracks `total_deposited`, `total_paid_out`, `total_refunded`, `deposit_count`, and `created_at`, enabling full fund-movement auditing. `Contract/earn-quest` stores only a balance.

6. **Aligned with issue preference.** Issue #260 explicitly states: *"prefer contracts/earn-quest"*.

### What `Contract/earn-quest` had that is not in `contracts/earn-quest`

- `max_participants` field on quests — intentionally absent in `contracts/earn-quest`; open quests are the design choice.
- `dispute.rs` / `templates.rs` — experimental modules, not referenced by any test.
- `upgrade.rs` / `trigger_migration` / `trigger_rollback` — upgrade tooling; can be ported if needed in a follow-up.
- `pausable` with grace-period emergency withdrawals — `contracts/earn-quest` uses a simpler but sufficient multi-sig + timelock model.

None of these missing pieces block production use. They can be evaluated and ported individually as separate issues.

---

## Action Taken

- `contracts/earn-quest` — **retained as canonical codebase** (no changes).
- `Contract/` — **archived** by renaming to `Contract_ARCHIVED/`. The directory is preserved in git history and can be deleted after a review period.

---

## Next Steps

1. Update CI/CD pipelines to point exclusively at `contracts/earn-quest`.
2. Update the root `README.md` to reference `contracts/earn-quest`.
3. Evaluate porting `upgrade_contract` / migration tooling from `Contract/earn-quest` (separate issue).
4. Delete `Contract_ARCHIVED/` after team sign-off.

Closes #260
