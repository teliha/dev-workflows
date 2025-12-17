---
name: generate-spec
description: Generate a specification file from user input/requirements. Use when user says "generate spec", "create spec", "write spec", "spec from requirements", or provides feature requirements and asks to create a spec. (user)
category: Specification
tags: [spec, generation, requirements, design]
---

# Generate Spec Skill

## Purpose

Transform user input (requirements, ideas, feature requests) into a structured specification file.

## Activation

This skill activates when:
- User says "generate spec", "create spec", "write spec"
- User provides requirements and asks to create a specification
- User says "spec from [input]" or "turn this into a spec"

## Input

Accept various forms of input:
- Bullet points of requirements
- Natural language description
- User stories
- Conversation about a feature
- Rough notes or ideas

### Reference Files (Context)

Use `--context` or `-c` to specify files to read as background context:

```
generate spec -c docs/requirements.md from: ユーザー認証
generate spec -c design/*.md -c notes.txt from: 決済機能
generate spec --context existing-api.yaml from: API拡張
```

The skill will:
1. Read specified files first
2. Use their content as context for spec generation
3. Align new spec with existing conventions/patterns

## Output

Generate `specs/[feature-name]/spec.md` with structured content.

## Generation Process

### Step 1: Understand Input

Extract from user input:
- Core purpose/goal
- Target users
- Key features
- Constraints mentioned
- Any technical preferences

### Step 2: Ask Clarifying Questions (if needed)

If input is too vague, ask about:
- Target users
- Main use cases
- Technical constraints
- Integration requirements

### Step 3: Generate Spec

Create a structured spec file following this template:

```markdown
# [Feature Name] Specification

## Overview
[Brief description of the feature]

## Purpose
[What problem this solves, business value]

## Scope

### In Scope
- [Feature 1]
- [Feature 2]

### Out of Scope
- [Excluded item 1]
- [Excluded item 2]

## Functional Requirements

### 1. [Feature/Endpoint Name]

**Endpoint**: `METHOD /api/path`

**Request**:
```json
{
  "field": "type (constraints, required/optional)"
}
```

**Response (2xx)**:
```json
{
  "data": {},
  "meta": {}
}
```

**Validation**:
- [Rule 1]
- [Rule 2]

## Non-Functional Requirements

- Performance: [requirements]
- Availability: [requirements]
- Security: [requirements]

## Error Codes

| HTTP | Code | Condition |
|------|------|-----------|
| 400 | ERROR_CODE | [condition] |

## Edge Cases

| Input | Expected |
|-------|----------|
| [case] | [behavior] |

## Acceptance Criteria

### AC1: [Scenario Name]
**Given** [precondition]
**When** [action]
**Then** [expected result]
```

### Step 4: Write File

1. Create directory `specs/[feature-name]/`
2. Write `spec.md`
3. Report completion

## Precision Levels

The `--precision` or `-p` flag controls spec detail level:

| Level | Detail |
|-------|--------|
| 10-30% | Overview, scope, basic requirements only |
| 40-60% | + API design, validation rules |
| 70-80% | + Edge cases, error codes |
| 90-100% | + Detailed acceptance criteria, test scenarios |

**Default**: 70%

### Usage Examples

```
generate spec from: ユーザー認証機能が欲しい
generate spec -p 30 from: 簡単な TODO アプリ
generate spec --precision 100 from: [detailed requirements]
```

### Precision 30% Output (Quick)

```markdown
# Feature Specification

## Overview
[1-2 sentences]

## Scope
### In Scope
- [3-5 bullet points]

### Out of Scope
- [2-3 bullet points]

## Key Requirements
- [5-10 bullet points of main features]
```

### Precision 70% Output (Standard)

Full template with:
- Overview, Purpose, Scope
- Functional Requirements with API design
- Basic validation rules
- Key error codes
- Main edge cases

### Precision 100% Output (Thorough)

Full template plus:
- Detailed validation for every field
- Complete error code table
- Comprehensive edge cases
- Multiple acceptance criteria per feature
- Test scenarios
- Data model details
- Security considerations
- Performance requirements

## Notes

- Spec is written in English (technical standard)
- Use `generate-spec-docs` skill after to create Japanese documentation
- Generated specs can be refined with `spec-review` skill
