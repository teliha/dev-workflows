---
name: Code Review Expert
description: Comprehensive code review for pull requests with quality, security, and best practices analysis
category: Quality
tags: [code-review, pull-request, quality, security, best-practices]
---

<!-- CODE-REVIEW:START -->

# Code Review Expert Skill

## When to Use This Skill

This skill automatically activates when:
- Reviewing pull request changes
- User asks about code review or PR review
- User mentions "review", "PR", or "pull request" in review context
- Analyzing diff or code changes
- User requests feedback on code quality

## Review Process

### Step 1: Understand the Context

Before reviewing, understand:

1. **PR Scope**
   - What is this PR trying to accomplish?
   - What files are changed?
   - What is the expected behavior change?

2. **Project Context**
   - What type of project? (Foundry/Solidity, Next.js/React, etc.)
   - What patterns does this project follow?
   - Are there project-specific conventions?

3. **Read CLAUDE.md**
   - Check for project-specific review guidelines
   - Understand architecture and patterns
   - Note any special considerations

### Step 2: Get PR Information

```bash
# Get PR details
gh pr view <PR_NUMBER>

# Get PR diff
gh pr diff <PR_NUMBER>

# Get PR files changed
gh pr view <PR_NUMBER> --json files
```

### Step 3: Systematic Review (PARALLEL)

**PARALLELIZATION OPPORTUNITY**: Run review categories in parallel for faster analysis:

```
Use the Task tool with parallel subagents:

Task 1: Security Review
  - Check for vulnerabilities
  - Review access control
  - Verify input validation
  - Check external call safety

Task 2: Code Quality Review
  - Check best practices
  - Review naming conventions
  - Check error handling
  - Identify code duplication

Task 3: Performance Review
  - Check for inefficiencies
  - Review loop optimizations
  - Check caching opportunities
  - Review database queries (if applicable)

Task 4: Testing Review
  - Check test coverage
  - Identify missing test cases
  - Review test quality
```

**For large PRs with many files**: Review files in parallel:
```
Task 1: Review src/auth/*.ts
Task 2: Review src/api/*.ts
Task 3: Review src/components/*.tsx
...
```

Each subagent provides findings for its scope, results are combined.

Review code changes systematically across these areas:

## Review Categories

### 1. Security Analysis

**What to check:**
- [ ] Potential security vulnerabilities
- [ ] Access control issues
- [ ] Input validation problems
- [ ] External call safety (for smart contracts)
- [ ] Reentrancy risks (for smart contracts)
- [ ] XSS/CSRF risks (for web apps)
- [ ] Injection vulnerabilities
- [ ] Sensitive data exposure

**Severity levels:**
- **CRITICAL**: Immediate security risk
- **HIGH**: Significant vulnerability
- **MEDIUM**: Moderate risk
- **LOW**: Minor concern

### 2. Code Quality

**What to check:**
- [ ] Best practices adherence
- [ ] Code clarity and maintainability
- [ ] Error handling completeness
- [ ] Edge case coverage
- [ ] Naming conventions
- [ ] Code duplication
- [ ] Function/method length
- [ ] Complexity management

**Quality indicators:**
- Is the code self-documenting?
- Are error messages helpful?
- Is the intent clear?

### 3. Performance Considerations

**For Solidity/Smart Contracts:**
- [ ] Gas optimization opportunities
- [ ] Storage vs memory usage
- [ ] Loop efficiency
- [ ] Unnecessary computations

**For Web/Backend:**
- [ ] Database query efficiency
- [ ] API call optimization
- [ ] Memory usage
- [ ] Caching opportunities

### 4. Testing

**What to check:**
- [ ] Test coverage for new code
- [ ] Missing test cases
- [ ] Edge cases not covered
- [ ] Test quality and clarity

**Questions to ask:**
- Are all new paths tested?
- Are error conditions tested?
- Are boundary conditions tested?

### 5. Architecture

**What to check:**
- [ ] Design pattern usage
- [ ] Code organization
- [ ] Separation of concerns
- [ ] Dependency management
- [ ] Interface design

### Step 4: Project-Specific Checks

#### Foundry/Solidity Projects

- [ ] `forge fmt` compliance
- [ ] Proper use of Foundry test patterns
- [ ] Solidity version compatibility
- [ ] Integration issues with external protocols

**For EVK/EVC integration:**
- [ ] `callThroughEVC` modifier usage on operators
- [ ] `evc.getCurrentOnBehalfOfAccount()` for user identification
- [ ] `evc.requireAccountStatusCheck()` after state changes
- [ ] Vault operations through EVC

#### Next.js/TypeScript Projects

- [ ] TypeScript type safety
- [ ] React best practices
- [ ] Server/client component boundaries
- [ ] API route security
- [ ] Environment variable handling

#### General Projects

- [ ] Linting compliance
- [ ] Formatting consistency
- [ ] Documentation updates
- [ ] Changelog updates (if applicable)

## Review Guidelines

### Be Constructive
- Provide specific, actionable feedback
- Explain the reasoning behind suggestions
- Offer alternative solutions when possible

### Prioritize
- Highlight critical issues first
- Separate blocking issues from suggestions
- Focus on impact

### Explain Why
- Don't just point out issues
- Explain the potential consequences
- Reference best practices or documentation

### Suggest Fixes
- Provide code examples where helpful
- Show the "before and after"
- Link to relevant documentation

### Acknowledge Good Work
- Point out well-written code
- Recognize good patterns
- Celebrate improvements

## Output Format

Post review as a PR comment using:

```bash
gh pr comment <PR_NUMBER> --body "## Code Review

### Summary
[Brief overview of the PR and review findings]

### Critical Issues
[List any critical security or functionality issues - MUST be fixed]

### Suggestions
[List improvements and optimizations - SHOULD be considered]

### Questions
[Any clarifications needed from the author]

### Positive Findings
[Highlight well-written code and good practices]

### Recommendation
- [ ] Approve - Ready to merge
- [ ] Request Changes - Issues must be addressed
- [ ] Comment Only - Feedback provided, author decides
"
```

## Common Patterns to Flag

### Security Issues

```typescript
// BAD: No authentication
export async function GET(req: Request) {
    const users = await db.users.findMany();
    return Response.json(users);
}

// GOOD: With authentication
export async function GET(req: Request) {
    const session = await auth(req);
    if (!session) return Response.json({ error: 'Unauthorized' }, { status: 401 });
    const users = await db.users.findMany();
    return Response.json(users);
}
```

### Code Quality Issues

```typescript
// BAD: Magic numbers
if (status === 1) { ... }

// GOOD: Named constants
const STATUS_ACTIVE = 1;
if (status === STATUS_ACTIVE) { ... }
```

### Performance Issues

```solidity
// BAD: Reading array length every iteration
for (uint256 i = 0; i < array.length; i++) { }

// GOOD: Cache array length
uint256 length = array.length;
for (uint256 i; i < length;) {
    unchecked { ++i; }
}
```

## Integration with CI/CD

This skill can be triggered automatically in GitHub Actions:

```yaml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          claude_args: '--allowed-tools "Bash(gh pr:*),mcp__github_inline_comment__create_inline_comment"'
          prompt: |
            /code-review

            PR #${{ github.event.pull_request.number }}
            Repository: ${{ github.repository }}
```

<!-- CODE-REVIEW:END -->
