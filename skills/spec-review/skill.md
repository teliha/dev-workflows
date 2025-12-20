---
name: spec-review
description: Review specifications for completeness, clarity, testability, and quality. Use with --mode quick/standard/thorough to control review depth. Default is thorough (3 consecutive passes). (user)
category: Documentation
tags: [specification, review, quality, requirements, documentation]
---

<!-- SPEC-REVIEW:START -->

# Specification Review Expert Skill

## When to Use This Skill

This skill automatically activates when:
- User finishes writing a specification
- User says "review this spec" or "check my specification"
- User asks "is this spec complete?"
- User mentions "spec review" or "specification review"
- User creates or modifies files in `specs/` directory

## Purpose

Ensure specifications are complete, clear, and implementable BEFORE development begins. Catching issues in specs is much cheaper than fixing them in code.

## Precision Modes

Use `--mode` or `-m` to control review depth:

| Mode | Passes | Categories | Use Case |
|------|--------|------------|----------|
| quick | 1 | COMPLETENESS, CLARITY only | Fast sanity check |
| standard | 2 consecutive | All 5 categories | Balanced review |
| thorough | 3 consecutive | All 5 categories | Full review (default) |

### Usage Examples

```
spec review --mode quick specs/feature/spec.md
spec review -m standard specs/feature/spec.md
spec review specs/feature/spec.md  # defaults to thorough
```

### Mode Details

**Quick Mode** (1 pass):
- Only checks COMPLETENESS and CLARITY
- No consecutive pass requirement
- Best for: Early drafts, quick sanity checks

**Standard Mode** (2 consecutive passes):
- All 5 categories checked
- Requires 2 consecutive passes without issues
- Best for: Most reviews, good balance of speed/quality

**Thorough Mode** (3 consecutive passes) - DEFAULT:
- All 5 categories checked
- Requires 3 consecutive passes without issues
- Best for: Critical specs, production-ready validation

## CRITICAL: Context Isolation for Objective Review

**Problem**: If you (Claude) helped write the specification, reviewing it in the same conversation introduces bias. You may unconsciously:
- Assume things that aren't written (because you remember the discussion)
- Overlook issues you introduced
- Be lenient on ambiguous parts you understood from context

**Solution**: Always perform reviews with a FRESH context using a subagent.

### How to Ensure Clean Context

**MANDATORY**: When reviewing a specification, spawn a NEW agent with NO prior context:

```
Use the Task tool with subagent_type="general-purpose" and a prompt that:
1. Contains ONLY the spec file path to review
2. Does NOT include conversation history
3. Asks for objective review against the checklist
```

**The reviewer should have NO memory of writing the spec.**

## Review Process

### Step 1: Identify Target

Find the specification to review:
- If path provided: Use that path
- If not provided: Ask user or detect from context

### Step 2: Parallel Subagent Analysis

Launch 5+ parallel subagents using Task tool (subagent_type: general-purpose) for context-isolated review:

```
Category 1: COMPLETENESS
- Purpose/Overview defined
- Scope (included/excluded) defined
- Functional requirements listed
- Non-functional requirements listed
- Data requirements (inputs, outputs, formats)
- Error handling specified
- Edge cases documented
- Dependencies listed
- Constraints documented
- Acceptance criteria defined

Category 2: CLARITY
- No ambiguous terms ("fast", "many", "appropriate")
- Specific values (numbers, limits, thresholds)
- Clear ownership (who/what is responsible)
- Defined terms (technical terms explained)
- No hidden assumptions

Category 3: TESTABILITY
- Measurable criteria
- Clear pass/fail conditions
- Expected outputs for given inputs
- Error conditions specified
- Edge case tests defined

Category 4: EDGE CASES & CONSISTENCY
- Zero/empty values handled
- Maximum values handled
- Boundary conditions specified
- Null/undefined cases covered
- Internal consistency (no contradictions)
- Consistent terminology

Category 5: TECHNICAL & SECURITY
- Technically feasible
- Resource requirements realistic
- Performance targets achievable
- Dependencies available
- Authentication requirements documented
- Authorization checks specified
- Data protection defined
- Input validation specified
```

Each subagent returns:
```json
{
  "category": "...",
  "issues": [
    {"severity": "error|warning", "location": "file:section", "description": "..."}
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
     - Fix ALL errors first
     - Then fix ALL warnings
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
## Spec Review Results: <spec-name>

| Category | Status | Issues |
|----------|--------|--------|
| Completeness | PASSED/FAILED | N errors, M warnings |
| Clarity | PASSED/FAILED | N errors, M warnings |
| Testability | PASSED/FAILED | N errors, M warnings |
| Edge Cases & Consistency | PASSED/FAILED | N errors, M warnings |
| Technical & Security | PASSED/FAILED | N errors, M warnings |

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
| error | Blocks implementation | Must fix before proceeding |
| warning | Could cause problems | Should fix (also auto-fixed in loop) |

## Overlooked Issues Recording

When an issue is found after a previous PASS iteration, it indicates a review oversight.
Record these in CLAUDE.md at the directory level where the issue exists.

### Recording Location

Place overlooked issues in `CLAUDE.md` files at the directory containing the problematic file.

**Example**: If issue is in `specs/feature/spec.md`
- Create/update: `specs/feature/CLAUDE.md`

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

**Category**: COMPLETENESS/CLARITY/TESTABILITY/EDGE_CASES/TECHNICAL/SECURITY
**File**: filename.md
**Missed in iteration**: N
**Found in iteration**: M
**Description**: What was missed
**Check instruction**: Specific verification step

---
```

### Subagent Integration

When reviewing, subagents should:
1. Check for CLAUDE.md in spec directories
2. Include check instructions from CLAUDE.md in verification
3. Pay extra attention to previously overlooked patterns

## Ambiguous Patterns to Flag

| Ambiguous | Clear |
|-----------|-------|
| "The system should respond quickly" | "Response time must be < 200ms" |
| "Handle large files" | "Support files up to 100MB" |
| "Users can access appropriate data" | "Users can access data they own" |
| "Should be secure" | "Must use TLS 1.3, authenticate all requests" |
| "As needed" | "When X condition is met" |

## Best Practices for Spec Authors

Based on common issues, specs should:

1. **Use RFC 2119 keywords**
   - MUST, MUST NOT, REQUIRED
   - SHOULD, SHOULD NOT, RECOMMENDED
   - MAY, OPTIONAL

2. **Quantify everything**
   - Replace "fast" with "< 200ms"
   - Replace "many" with "up to 1000"

3. **Include examples**
   - Show sample inputs and outputs
   - Provide API request/response examples

4. **Document decisions**
   - Why was this approach chosen?
   - What alternatives were considered?

5. **Consider the implementer**
   - What questions will they have?
   - What context do they need?

## Integration

This skill works well with:
- `check-spec-contradictions` - After individual review, check against other specs
- `implementation-review` - After implementation, verify against reviewed spec

## Workflow

```
Write Spec -> Spec Review (this skill) -> Fix Issues ->
  -> Check Contradictions -> [USER DECISION] -> Implement -> Implementation Review
```

## IMPORTANT: Post-Approval Behavior

**After spec is approved, DO NOT automatically start implementation.**

When the spec review passes (SPEC APPROVED):
1. Report the approval to the user
2. **STOP and wait for explicit user instruction**
3. User decides whether to:
   - Proceed with implementation
   - Make additional changes
   - Check contradictions with other specs
   - Do something else entirely

**Rationale**: The user should maintain control over when implementation begins.

## Notes

- Each review iteration uses fresh subagents (context isolation)
- Parallel execution for speed
- Both errors and warnings are auto-fixed
- **Modes**: quick (1 pass), standard (2 passes), thorough (3 passes, default)
- Any issue found resets the consecutive pass counter to 0
- **Check overlooked issues database** before each review

<!-- SPEC-REVIEW:END -->
