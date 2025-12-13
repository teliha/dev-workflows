---
description: Analyze specification files for contradictions and inconsistencies
---

# Check Spec Contradictions Command

Analyze specification files for contradictions, inconsistencies, and ambiguities.

This command uses the [Specification Contradiction Checker skill](../skills/check-spec-contradictions/skill.md) for thorough analysis.

## Usage

```bash
/check-spec-contradictions
```

## What It Does

The skill analyzes all specification files and identifies:

| Issue Type | Description |
|------------|-------------|
| Direct Contradictions | Statements that conflict with each other |
| Inconsistencies | Same concept described differently |
| Ambiguities | Vague requirements open to interpretation |
| Missing Information | Critical details not specified |
| Outdated Information | Specs that may not match implementation |

## Output

For each finding, the report includes:
- Location (file path and section)
- Description of the issue
- Potential impact
- Clarifying questions to resolve

## Example Report

```markdown
## Direct Contradictions

### [CRITICAL] Session Expiry Conflict
**Files**: specs/auth.md, specs/security.md

Statement 1: "Sessions expire after 24 hours"
Statement 2: "Premium users have non-expiring sessions"

**Impact**: Unclear security policy
**Questions**: Which statement is authoritative?
```

## Integration with CI/CD

```yaml
name: Spec Check

on:
  pull_request:
    paths:
      - 'specs/**/*.md'
      - 'docs/**/*.md'

jobs:
  check:
    uses: teliha/dev-workflows/.github/workflows/spec-check.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Best Practices

1. **Run on every spec change** - Catch issues early
2. **Resolve before implementation** - Don't code ambiguous specs
3. **Track questions** - Create issues for unresolved items
4. **Keep specs updated** - Sync with implementation

## Related

- [Specification Contradiction Checker Skill](../skills/check-spec-contradictions/skill.md) - Detailed analysis process
