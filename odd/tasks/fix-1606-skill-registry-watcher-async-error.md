# Tasks: Fix #1606 Unhandled async error (EMFILE) from skill registry directory watcher crashes Pi

## Objective
Prevent Pi from crashing with an `uncaughtException` (e.g. `Error: EMFILE: too many open files, watch`) when an `FSWatcher` instance in `startSkillRegistryWatcher` emits an asynchronous error.

## Root Cause
In `extensions/skill-registry.ts`, `watch(dir, { recursive: true }, refresh)` returns an `FSWatcher` instance. `watch()` is synchronous at creation time, but failures (such as `EMFILE` or permission revocations on certain OSes like macOS) are emitted asynchronously as `'error'` events on the `FSWatcher` (which inherits from `EventEmitter`). Because no `'error'` listener was attached to the watcher, Node.js treats the unhandled `'error'` event as fatal, terminating the entire Pi host process.

## Tasks
- [x] 1. Write failing regression test in `tests/skill-registry.test.ts` proving an unhandled watcher error event crashes without an error listener and is contained gracefully with an error listener (RED).
- [x] 2. Update `extensions/skill-registry.ts` to attach an error listener to each `FSWatcher` that safely closes the watcher and drops it from `activeWatchers` (GREEN).
- [x] 3. Run full skill registry test suite and typecheck verification.
- [ ] 4. Commit work unit with Conventional Commit and publish architectural triage on Issue #1606.

## Evidence
- Reproduction confirmed RED: `Error: EMFILE: too many open files, watch` unhandled event in `tests/skill-registry.test.ts`.
- Verified GREEN: 19/19 tests in `tests/skill-registry.test.ts` pass cleanly (including subtest 11 covering asynchronous watcher error containment, cleanup and activeWatcherCount decrement).
- `npm run typecheck`: clean, 0 regressions.
- `check:provider-contract` and `check:runtime-modules`: clean.
