---
description: Review pull request code for quality, security, and best practices
---

# Code Review Command

Perform a comprehensive code review of the current pull request and provide constructive feedback.

## Instructions

Please review this pull request and provide feedback on:

### Security Analysis
- Potential security vulnerabilities
- Access control issues
- Input validation problems
- External call safety
- Reentrancy risks

### Code Quality
- Best practices adherence
- Code clarity and maintainability
- Error handling
- Edge case coverage
- Naming conventions

### Performance Considerations
- Gas optimization opportunities (for Solidity)
- Inefficient patterns
- Unnecessary computations
- Storage vs memory usage

### Testing
- Test coverage for new code
- Missing test cases
- Edge cases not covered

### Architecture
- Design pattern usage
- Code organization
- Separation of concerns

## Review Guidelines

1. **Be Constructive**: Provide specific, actionable feedback
2. **Prioritize**: Highlight critical issues first
3. **Explain Why**: Don't just point out issues, explain the reasoning
4. **Suggest Fixes**: Provide code examples where helpful
5. **Acknowledge Good Work**: Point out well-written code too

## Output Format

After reviewing the code, use the `gh pr comment` command to post your review:

```bash
gh pr comment <PR_NUMBER> --body "## Code Review

### Summary
[Brief overview of the PR]

### Critical Issues
[List any critical security or functionality issues]

### Suggestions
[List improvements and optimizations]

### Positive Findings
[Highlight well-written code]

### Recommendation
- [ ] Approve
- [ ] Request Changes
- [ ] Comment Only
"
```

## Project-Specific Context

When reviewing Foundry/Solidity projects:
- Check for proper use of Foundry test patterns
- Verify Solidity version compatibility (0.8.24)
- Look for integration issues with external protocols (EVK, Uniswap, etc.)
- Ensure `forge fmt` has been run
- Verify test coverage with `forge coverage`

For EVK/EVC integration:
- Verify `callThroughEVC` modifier usage on operators
- Check `evc.getCurrentOnBehalfOfAccount()` for user identification
- Ensure `evc.requireAccountStatusCheck()` is called after state changes
- Validate vault operations go through EVC

Use the repository's CLAUDE.md file for project-specific conventions and patterns.
