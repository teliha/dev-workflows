---
description: Review pull request code for quality, security, and best practices
---

# Code Review Command

Perform a comprehensive code review of the current pull request and provide constructive feedback.

This command uses the [Code Review Expert skill](../skills/code-review/skill.md) for detailed analysis.

## Usage

```bash
/code-review
```

## What It Does

The code review skill automatically:

1. **Analyzes the PR** - Gets diff and changed files
2. **Reviews systematically** across:
   - Security vulnerabilities
   - Code quality and maintainability
   - Performance considerations
   - Test coverage
   - Architecture and design
3. **Posts feedback** as a PR comment

## Review Categories

| Category | What's Checked |
|----------|---------------|
| Security | Vulnerabilities, access control, input validation |
| Quality | Best practices, error handling, naming |
| Performance | Gas optimization, efficiency, caching |
| Testing | Coverage, edge cases, error conditions |
| Architecture | Patterns, organization, separation of concerns |

## Output Format

Reviews are posted as PR comments with:
- Summary of findings
- Critical issues (must fix)
- Suggestions (should consider)
- Positive findings (well-done areas)
- Recommendation (approve/request changes/comment)

## Project-Specific Context

### Foundry/Solidity
- Checks Foundry test patterns
- Verifies `forge fmt` compliance
- Reviews EVK/EVC integration patterns

### Next.js/TypeScript
- TypeScript type safety
- React best practices
- API route security

## Integration with CI/CD

```yaml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    uses: teliha/dev-workflows/.github/workflows/code-review.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Related

- [Code Review Expert Skill](../skills/code-review/skill.md) - Detailed review process
- [Audit Command](./audit.md) - Deep security audit
