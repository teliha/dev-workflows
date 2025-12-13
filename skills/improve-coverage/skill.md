---
name: Test Coverage Improvement Expert
description: Analyze test coverage and add new test cases to improve coverage without changing implementation
category: Quality
tags: [testing, coverage, quality, test-cases, tdd]
---

<!-- IMPROVE-COVERAGE:START -->

# Test Coverage Improvement Expert Skill

## When to Use This Skill

This skill automatically activates when:
- User asks about test coverage
- User mentions "coverage", "test coverage", or "add tests"
- Working on improving test quality
- User asks to increase coverage percentage
- Reviewing coverage reports

## Coverage Improvement Process

### Step 1: Run Coverage Analysis

Detect project type and run appropriate coverage tool:

#### Foundry/Solidity
```bash
forge coverage --report summary
forge coverage --report lcov  # For detailed report
```

#### Jest (Next.js/React/Node.js)
```bash
npm run test:coverage
# or
jest --coverage
```

#### Vitest
```bash
vitest --coverage
# or
npm run test:coverage
```

#### Other
- Detect from `package.json` scripts
- Use project-specific coverage tools

### Step 2: Identify Uncovered Code

Analyze coverage report for:

1. **Functions with low coverage** (<80%)
2. **Uncovered branches**
   - If statements
   - Ternary operators
   - Logical operators (&&, ||)
3. **Error cases not tested**
   - Error throws/reverts
   - Catch blocks
   - Validation failures
4. **Edge cases**
   - Zero values
   - Maximum values
   - Empty arrays/strings
   - Null/undefined handling
5. **State transitions**
   - State machine transitions
   - Multi-step processes

### Step 3: Prioritize Test Cases

Priority order for new tests:

1. **Critical paths** - Core business logic (target: 100%)
2. **Security-critical** - Auth, permissions, validation
3. **Complex logic** - High cyclomatic complexity
4. **Error handling** - All error conditions
5. **Edge cases** - Boundary conditions
6. **Utilities** - Helper functions

### Step 4: Write New Tests

## Test Writing Guidelines

### Critical Rule: DO NOT Modify Implementation

- ONLY add new test cases
- DO NOT change production code
- DO NOT refactor existing tests
- DO NOT change test behavior

### Follow Existing Patterns

Study existing tests and match:
- File organization
- Naming conventions
- Setup/teardown patterns
- Assertion style

### Use Descriptive Names

```typescript
// Good: Describes scenario and expected outcome
it('should return 401 when user is not authenticated', () => {})
it('should throw ValidationError when amount is negative', () => {})

// Bad: Vague or incomplete
it('works', () => {})
it('test error', () => {})
```

### Test Structure

Follow Arrange-Act-Assert pattern:

```typescript
it('should calculate total with discount', () => {
    // Arrange
    const cart = createCart([item1, item2]);
    const discount = 0.1;

    // Act
    const total = cart.calculateTotal(discount);

    // Assert
    expect(total).toBe(expectedTotal);
});
```

## Project-Specific Patterns

### Foundry/Solidity

```solidity
// test/MyContract.t.sol

// Test successful case
function test_Deposit_Success() public {
    uint256 amount = 1 ether;
    vm.deal(user, amount);

    vm.prank(user);
    myContract.deposit{value: amount}();

    assertEq(myContract.balances(user), amount);
}

// Test revert case
function test_RevertWhen_InsufficientBalance() public {
    vm.prank(user);
    vm.expectRevert(MyContract.InsufficientBalance.selector);
    myContract.withdraw(1 ether);
}

// Test edge case
function test_Deposit_ZeroAmount() public {
    vm.prank(user);
    vm.expectRevert(MyContract.InvalidAmount.selector);
    myContract.deposit{value: 0}();
}
```

### Jest/TypeScript

```typescript
// __tests__/api/users.test.ts
describe('GET /api/users', () => {
    // Success case
    it('should return users when authenticated', async () => {
        mockAuth.mockResolvedValue({ user: testUser });

        const res = await request(app).get('/api/users');

        expect(res.status).toBe(200);
        expect(res.body.data).toHaveLength(3);
    });

    // Error case
    it('should return 401 when not authenticated', async () => {
        mockAuth.mockResolvedValue(null);

        const res = await request(app).get('/api/users');

        expect(res.status).toBe(401);
    });

    // Edge case
    it('should return empty array when no users exist', async () => {
        mockDb.users.findMany.mockResolvedValue([]);

        const res = await request(app).get('/api/users');

        expect(res.body.data).toEqual([]);
    });
});
```

### React Component Tests

```typescript
// components/__tests__/Button.test.tsx
describe('Button', () => {
    it('should render with text', () => {
        render(<Button>Click me</Button>);
        expect(screen.getByText('Click me')).toBeInTheDocument();
    });

    it('should call onClick when clicked', () => {
        const onClick = jest.fn();
        render(<Button onClick={onClick}>Click</Button>);

        fireEvent.click(screen.getByText('Click'));

        expect(onClick).toHaveBeenCalledTimes(1);
    });

    it('should be disabled when disabled prop is true', () => {
        render(<Button disabled>Click</Button>);

        expect(screen.getByRole('button')).toBeDisabled();
    });

    it('should show loading state', () => {
        render(<Button loading>Click</Button>);

        expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
    });
});
```

## Coverage Goals

Target coverage by area:

| Area | Target | Priority |
|------|--------|----------|
| Critical paths | 100% | Highest |
| Business logic | 90%+ | High |
| Utilities | 80%+ | Medium |
| UI components | 70%+ | Lower |

Focus on:
1. Security-critical functions
2. Complex business logic
3. Error handling
4. Edge cases

## Verification

After adding tests:

1. **Run all tests**
   ```bash
   # Foundry
   forge test

   # Jest/Vitest
   npm test
   ```

2. **Re-run coverage**
   ```bash
   # Foundry
   forge coverage --report summary

   # Jest
   npm run test:coverage
   ```

3. **Verify improvement**
   - Coverage increased (target: +5% or more)
   - No test failures
   - No regressions

## Output Format

After adding tests:

```markdown
## Coverage Improvement Summary

### Before
- Overall: 65%
- src/utils/: 45%
- src/api/: 70%

### After
- Overall: 78% (+13%)
- src/utils/: 85% (+40%)
- src/api/: 82% (+12%)

### Tests Added
1. `test/utils/format.test.ts` - 12 new tests
   - Edge cases for date formatting
   - Error handling for invalid inputs

2. `test/api/users.test.ts` - 8 new tests
   - Unauthorized access tests
   - Pagination edge cases

### Files Modified
- `test/utils/format.test.ts` (new)
- `test/api/users.test.ts` (modified)

### Verification
- [x] All tests pass
- [x] Coverage increased by 13%
- [x] No regressions
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

<!-- IMPROVE-COVERAGE:END -->
