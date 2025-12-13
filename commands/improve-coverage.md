---
description: Analyze test coverage and add tests to improve coverage
---

# Improve Coverage Command

Analyze test coverage and add new test cases to improve coverage without changing implementation.

This command uses the [Test Coverage Improvement Expert skill](../skills/improve-coverage/skill.md) for analysis and test generation.

## Usage

```bash
/improve-coverage
```

## What It Does

The improve-coverage skill automatically:

1. **Runs coverage analysis** - Uses appropriate tool for project type
2. **Identifies gaps** - Functions, branches, and lines not covered
3. **Prioritizes** - Critical paths and security functions first
4. **Writes tests** - Following existing patterns
5. **Verifies improvement** - Confirms coverage increase

## Critical Rules

| DO | DON'T |
|----|-------|
| Add new test cases | Modify implementation code |
| Follow existing patterns | Refactor existing tests |
| Use descriptive names | Change production behavior |
| Ensure all tests pass | Skip verification |

## Coverage Goals

| Area | Target |
|------|--------|
| Critical paths | 100% |
| Business logic | 90%+ |
| Utilities | 80%+ |
| UI components | 70%+ |

## Project Support

| Project | Coverage Command |
|---------|-----------------|
| Foundry | `forge coverage --report summary` |
| Jest | `jest --coverage` |
| Vitest | `vitest --coverage` |

## Test Case Categories

- **Success cases**: Normal operation with valid inputs
- **Error cases**: Invalid inputs, unauthorized access
- **Edge cases**: Zero values, max values, boundaries
- **State transitions**: Multi-step processes

## Example Tests

### Foundry/Solidity
```solidity
function test_RevertWhen_Unauthorized() public {
    vm.prank(unauthorizedUser);
    vm.expectRevert(MyContract.Unauthorized.selector);
    myContract.protectedFunction();
}
```

### Jest/TypeScript
```typescript
it('should return 401 when unauthorized', async () => {
    const res = await request(app).get('/api/users');
    expect(res.status).toBe(401);
});
```

## Integration with CI/CD

```yaml
name: Improve Coverage

on:
  workflow_dispatch:
  schedule:
    - cron: "0 0 * * 0"  # Weekly

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - uses: teliha/dev-workflows/.github/actions/improve-coverage@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Related

- [Test Coverage Improvement Skill](../skills/improve-coverage/skill.md) - Detailed process
- [Code Review Command](./code-review.md) - Review including test coverage
