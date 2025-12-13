---
name: Implementation Review Loop
description: Self-feedback review loop to verify implementation matches specification
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

**Example invocation:**
```
Task: Review implementation against spec
Prompt: |
  You are reviewing an implementation with completely fresh eyes.
  You have NO prior context - you are seeing this code for the first time.

  SPECIFICATION: specs/feature/spec.md
  IMPLEMENTATION FILES:
  - src/feature/index.ts
  - src/feature/utils.ts

  Your task:
  1. Read the specification completely
  2. Read all implementation files
  3. Compare implementation against EVERY requirement in the spec
  4. Flag ANY gap, missing feature, or deviation
  5. Be critical and thorough - assume nothing

  Generate a detailed gap report with:
  - Each requirement and its implementation status
  - Missing or incomplete implementations
  - Deviations from spec
  - Specific fixes needed
```

### Why This Matters

| Reviewer with Context | Reviewer without Context |
|----------------------|--------------------------|
| "I handled that edge case" | "Edge case X is not handled in code" |
| "The error handling is there" | "No try/catch around API call" |
| "That validation is implicit" | "Input validation is missing" |

**The reviewer must have ZERO memory of writing the code.**

### Review Loop with Context Isolation

```
┌─────────────────────────────────────────────────────┐
│              CONTEXT-ISOLATED REVIEW LOOP            │
│                                                      │
│  ┌──────────┐    ┌──────────────┐    ┌──────────┐  │
│  │  Spec +  │───▶│  SUBAGENT    │───▶│  Gaps?   │  │
│  │  Impl    │    │ (Fresh eyes) │    │          │  │
│  └──────────┘    └──────────────┘    └────┬─────┘  │
│                                           │         │
│                             ┌─────────────┴───┐     │
│                             ▼                 ▼     │
│                        ┌────────┐       ┌────────┐  │
│                        │  YES   │       │   NO   │  │
│                        └───┬────┘       │ (Done) │  │
│                            │            └────────┘  │
│                            ▼                        │
│                       ┌─────────┐                   │
│                       │  Apply  │                   │
│                       │  Fixes  │                   │
│                       └────┬────┘                   │
│                            │                        │
│                            ▼                        │
│                   ┌────────────────┐                │
│                   │ NEW SUBAGENT   │────────┐       │
│                   │ (Fresh review) │        │       │
│                   └────────────────┘        │       │
│                            ▲                │       │
│                            └────────────────┘       │
│                         (Each review = new agent)   │
└─────────────────────────────────────────────────────┘
```

**Each iteration of the review loop should use a NEW subagent.**

## Review Loop Process

### Overview

```
┌─────────────────────────────────────────────────┐
│                 REVIEW LOOP                      │
│                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │  Spec    │───▶│  Check   │───▶│  Gaps?   │  │
│  │  File    │    │  Impl    │    │          │  │
│  └──────────┘    └──────────┘    └────┬─────┘  │
│                                       │         │
│                         ┌─────────────┴───┐     │
│                         ▼                 ▼     │
│                    ┌────────┐       ┌────────┐  │
│                    │  YES   │       │   NO   │  │
│                    │ (Fix)  │       │ (Done) │  │
│                    └───┬────┘       └────────┘  │
│                        │                        │
│                        ▼                        │
│                   ┌─────────┐                   │
│                   │  Apply  │                   │
│                   │  Fixes  │──────────┐        │
│                   └─────────┘          │        │
│                        ▲               │        │
│                        └───────────────┘        │
│                      (Loop until done)          │
└─────────────────────────────────────────────────┘
```

### Step 1: Locate Specification

Find the specification that was used for implementation:

```bash
# Common spec locations
find specs/ -name "*.md" -type f
find docs/ -name "*spec*.md" -type f
find requirements/ -name "*.md" -type f
```

**Ask if unclear**: "Which specification file should I review against?"

### Step 2: Read Specification Completely

**CRITICAL**: Read the ENTIRE specification before reviewing.

Extract from the spec:
1. **Functional Requirements** - What the code must do
2. **Technical Requirements** - How it must be implemented
3. **Constraints** - Limitations and boundaries
4. **Edge Cases** - Special conditions to handle
5. **Error Handling** - Expected error behaviors
6. **Performance Requirements** - Speed, memory, gas limits
7. **Security Requirements** - Access control, validation

### Step 3: Identify Implementation Files

Find all files that implement the specification:

```bash
# Find implementation files
find src/ -name "*.ts" -o -name "*.sol" -o -name "*.tsx"

# Or based on spec mentions
grep -r "FeatureName" src/
```

### Step 4: Systematic Review

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
| [Requirement 1] | ✅/❌/⚠️ | `file.ts:line` | [Notes] |
| [Requirement 2] | ✅/❌/⚠️ | `file.ts:line` | [Notes] |
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

### Step 5: Generate Gap Report

After reviewing, generate a detailed gap report:

```markdown
# Implementation Review Report

**Spec**: `[spec file path]`
**Implementation**: `[impl file paths]`
**Date**: [YYYY-MM-DD HH:MM]
**Review Iteration**: [N]

---

## Summary

| Category | Status | Issues |
|----------|--------|--------|
| Functional Completeness | ✅/⚠️/❌ | [count] |
| Edge Cases | ✅/⚠️/❌ | [count] |
| Error Handling | ✅/⚠️/❌ | [count] |
| Technical Accuracy | ✅/⚠️/❌ | [count] |
| Security | ✅/⚠️/❌ | [count] |
| Performance | ✅/⚠️/❌ | [count] |

**Overall Status**: [PASS / NEEDS FIXES]

---

## Gaps Found

### Gap #1: [Title]

**Category**: [Functional/Edge Case/Error/Technical/Security/Performance]
**Severity**: [Critical/High/Medium/Low]

**Spec Says**:
> [Quote from specification]

**Implementation Has**:
> [Current implementation or "Missing"]

**Location**: `[file:line]`

**Required Fix**:
[Specific steps to fix]

---

[Repeat for each gap]

---

## Verification Checklist

After fixes, verify:
- [ ] All gaps addressed
- [ ] No regressions introduced
- [ ] Tests updated if needed
- [ ] Tests pass
```

### Step 6: Apply Fixes

For each gap found:

1. **Understand the gap** - Why does implementation differ from spec?
2. **Plan the fix** - What exactly needs to change?
3. **Apply minimal fix** - Change only what's needed
4. **Verify fix** - Does it now match spec?

### Step 7: Loop Until Complete

**CRITICAL**: After applying fixes, RESTART the review from Step 4.

Continue looping until:
- All functional requirements are met
- All edge cases are handled
- All error handling is correct
- No gaps remain

### Step 8: Final Verification

When all gaps are closed:

```markdown
## Final Review Summary

**Iterations Required**: [N]
**Total Gaps Found**: [count]
**All Gaps Resolved**: ✅

### Changes Made
1. [Change 1]
2. [Change 2]

### Tests
- [ ] All existing tests pass
- [ ] New tests added for gaps
- [ ] Coverage adequate

### Confidence Level
[High/Medium/Low] - [Explanation]

### Ready for
- [ ] Code review
- [ ] Merge
- [ ] Deployment
```

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
- ✅ All spec requirements implemented
- ✅ All edge cases handled
- ✅ All tests pass
- ✅ No known gaps remain

Don't stop if:
- ❌ "Good enough" but gaps remain
- ❌ Tired of iterating
- ❌ Minor issues seem unimportant

## Integration with Workflow

### After `/implement` or manual implementation

```
User: "I've finished implementing the vault spec"

Skill activates:
1. Finds vault spec
2. Reviews implementation
3. Reports gaps
4. Applies fixes
5. Loops until complete
6. Reports final status
```

### Trigger phrases

- "Review my implementation against the spec"
- "Check if implementation matches spec"
- "Run implementation review loop"
- "Verify implementation completeness"
- "Self-review the implementation"

## Output Examples

### Gap Found Example

```markdown
### Gap #1: Missing Error Handler

**Category**: Error Handling
**Severity**: High

**Spec Says**:
> "Return InvalidAmount error when deposit amount is zero"

**Implementation Has**:
No check for zero amount in deposit function

**Location**: `src/Vault.sol:45`

**Required Fix**:
Add require statement:
```solidity
require(amount > 0, InvalidAmount());
```
```

### All Clear Example

```markdown
## Implementation Review Complete

**Status**: ✅ ALL REQUIREMENTS MET

- Functional: 12/12 requirements ✅
- Edge Cases: 8/8 handled ✅
- Errors: 5/5 implemented ✅
- Security: 3/3 checks ✅

**Iterations**: 3
**Total Fixes**: 7

Ready for code review and merge.
```

<!-- IMPL-REVIEW:END -->
