---
name: Specification Contradiction Checker
description: Analyze specification files for contradictions, inconsistencies, and ambiguities
category: Documentation
tags: [specification, documentation, analysis, quality, consistency]
---

<!-- SPEC-CHECK:START -->

# Specification Contradiction Checker Skill

## When to Use This Skill

This skill automatically activates when:
- User asks to review specifications
- User mentions "spec", "specification", or "requirements"
- Working with files in `specs/` directory
- User asks about inconsistencies or contradictions
- Reviewing documentation for accuracy

## Analysis Process

### Step 1: Identify Specification Files

Find all specification files:

```bash
# Default location
find specs/ -name "*.md" -type f

# Also check common locations
find docs/ -name "*spec*.md" -type f
find requirements/ -name "*.md" -type f
```

### Step 2: Read All Specifications (PARALLEL)

**CRITICAL**: Read ALL specification files completely before analysis.

**PARALLELIZATION OPPORTUNITY**: When there are multiple spec files, read them in parallel using multiple subagents:

```
Use the Task tool with multiple parallel invocations:

Task 1: Read and extract from specs/auth.md
Task 2: Read and extract from specs/api.md
Task 3: Read and extract from specs/data-model.md
...

Each subagent extracts:
- Key statements and requirements
- Defined terms and concepts
- Cross-references to other specs
- Numerical values and thresholds
```

**Example parallel invocation:**
```
Launch 3+ subagents simultaneously, each with:
- subagent_type: "general-purpose"
- prompt: "Read [spec_file] and extract: 1) All requirements 2) Defined terms 3) Numerical values 4) Cross-references. Return structured JSON."
```

For each file:
1. Parse the document structure
2. Extract key statements and requirements
3. Note defined terms and concepts
4. Identify cross-references

### Step 3: Systematic Analysis

Analyze for the following issue types:

## Issue Categories

### 1. Direct Contradictions

Statements that directly conflict with each other.

**Examples:**
```markdown
# File: specs/auth.md
"Users must re-authenticate every 24 hours"

# File: specs/security.md
"Session tokens never expire for premium users"
```

**Check for:**
- [ ] Conflicting numerical values
- [ ] Opposite boolean conditions
- [ ] Mutually exclusive requirements
- [ ] Incompatible state transitions

### 2. Inconsistencies

Similar features described differently across specs.

**Examples:**
```markdown
# File: specs/api.md
"API responses use camelCase"

# File: specs/data-model.md
"All fields use snake_case"
```

**Check for:**
- [ ] Naming convention differences
- [ ] Different default values
- [ ] Inconsistent terminology
- [ ] Varying data types for same concept

### 3. Ambiguities

Unclear or vague requirements that could be interpreted multiple ways.

**Examples:**
```markdown
"The system should respond quickly"  # How quickly?
"Large files are handled differently"  # What size is "large"?
"Users have appropriate access"  # What determines "appropriate"?
```

**Check for:**
- [ ] Undefined thresholds
- [ ] Vague qualifiers (quickly, many, some)
- [ ] Missing units
- [ ] Unclear ownership/responsibility

### 4. Missing Information

Critical details that should be specified but aren't.

**Examples:**
- Error handling not defined
- Edge cases not covered
- Integration points not specified
- Security requirements missing

**Check for:**
- [ ] Undefined error states
- [ ] Missing boundary conditions
- [ ] Unspecified defaults
- [ ] Incomplete flows

### 5. Outdated Information

Specs that may not match current implementation.

**Examples:**
- References to deprecated features
- Old API endpoints
- Removed functionality
- Changed behavior

**Check for:**
- [ ] TODO/FIXME markers
- [ ] References to "old" or "legacy"
- [ ] Dates in the past
- [ ] Dead links

## Analysis Checklist

### Parallel Analysis Strategy

When analyzing specs for contradictions, run multiple analysis tasks in parallel:

```
Use the Task tool with parallel subagents:

Task 1 (Numerical Analysis):
  - Extract all numbers, limits, thresholds
  - Compare across specs for conflicts

Task 2 (Terminology Analysis):
  - Extract all defined terms
  - Check for inconsistent usage

Task 3 (Flow Analysis):
  - Extract all state machines and flows
  - Check for missing transitions or conflicts

Task 4 (Cross-Reference Analysis):
  - Extract all references between specs
  - Verify all referenced items exist
```

This parallelization can reduce analysis time by 3-4x for large spec sets.

### Cross-Reference Analysis
- [ ] All referenced terms are defined
- [ ] All referenced files exist
- [ ] All links are valid
- [ ] Referenced sections exist

### Numerical Consistency
- [ ] Same limits across specs
- [ ] Consistent units
- [ ] Matching thresholds
- [ ] Aligned timeouts/TTLs

### Terminology Consistency
- [ ] Same concept = same name
- [ ] Glossary terms used consistently
- [ ] Acronyms defined and used correctly
- [ ] No ambiguous pronouns

### Flow Consistency
- [ ] State machines are complete
- [ ] All paths have outcomes
- [ ] Error flows are defined
- [ ] Recovery procedures exist

## Output Format

Generate a comprehensive report:

```markdown
# Specification Analysis Report

**Generated**: [YYYY-MM-DD HH:MM]
**Files Analyzed**: [count]
**Issues Found**: [count]

---

## Summary

| Category | Count | Severity |
|----------|-------|----------|
| Direct Contradictions | X | Critical |
| Inconsistencies | X | High |
| Ambiguities | X | Medium |
| Missing Information | X | Medium |
| Outdated Information | X | Low |

---

## Direct Contradictions

### [CRITICAL] Contradiction #1: [Title]

**Files**:
- `specs/file1.md` (line X)
- `specs/file2.md` (line Y)

**Statement 1**:
> [Quote from file 1]

**Statement 2**:
> [Quote from file 2]

**Impact**:
[What could go wrong if not resolved]

**Clarifying Questions**:
1. Which statement is correct?
2. Under what conditions does each apply?

---

## Inconsistencies

### [HIGH] Inconsistency #1: [Title]

**Files**:
- `specs/file1.md`
- `specs/file2.md`

**Description**:
[Detailed explanation]

**Suggested Resolution**:
[How to make consistent]

---

## Ambiguities

### [MEDIUM] Ambiguity #1: [Title]

**File**: `specs/file.md` (line X)

**Statement**:
> [Ambiguous quote]

**Possible Interpretations**:
1. [Interpretation A]
2. [Interpretation B]

**Clarifying Questions**:
1. [Question to resolve ambiguity]

---

## Missing Information

### [MEDIUM] Missing #1: [Title]

**File**: `specs/file.md`

**Context**:
[What the spec describes]

**Missing**:
[What information is needed]

**Impact**:
[Why this matters]

---

## Outdated Information

### [LOW] Outdated #1: [Title]

**File**: `specs/file.md` (line X)

**Issue**:
[What appears outdated]

**Evidence**:
[Why we think it's outdated]

---

## Recommendations

### High Priority
1. [Action item]
2. [Action item]

### Medium Priority
1. [Action item]

### Low Priority
1. [Action item]

---

## Questions for Team

These questions should be answered to resolve ambiguities:

1. [Question 1]
2. [Question 2]
3. [Question 3]
```

## Integration with Workflows

```yaml
name: Spec Check

on:
  pull_request:
    paths:
      - 'specs/**/*.md'
      - 'docs/**/*.md'

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          prompt: /check-spec-contradictions
```

## Best Practices

1. **Run regularly**: Check specs on every PR that modifies them
2. **Resolve promptly**: Address contradictions before implementation
3. **Track questions**: Create issues for unresolved ambiguities
4. **Update continuously**: Keep specs in sync with implementation
5. **Cross-reference**: Link related specs to each other

<!-- SPEC-CHECK:END -->
