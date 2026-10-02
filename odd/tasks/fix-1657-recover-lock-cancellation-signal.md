# Fix #1657: forward AbortSignal on recover-lock and clean dead NATIVE_REVIEW_OPERATION.VERSION check

## Objective

Ensure the caller's `AbortSignal` is forwarded to the native `reclaim` operation when invoking `recover-lock` in `executeReviewControllerOperation` (`extensions/gentle-ai.ts`). Remove dead reference to `NATIVE_REVIEW_OPERATION.VERSION` in target status diagnostic preservation.

## Problem

In `extensions/gentle-ai.ts`, the `RECOVER_LOCK` branch calls `executeNativeRecoveryRoute` with 7 arguments instead of 6:
```ts
return await executeNativeRecoveryRoute(parameters.operation, "reclaim", input, defaultCwd, nativeReviewCli, undefined, signal);
```
Because `executeNativeRecoveryRoute` expects `(operation, nativeOperation, input, cwd, nativeReviewCli, signal)`, passing an extra `undefined` before `signal` places `signal` into the discarded 7th argument slot, resulting in `signal` being `undefined` inside `executeNativeRecoveryRoute`. As a result, cancellation (e.g. via `AbortController` or pressing `Esc`) never reaches the native `review reclaim` process.

Additionally, `extensions/gentle-ai.ts:6350` checks `nativeDiagnostics?.operation === NATIVE_REVIEW_OPERATION.VERSION`, but `VERSION` does not exist on `NATIVE_REVIEW_OPERATION` (causing TS2339).

## Scope

- In `extensions/gentle-ai.ts`, remove the extra `undefined` argument in the `RECOVER_LOCK` call to `executeNativeRecoveryRoute`, passing `signal` in the 6th slot.
- Remove `NATIVE_REVIEW_OPERATION.VERSION` from `preservesNativeTargetStatusDiagnostic` in `extensions/gentle-ai.ts`.
- In `tests/review-controller-native-recovery.test.ts`, add a unit test proving that `recover-lock` forwards the caller's `AbortSignal` to the native reclaim method.
- Update typecheck baseline via `pnpm run typecheck -- --update` or verify zero regressions.

## Tasks

- [x] T1 Reproduce #1657 with failing unit test in `tests/review-controller-native-recovery.test.ts` (RED).
- [x] T2 Fix argument forwarding and clean `NATIVE_REVIEW_OPERATION.VERSION` in `extensions/gentle-ai.ts` (GREEN).
- [x] T3 Verify full test suite, runtime module checks, and typechecks.
- [x] T4 Commit work unit and document verification evidence (commit `7a297003`).

## Verification Evidence

- **RED observed**:
  - `RECOVER_LOCK forwards the caller AbortSignal to native reclaim`: failed with `strictEqual` assertion error because `calls[0]?.signal` was `undefined` while expecting `AbortSignal`.
- **GREEN observed**:
  - Test passed after removing extra `undefined` argument in `executeNativeRecoveryRoute`.
  - `node --experimental-strip-types --test tests/review-controller-native-recovery.test.ts`: 22 passed, 0 failed.
  - `pnpm run typecheck`: clean (baseline updated from 200 to 185, TS2554 and TS2339 in `gentle-ai.ts` resolved).
  - `pnpm run check:runtime-modules`: clean (8 generated modules).
  - `pnpm test`: 4,539 passed, 0 failed, 34 skipped (all three stages PASS: `unit-tests`, `provider-contract`, `runtime-harness`).


## Acceptance Criteria

- `executeReviewControllerOperation` with `recover-lock` forwards `signal` to `nativeReviewCli.reclaim`.
- TS2554 (expected 6 arguments, got 7) and TS2339 (property VERSION does not exist) are resolved in `extensions/gentle-ai.ts`.
- All tests in `tests/review-controller-native-recovery.test.ts` and the full test suite pass.
