---
name: Specification Review Expert
description: Review specifications for completeness, clarity, testability, and quality before implementation
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

**Example invocation:**
```
Task: Review specification
Prompt: |
  You are reviewing a specification file with fresh eyes.
  You have NO prior context about this spec - you are seeing it for the first time.

  Please read and review: specs/feature/spec.md

  Review against these criteria:
  - Completeness: Are all necessary sections present?
  - Clarity: Are there any ambiguous terms?
  - Testability: Can acceptance criteria be objectively verified?
  - Edge Cases: Are boundary conditions covered?
  - Consistency: Is the spec internally consistent?

  Be critical and objective. Flag anything unclear or missing.
  Generate a detailed review report.
```

### Why This Matters

| With Context | Without Context |
|--------------|-----------------|
| "I know what they meant by 'fast'" | "What is 'fast'? Not defined." |
| "We discussed error handling" | "Error handling section is missing" |
| "The edge cases are obvious" | "Edge cases not documented" |

**The reviewer should have NO memory of writing the spec.**

## Review Process

### Step 1: Read the Entire Specification

**CRITICAL**: Read the complete specification before reviewing.

Understand:
- What is being specified?
- What is the scope?
- Who is the audience?

### Step 2: Systematic Review

## Review Categories

### A. Completeness (必要な要件が全て含まれているか)

**Checklist:**
- [ ] **Purpose/Overview** - Why does this feature exist?
- [ ] **Scope** - What is included/excluded?
- [ ] **Functional Requirements** - What must the system do?
- [ ] **Non-Functional Requirements** - Performance, security, etc.
- [ ] **Data Requirements** - Inputs, outputs, formats
- [ ] **Error Handling** - What happens when things go wrong?
- [ ] **Edge Cases** - Boundary conditions and special cases
- [ ] **Dependencies** - External systems, libraries, APIs
- [ ] **Constraints** - Limitations and restrictions
- [ ] **Acceptance Criteria** - How do we know it's done?

**Questions to ask:**
- What's missing that an implementer would need to know?
- Are all user scenarios covered?
- Are failure modes documented?

### B. Clarity (曖昧な表現がないか)

**Checklist:**
- [ ] **No ambiguous terms** - "fast", "many", "appropriate", etc.
- [ ] **Specific values** - Numbers, limits, thresholds defined
- [ ] **Clear ownership** - Who/what is responsible for each action
- [ ] **Defined terms** - Technical terms explained or glossaried
- [ ] **No assumptions** - Implicit knowledge made explicit

**Ambiguous patterns to flag:**

| Ambiguous | Clear |
|-----------|-------|
| "The system should respond quickly" | "Response time must be < 200ms" |
| "Handle large files" | "Support files up to 100MB" |
| "Users can access appropriate data" | "Users can access data they own" |
| "Should be secure" | "Must use TLS 1.3, authenticate all requests" |
| "As needed" | "When X condition is met" |

**Questions to ask:**
- Could two developers interpret this differently?
- Are all "should", "may", "might" intentional or should they be "must"?

### C. Testability (受け入れ基準が明確か)

**Checklist:**
- [ ] **Measurable criteria** - Can be objectively verified
- [ ] **Test scenarios** - Clear pass/fail conditions
- [ ] **Expected outputs** - For given inputs, what's expected?
- [ ] **Error conditions** - What errors should occur when?
- [ ] **Edge case tests** - Boundary conditions specified

**Template for good acceptance criteria:**
```markdown
## Acceptance Criteria

### Scenario: [Name]
**Given** [precondition]
**When** [action]
**Then** [expected result]

### Test Cases
| Input | Expected Output | Notes |
|-------|-----------------|-------|
| [value] | [result] | [edge case] |
```

**Questions to ask:**
- How would QA test this?
- Can we write automated tests from this spec?

### D. Edge Cases (境界条件が考慮されているか)

**Common edge cases to check:**

| Category | Examples |
|----------|----------|
| Empty/Zero | Empty string, zero amount, null value |
| Boundaries | Min value, max value, exactly at limit |
| Invalid | Wrong type, negative when positive expected |
| Timing | Concurrent requests, race conditions |
| State | First time, already exists, deleted |
| Permissions | No access, partial access, admin |
| Resources | Out of memory, disk full, timeout |

**Questions to ask:**
- What happens at the boundaries?
- What if the input is invalid?
- What if something fails midway?

### E. Internal Consistency (内部で矛盾がないか)

**Checklist:**
- [ ] **No conflicting requirements** - A doesn't contradict B
- [ ] **Consistent terminology** - Same term = same meaning
- [ ] **Aligned numbers** - Values match throughout
- [ ] **Coherent flow** - Steps make logical sense

**Questions to ask:**
- Does section A align with section B?
- Are all referenced items defined?

### F. Technical Feasibility (技術的に実現可能か)

**Checklist:**
- [ ] **Technically possible** - Can be implemented with available tech
- [ ] **Resource reasonable** - Memory, CPU, storage realistic
- [ ] **Time realistic** - Performance requirements achievable
- [ ] **Dependencies available** - Required services/APIs exist
- [ ] **Security achievable** - Security requirements implementable

**Questions to ask:**
- Is this physically possible?
- Are there any impossible requirements?
- Are performance targets realistic?

### G. Security Considerations (セキュリティ)

**Checklist:**
- [ ] **Authentication** - How are users identified?
- [ ] **Authorization** - Who can do what?
- [ ] **Data protection** - Sensitive data handling?
- [ ] **Input validation** - How is input sanitized?
- [ ] **Audit logging** - What actions are logged?

### H. Structure and Format (構造とフォーマット)

**Checklist:**
- [ ] **Logical organization** - Easy to navigate
- [ ] **Consistent formatting** - Headers, lists, tables
- [ ] **Version/date** - When was this written?
- [ ] **Author/owner** - Who is responsible?
- [ ] **Status** - Draft, review, approved?

## Output Format

### Review Report

```markdown
# Specification Review Report

**Spec**: `[file path]`
**Date**: [YYYY-MM-DD HH:MM]
**Status**: [NEEDS WORK / APPROVED WITH NOTES / APPROVED]

---

## Summary

| Category | Status | Issues |
|----------|--------|--------|
| Completeness | ✅/⚠️/❌ | [count] |
| Clarity | ✅/⚠️/❌ | [count] |
| Testability | ✅/⚠️/❌ | [count] |
| Edge Cases | ✅/⚠️/❌ | [count] |
| Consistency | ✅/⚠️/❌ | [count] |
| Feasibility | ✅/⚠️/❌ | [count] |
| Security | ✅/⚠️/❌ | [count] |
| Structure | ✅/⚠️/❌ | [count] |

**Overall**: [X] issues found, [Y] critical

---

## Critical Issues (Must Fix)

### Issue #1: [Title]

**Category**: [Completeness/Clarity/etc.]
**Location**: [Section/Line]

**Problem**:
[Description of the issue]

**Example**:
> [Quote from spec]

**Recommendation**:
[How to fix it]

---

## Warnings (Should Fix)

### Warning #1: [Title]

**Category**: [Category]
**Location**: [Section]

**Problem**: [Description]
**Recommendation**: [Fix]

---

## Suggestions (Nice to Have)

### Suggestion #1: [Title]

[Description and recommendation]

---

## Questions for Clarification

These need answers before implementation:

1. [Question 1]
2. [Question 2]

---

## Positive Findings

What the spec does well:
- [Good point 1]
- [Good point 2]

---

## Checklist Summary

### Completeness
- [x] Purpose defined
- [x] Scope defined
- [ ] Edge cases documented ⚠️
- [ ] Error handling specified ❌

[Continue for each category...]

---

## Recommendation

**Status**: [NEEDS WORK / APPROVED WITH NOTES / APPROVED]

**Next Steps**:
1. [Action 1]
2. [Action 2]
```

## Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| ❌ Critical | Blocks implementation | Must fix before proceeding |
| ⚠️ Warning | Could cause problems | Should fix |
| 💡 Suggestion | Improvement opportunity | Nice to have |
| ✅ Pass | No issues | Good to go |

## Example Review Findings

### Critical: Missing Error Handling

```markdown
### Issue #1: No error handling specified for API failures

**Category**: Completeness
**Location**: Section 3.2 - API Integration

**Problem**:
The spec describes calling the payment API but doesn't specify what happens if the API:
- Returns an error
- Times out
- Is unavailable

**Quote**:
> "The system calls the payment API with the transaction details"

**Recommendation**:
Add error handling section:
- Define retry policy (e.g., 3 retries with exponential backoff)
- Specify error messages to show users
- Define fallback behavior if API is down
```

### Warning: Ambiguous Requirement

```markdown
### Warning #1: "Large files" not defined

**Category**: Clarity
**Location**: Section 2.1 - File Upload

**Problem**:
"Large files should be handled differently" - what is "large"?

**Quote**:
> "Large files should be processed asynchronously"

**Recommendation**:
Define threshold: "Files larger than 10MB should be processed asynchronously"
```

### Suggestion: Add Acceptance Criteria

```markdown
### Suggestion #1: Add test scenarios

**Category**: Testability
**Location**: Section 4 - Requirements

**Problem**:
Requirements are clear but no acceptance criteria provided.

**Recommendation**:
Add "Acceptance Criteria" section with Given/When/Then scenarios.
```

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
Write Spec → Spec Review (this skill) → Fix Issues →
  → Check Contradictions → Implement → Implementation Review
```

<!-- SPEC-REVIEW:END -->
