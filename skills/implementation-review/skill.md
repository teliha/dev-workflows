---
name: implementation-review
description: Self-feedback review loop to verify implementation matches specification. Use with --mode quick/standard/thorough to control review depth. Default is thorough (3 consecutive passes). (user)
category: Quality
tags: [review, spec, implementation, feedback, quality-assurance, verification]
---

<!-- IMPL-REVIEW:START -->

# Implementation Review Loop Skill

## When to Use This Skill

This skill automatically activates when:
- User says "implementation is done" or "finished implementing"
- User asks to "verify implementation against spec"
- User mentions "review my implementation"
- After completing implementation from a specification
- User asks for "self-review" or "feedback loop"

## Purpose

Ensure that implementations fully and correctly match their specifications through systematic self-review and iterative improvement.

## Precision Modes

Use `--mode` or `-m` to control review depth:

| Mode | Passes | Categories | Use Case |
|------|--------|------------|----------|
| quick | 1 | SPEC_COMPLIANCE only | Fast sanity check |
| standard | 2 consecutive | All 4 categories | Balanced review |
| thorough | 3 consecutive | All 4 categories | Full review (default) |

### Usage Examples

```
implementation review --mode quick
implementation review -m standard src/feature/
implementation review  # defaults to thorough
```

### Mode Details

**Quick Mode** (1 pass):
- Only checks SPEC_COMPLIANCE
- No consecutive pass requirement
- Best for: Quick verification, WIP reviews

**Standard Mode** (2 consecutive passes):
- All 4 categories checked
- Requires 2 consecutive passes without issues
- Best for: Most implementations, good balance of speed/quality

**Thorough Mode** (3 consecutive passes) - DEFAULT:
- All 4 categories checked
- Requires 3 consecutive passes without issues
- Best for: Critical code, production-ready validation

## CRITICAL: Context Isolation for Objective Review

**Problem**: If you (Claude) wrote the implementation, reviewing it in the same conversation introduces severe bias:
- You remember WHY you wrote it that way (but the code might not reflect it)
- You unconsciously fill in gaps from memory
- You're lenient on your own code
- You miss bugs you introduced

**Solution**: Always perform reviews with a FRESH context using a subagent.

### How to Ensure Clean Context

**MANDATORY**: When reviewing implementation against spec, spawn a NEW agent with NO prior context:

```
Use the Task tool with subagent_type="general-purpose" and a prompt that:
1. Contains ONLY the spec file path and implementation file paths
2. Does NOT include any conversation history
3. Does NOT mention what was discussed or decided
4. Asks for objective comparison against the spec
```

**The reviewer must have ZERO memory of writing the code.**

## Review Process

### Step 1: Identify Target

Find the specification and implementation:
- Specification: Ask user or detect from context
- Implementation files: Based on spec references or project structure

### Step 2: Parallel Subagent Analysis

Launch parallel subagents using Task tool (subagent_type: general-purpose) for context-isolated review:

```
Category 1: SPEC COMPLIANCE
- All requirements implemented as specified
- Response formats match spec exactly
- Error codes and messages match spec
- Edge cases handled as specified

Category 2: CODE QUALITY
- No hardcoded values that should be configurable
- Proper error handling
- Type safety (no `any` types in TypeScript)
- Following project patterns

Category 3: TASK COMPLETION
- All tasks/requirements are completed
- No partial implementations
- Tests written for all scenarios

Category 4: SECURITY
- Authentication applied where required
- Input validation implemented
- No sensitive data exposure
- Authorization checks in place
```

Each subagent returns:
```json
{
  "category": "...",
  "issues": [
    {"severity": "error|warning", "location": "file:line", "description": "...", "fix": "..."}
  ],
  "passed": true/false
}
```

### Step 3: Self-Correcting Loop

**IMPORTANT**: Fix ALL issues including warnings, not just errors.

```
# Passes required based on mode:
# - quick: 1 pass (no consecutive requirement)
# - standard: 2 consecutive passes
# - thorough: 3 consecutive passes (default)

consecutive_passes = 0
iteration = 0
REQUIRED_PASSES = mode_to_passes(mode)  # 1, 2, or 3

WHILE consecutive_passes < REQUIRED_PASSES:
  iteration++
  1. Aggregate results from all subagents
  2. If issues found (errors OR warnings):
     - consecutive_passes = 0  # Reset counter
     - Apply fixes to code
     - Update tests if needed
  3. If no issues (PASS):
     - consecutive_passes++
  4. Re-run parallel review with fresh subagents

```

**Termination Conditions by Mode:**
- **Quick**: 1 pass with no issues (fast, minimal validation)
- **Standard**: 2 consecutive passes (balanced)
- **Thorough**: 3 consecutive passes (comprehensive, default)

### Step 4: Final Report

Output structured summary:

```markdown
## Implementation Review Results: <feature-name>

| Category | Status | Issues |
|----------|--------|--------|
| Spec Compliance | PASSED/FAILED | N errors, M warnings |
| Code Quality | PASSED/FAILED | N errors, M warnings |
| Task Completion | PASSED/FAILED | N errors, M warnings |
| Security | PASSED/FAILED | N errors, M warnings |

### Iterations Summary
| Iteration | Issues Found | Consecutive Passes |
|-----------|--------------|-------------------|
| 1 | 5 errors, 3 warnings | 0 |
| 2 | 0 | 1 |
| 3 | 0 | 2 |
| 4 | 0 | 3 |

### Fixed Issues (per iteration)
- Iteration 1: [list of fixes]

### Final Status: PASSED (3 consecutive)
Review stability confirmed with 3 consecutive passes.
```

## Severity Definitions

| Level | Meaning | Action |
|-------|---------|--------|
| error | Must fix before commit | Blocks merge |
| warning | Should fix for quality | Also fixed in feedback loop |

## Review Checklist

### A. Functional Completeness

For each requirement in the spec:

- [ ] Is the requirement implemented?
- [ ] Does implementation match spec exactly?
- [ ] Are all specified behaviors present?
- [ ] Are optional features handled correctly?

**Template:**
```markdown
| Spec Requirement | Status | Implementation Location | Notes |
|-----------------|--------|------------------------|-------|
| [Requirement 1] | PASS/FAIL/PARTIAL | `file.ts:line` | [Notes] |
| [Requirement 2] | PASS/FAIL/PARTIAL | `file.ts:line` | [Notes] |
```

### B. Edge Cases

- [ ] Zero/empty values handled?
- [ ] Maximum values handled?
- [ ] Boundary conditions tested?
- [ ] Null/undefined cases covered?
- [ ] Invalid input rejected properly?

### C. Error Handling

- [ ] All error cases from spec implemented?
- [ ] Error messages match spec?
- [ ] Recovery behavior correct?
- [ ] Errors properly propagated?

### D. Technical Accuracy

- [ ] Data types match spec?
- [ ] Function signatures match spec?
- [ ] Return values match spec?
- [ ] API contracts fulfilled?

### E. Security (if applicable)

- [ ] Access control implemented per spec?
- [ ] Input validation present?
- [ ] Authorization checks in place?
- [ ] No security gaps?

### F. Performance (if specified)

- [ ] Performance requirements met?
- [ ] No obvious inefficiencies?
- [ ] Resource limits respected?

## Overlooked Issues Recording

When an issue is found after a previous PASS iteration, it indicates a review oversight.
Record these in CLAUDE.md at the directory level where the issue exists.

### Recording Location

Place overlooked issues in `CLAUDE.md` files at the directory containing the problematic file.

**Example**: If issue is in `src/features/auth/login.ts`
- Create/update: `src/features/auth/CLAUDE.md`

### Recording Process

When oversight occurs (PASS then FAIL):
1. Identify the directory containing the file with the issue
2. Create or update `CLAUDE.md` in that directory
3. Add the oversight pattern with check instruction
4. Future reviews will read these CLAUDE.md files

### CLAUDE.md Format

```markdown
# Overlooked Issues for This Directory

## [Issue Pattern Name]

**Category**: SPEC_COMPLIANCE/CODE_QUALITY/TASK_COMPLETION/SECURITY
**File**: filename.ts
**Missed in iteration**: N
**Found in iteration**: M
**Description**: What was missed
**Check instruction**: Specific verification step

---
```

### Subagent Integration

When reviewing, subagents should:
1. Check for CLAUDE.md in implementation directories
2. Include check instructions from CLAUDE.md in verification
3. Pay extra attention to previously overlooked patterns

## Review Categories by Project Type

### Solidity/Smart Contracts

Additional checks:
- [ ] Events emitted per spec
- [ ] Modifiers applied correctly
- [ ] State changes in correct order
- [ ] Gas considerations met
- [ ] Upgrade patterns followed (if applicable)

### TypeScript/JavaScript

Additional checks:
- [ ] Types match spec interfaces
- [ ] Async/await properly handled
- [ ] Error types correct
- [ ] API response formats match

### React Components

Additional checks:
- [ ] Props match spec interface
- [ ] State management correct
- [ ] Lifecycle handled properly
- [ ] Accessibility requirements met

## Best Practices

### During Review

1. **Be thorough** - Check every requirement
2. **Be objective** - Don't assume correctness
3. **Be precise** - Note exact locations
4. **Be constructive** - Provide clear fixes

### During Fixes

1. **Fix one thing at a time** - Avoid scope creep
2. **Verify immediately** - Don't batch fixes
3. **Minimal changes** - Don't refactor unrelated code
4. **Document decisions** - Note why changes were made

### Knowing When to Stop

Stop the loop when:
- All spec requirements implemented
- All edge cases handled
- All tests pass
- No known gaps remain
- 3 consecutive PASS results achieved

Don't stop if:
- "Good enough" but gaps remain
- Tired of iterating
- Minor issues seem unimportant

## Integration with Workflow

### After implementation

```
User: "I've finished implementing the feature"

Skill activates:
1. Finds related spec
2. Reviews implementation
3. Reports gaps
4. Applies fixes
5. Loops until complete (3 consecutive passes)
6. Reports final status
```

### Trigger phrases

- "Review my implementation against the spec"
- "Check if implementation matches spec"
- "Run implementation review loop"
- "Verify implementation completeness"
- "Self-review the implementation"

## IMPORTANT: Post-Approval Behavior

**After implementation review passes, DO NOT automatically proceed to next steps.**

When the implementation review passes (ALL REQUIREMENTS MET):
1. Report the approval to the user
2. **STOP and wait for explicit user instruction**
3. User decides whether to:
   - Proceed with code review
   - Create a pull request
   - Merge to main branch
   - Run additional tests
   - Deploy to staging/production
   - Do something else entirely

**Rationale**: The user should maintain control over when to proceed.

## Notes

- Each review iteration uses fresh subagents (context isolation)
- Parallel execution for speed
- Both errors and warnings are auto-fixed
- **Modes**: quick (1 pass), standard (2 passes), thorough (3 passes, default)
- Any issue found resets the consecutive pass counter to 0
- Compares implementation against spec, not just code quality
- **Check overlooked issues database** before each review

<!-- IMPL-REVIEW:END -->
