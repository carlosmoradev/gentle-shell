# Tasks: Fix #1625 Vim editor host gate rejected Pi >=0.99.2 bump

## Objective
Restore green CI and verify status on `main` by updating `SUPPORTED_VERSIONS` and `resolveVimRuntime()` to certify Pi 0.99.2 (and 0.99.1), synchronizing `tests/package-manifest.test.ts`, `tests/vim-editor-adapter.test.ts`, and `tests/gentle-shell.test.ts`.

## Root Cause
PR #1624 bumped Pi devDependencies to `>=0.99.2` in `package.json` and committed a lockfile resolving `@earendil-works/pi-tui@0.99.2` and `@earendil-works/pi-coding-agent@0.99.2`. However, `SUPPORTED_VERSIONS` in `lib/vim-editor-adapter.ts` and `resolveVimRuntime()` in `extensions/gentle-shell.ts` remained hardcoded to `0.99.1`. Consequently, all Vim editor identity checks failed closed with `Error: Unsupported Pi editor layout/version`, breaking 114 tests across the test suite and turning CI red on `main`.

## Tasks
- [x] 1. Export and expand `SUPPORTED_VERSIONS` in `lib/vim-editor-adapter.ts` to include `"0.99.2"` alongside `"0.99.1"`.
- [x] 2. Update `resolveVimRuntime()` in `extensions/gentle-shell.ts` to use `SUPPORTED_VERSIONS` as single source of truth.
- [x] 3. Update `tests/package-manifest.test.ts` devDependencies assertion to `>=0.99.2`.
- [x] 4. Update `tests/vim-editor-adapter.test.ts` and `tests/gentle-shell.test.ts` to test against the installed Pi runtime version.
- [x] 5. Full test verification, typecheck clean (0 regressions).
- [ ] 6. Commit work unit with Conventional Commit and prepare triage comment for Issue #1625.

## Evidence
- `tests/package-manifest.test.ts`: 55/55 passed.
- `tests/vim-editor-adapter.test.ts`: 40/40 passed.
- `tests/gentle-shell.test.ts`: 243/243 passed.
- `npm run typecheck`: clean (0 regressions).
- `check:provider-contract` & `check:runtime-modules`: clean.
