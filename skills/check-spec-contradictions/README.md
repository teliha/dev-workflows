# Specification Contradiction Checker Skill

## Overview

This Claude Code skill analyzes specification files for contradictions, inconsistencies, ambiguities, and missing information.

## Features

- **Contradiction Detection**: Finds directly conflicting statements
- **Inconsistency Analysis**: Identifies varying descriptions of same concepts
- **Ambiguity Flagging**: Highlights vague or unclear requirements
- **Gap Analysis**: Identifies missing critical information
- **Cross-Reference Validation**: Ensures all references are valid

## How It Works

### Automatic Activation

The skill automatically activates when:
- User asks to review specifications
- User mentions "spec", "specification", or "requirements"
- Working with files in `specs/` directory
- User asks about inconsistencies or contradictions

### Analysis Process

1. **Find Specs**: Locate all specification files
2. **Read All**: Parse complete content of each file
3. **Extract**: Identify key statements and requirements
4. **Compare**: Cross-reference across all files
5. **Report**: Generate detailed findings report

## Issue Categories

| Category | Severity | Description |
|----------|----------|-------------|
| Direct Contradictions | Critical | Conflicting statements |
| Inconsistencies | High | Same concept, different descriptions |
| Ambiguities | Medium | Vague or unclear requirements |
| Missing Information | Medium | Critical details not specified |
| Outdated Information | Low | May not match current state |

## Usage Examples

### Example 1: Full Spec Review

```bash
"Check all specifications for contradictions"
```

### Example 2: Specific Directory

```bash
"Review the specs in specs/api/ for inconsistencies"
```

### Example 3: Before Implementation

```bash
"Before I implement this feature, check if the specs are consistent"
```

## Output Format

The skill generates a comprehensive report including:

```markdown
# Specification Analysis Report

## Summary
| Category | Count |
|----------|-------|
| Contradictions | 2 |
| Inconsistencies | 3 |

## Findings
[Detailed findings with locations and quotes]

## Recommendations
[Prioritized action items]

## Questions for Team
[Clarifying questions to resolve issues]
```

## Integration with CI/CD

```yaml
name: Spec Check

on:
  pull_request:
    paths:
      - 'specs/**/*.md'

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

1. **Run on every spec change**: Catch issues early
2. **Resolve before implementation**: Don't code ambiguous specs
3. **Track questions**: Create issues for unresolved items
4. **Keep specs updated**: Sync with implementation changes

## Related Commands

- `/check-spec-contradictions` - Explicitly invoke this skill
- `/code-review` - Review code against specifications

## Limitations

- Cannot verify specs against actual implementation
- May miss domain-specific contradictions
- Requires well-structured specification files
- Best with markdown format specifications

## Contributing

To improve this skill:
1. Add new contradiction patterns to detect
2. Improve ambiguity detection
3. Add domain-specific analysis
4. Test with real specification documents
