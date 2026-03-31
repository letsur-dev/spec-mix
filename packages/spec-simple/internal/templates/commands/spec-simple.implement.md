---
description: Execute implementation based on plan phases
---

## Execution Flow

### 1. Detect Feature

```bash
BRANCH=$(git branch --show-current 2>/dev/null)
SPEC_DIR="specs/$BRANCH"
```

If not on a feature branch, check `specs/` for the most recent feature directory.

### 2. Load Context

- Read `$SPEC_DIR/spec.md` (required)
- Read `$SPEC_DIR/plan.md` (required — abort if missing)
- Read `specs/constitution.md` if it exists

### 3. Choose Implementation Mode

Present the user with implementation mode options:

```text
Implementation mode:

| Mode     | Description                                        |
|----------|----------------------------------------------------|
| standard | Phase-by-phase with walkthrough review (default)   |
| stdd     | STDD methodology — Test-First with decoupled roles |

Select mode [standard]:
```

- **standard** → Continue to Step 4 (Standard Mode)
- **stdd** → Jump to Step 4-STDD (STDD Mode)

---

## Standard Mode

### 4. Identify Current Phase

Parse the **Implementation Phases** section of `plan.md`.

Check `$SPEC_DIR/` for existing walkthrough files (`walkthrough-phase-N.md`) to determine which phases are already completed.

Select the first incomplete phase.

### 5. Execute Phase

For the current phase:

1. **Implement**: Write code according to the phase deliverables
2. **Test**: Run relevant tests if a testing framework is configured
3. **Commit**: Commit changes with message `[Phase N] {phase name}`

### 6. Generate Walkthrough

Create `$SPEC_DIR/walkthrough-phase-{N}.md`:

```markdown
# Phase {N} Walkthrough: {Phase Name}

## Changes Made
- {file}: {what changed and why}

## How to Verify
- {step-by-step verification instructions}

## Status: PENDING REVIEW
```

### 7. Review

Present the walkthrough and ask:

```text
Phase {N} complete. Review the walkthrough above.

| Choice | Action                              |
|--------|-------------------------------------|
| ACCEPT | Mark phase done, proceed to next    |
| REJECT | Describe what needs fixing          |
```

- **ACCEPT**: Update walkthrough status to `ACCEPTED`, proceed to next phase
- **REJECT**: Fix issues based on feedback, regenerate walkthrough, re-review

### 8. Repeat or Finish

If more phases remain, go to step 4.

When all phases are complete:

```text
✓ All phases implemented and accepted.

Summary:
  Phase 1: {name} ✓
  Phase 2: {name} ✓
  ...

Next steps:
  - Review all changes: git log --oneline main..HEAD
  - Merge to main: git checkout main && git merge {BRANCH}
```

---

## STDD Mode

STDD (Spec & Test Driven Development) executes each plan phase using a 5-stage
methodology with decoupled Coder/Tester roles and Error-as-Prompt loop.

### 4-STDD. Setup

Create output directory: `$SPEC_DIR/stdd/`

Ask user for agent configuration:

```text
[STDD] Agent configuration:

| Option | Description                                    |
|--------|------------------------------------------------|
| multi  | 4 agents: spec-analyst, coder, tester, loop-driver |
| single | Single agent with role separation (default)    |

Select [single]:
```

Ask user for loop mode:

```text
[STDD] Loop mode:

| Mode   | Description                            |
|--------|----------------------------------------|
| auto   | Auto-retry until pass (max 5 loops)    |
| semi   | Confirm between loops (recommended)    |
| manual | Approve each step                      |

Select [semi]:
```

### 5-STDD. Per-Phase STDD Execution

For each phase in `plan.md`, run the 5-stage STDD cycle:

#### Stage 1: Define Spec

Extract the phase deliverables and acceptance criteria from `plan.md`.
Transform into a Zero-Ambiguity technical spec:

- **Purpose**: What this phase delivers
- **Input**: Existing code/files to work with
- **Output Schema**: Expected files, APIs, types
- **Constraints**: Technology, patterns, conventions from constitution
- **Error Cases**: Edge cases and expected behavior

All criteria must be O/X (pass/fail) verifiable. No subjective language.

Save to `$SPEC_DIR/stdd/phase-{N}-spec.md`. Ask user for approval.

#### Stage 2: Define Tests (Test-First)

Based on the confirmed spec, write test criteria **before** implementation:

- **Mechanical tests**: Syntax, lint, type check
- **Execution tests**: Unit tests, integration tests
- **Schema tests**: Output format, data types, API response shapes

Assign each test an ID (T1, T2, T3...).

Save to `$SPEC_DIR/stdd/phase-{N}-tests.md`. Ask user for confirmation.

#### Stage 3: Execute

Implement the phase deliverables following the spec and targeting all tests.

- In **multi** mode: Send [SPEC] + [TEST-CRITERIA] to coder agent
- In **single** mode: Implement in Coder mindset only (ignore test implementation details)

Commit with message `[Phase {N}] {phase name}`.

#### Stage 4: Test & Refactor Loop

Run all tests from Stage 2 against the implementation.

Report results in standard format:

```text
[STDD Test Report - Phase {N}, Loop #{L}]

PASSED:
- T1: {description} ✅

FAILED:
- T2: {description} ❌
  Expected: {expected}
  Actual: {actual}
  Action: {specific fix instruction}

SUMMARY: {passed}/{total} passed.
```

Loop behavior by mode:

- **All pass** → Proceed to Stage 5
- **Failures exist** → Feed error log back to coder for fix, re-test
- **Same error 2x** → Escalate to user
- **Max loops reached** → Stop and report current state

Save each loop result to `$SPEC_DIR/stdd/phase-{N}-loop-{L}.md`.

#### Stage 5: Phase Approval

Present results to user:

```text
[STDD Phase {N} Complete]

Tests: {passed}/{total} passed
Loops: {L}
Files changed: {list}

ACCEPT this phase? (Y/N)
```

- **Y**: Mark phase done, proceed to next phase
- **N**: User provides feedback → return to appropriate stage

### 6-STDD. All Phases Complete

When all phases pass STDD:

```text
✓ All phases implemented via STDD.

Summary:
  Phase 1: {name} — {tests} tests, {loops} loops ✓
  Phase 2: {name} — {tests} tests, {loops} loops ✓
  ...

Total: {total_tests} tests passed, {total_loops} loops

Next steps:
  - Review all changes: git log --oneline main..HEAD
  - Merge to main: git checkout main && git merge {BRANCH}
```
