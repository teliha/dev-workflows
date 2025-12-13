# Code Review Expert Skill

## Overview

This Claude Code skill provides automated comprehensive code review for pull requests, analyzing quality, security, and best practices across different project types.

## Features

- **Security Analysis**: Identifies vulnerabilities, access control issues, and security risks
- **Code Quality**: Checks maintainability, clarity, error handling, and naming conventions
- **Performance Review**: Identifies optimization opportunities (gas for Solidity, query efficiency for web)
- **Test Coverage**: Evaluates test completeness and missing edge cases
- **Architecture Review**: Assesses design patterns and code organization
- **Project-Aware**: Adapts to Foundry/Solidity, Next.js/React, or general projects

## How It Works

### Automatic Activation

The skill automatically activates when:
- Reviewing pull request changes
- User asks about code review or PR review
- User mentions "review", "PR", or "pull request"
- Analyzing diff or code changes

### Review Process

1. **Understand Context**: Reads PR details and project configuration
2. **Get Changes**: Fetches PR diff and changed files
3. **Systematic Review**: Analyzes across security, quality, performance, testing, and architecture
4. **Project-Specific Checks**: Applies framework-specific validations
5. **Generate Feedback**: Posts structured review comment to PR

## Usage Examples

### Example 1: Review a PR

```bash
# Ask Claude to review a specific PR
"Please review PR #42"
```

The skill will automatically:
- Fetch PR details and diff
- Analyze changes across all categories
- Post a structured review comment

### Example 2: Focus on Security

```bash
"Do a security review of PR #42"
```

### Example 3: Quick Code Review

```bash
"Give me feedback on these changes"
```

## Review Categories

### Security Analysis
- Authentication/authorization
- Input validation
- Injection vulnerabilities
- Sensitive data exposure
- Smart contract security (for Solidity)

### Code Quality
- Best practices adherence
- Maintainability
- Error handling
- Naming conventions

### Performance
- Gas optimization (Solidity)
- Query efficiency
- Memory usage
- Caching opportunities

### Testing
- Coverage for new code
- Edge cases
- Error conditions

### Architecture
- Design patterns
- Separation of concerns
- Dependency management

## Output Format

Reviews are posted as PR comments with this structure:

```markdown
## Code Review

### Summary
[Overview of PR and findings]

### Critical Issues
[Must-fix issues]

### Suggestions
[Recommendations]

### Questions
[Clarifications needed]

### Positive Findings
[Good patterns observed]

### Recommendation
- [ ] Approve
- [ ] Request Changes
- [ ] Comment Only
```

## Project-Specific Support

### Foundry/Solidity
- Forge fmt compliance
- Foundry test patterns
- EVK/EVC integration checks
- Gas optimization

### Next.js/TypeScript
- Type safety
- React best practices
- API route security
- Server/client boundaries

### General
- Linting compliance
- Formatting consistency
- Documentation updates

## Integration with CI/CD

Automatically run code reviews on PRs:

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

## Best Practices

1. **Review Early**: Run reviews as soon as PRs are opened
2. **Address Critical First**: Focus on blocking issues before suggestions
3. **Combine with Human Review**: Use as first-pass, not replacement
4. **Track Patterns**: Notice recurring issues for team learning
5. **Iterate**: Update skill with project-specific patterns

## Limitations

- Automated analysis may miss context-dependent issues
- Cannot understand business logic without documentation
- Should supplement, not replace, human review
- May generate false positives for intentional patterns

## Related Commands

- `/code-review` - Explicitly invoke this skill
- `/audit` - Deep security audit (for smart contracts)

## Contributing

To improve this skill:
1. Edit `skill.md` to add new check patterns
2. Update project-specific checks as needed
3. Add examples of common issues found
4. Test with real PR reviews
