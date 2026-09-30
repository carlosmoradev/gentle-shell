# Fix #1573: gentle_review input object coercion

## Objective

Allow `gentle_review` and review capture tools to accept JSON objects directly for `input`, `collectBinding`, and `collectBindings`, coercing them to canonical JSON strings, preventing TypeBox validation failures (`input: must be string`) when LLMs emit structured arguments.

## Constraints

- Strict TDD: observe focused RED before changing implementation.
- Support both serialized JSON strings (existing behavior) and direct JSON objects (coerced).
- Reject invalid inputs (non-object primitives like numbers or booleans where objects are expected).
- Conventional commits only, no AI co-author attribution.
- Feature branch `fix/1573-gentle-review-input-coercion`.

## Tasks

- [x] 1. Write failing regression tests for object input in gentle_review START/assess and capture tools (RED).
- [x] 2. Update REVIEW_CONTROLLER_PARAMETERS, REVIEW_CAPTURE_PARAMETERS, and parser normalization in extensions/gentle-ai.ts (GREEN).
- [x] 3. Run focused tests and verify no regressions in review controller suite.
- [ ] 4. Commit work unit with conventional commit.

## Verification Evidence

- RED observed:
  * Subtest 6 failed asserting nested object was rejected with string serialization guidance.
  * Subtest 8 failed with TypeBox validation error: "Validation failed for tool \"gentle_review\": - input: must be string".
- GREEN observed:
  * All 8 subtests in tests/review-controller.test.ts passed in 1.7s.
  * Full review suite (189 tests across 4 files) passed with 0 failures in 23s.
  * npm run typecheck passed with 0 regressions.
  * Runtime module check and provider contract check passed.
