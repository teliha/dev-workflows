---
description: Analyze test coverage and add tests to improve coverage
---

# Improve Coverage Command

Analyze test coverage and add new test cases to improve coverage without changing implementation.

## Auto-Detection

This command automatically detects your project type and uses the appropriate tools:

### Foundry/Solidity
```bash
forge coverage --report summary
```

### Next.js/Jest
```bash
npm run test:coverage
# or
jest --coverage
```

### Vitest
```bash
vitest --coverage
```

### Other Projects
Will attempt to detect test framework from package.json or project structure.

## Task

1. **Run coverage analysis**:
   - Foundry: `forge coverage --report lcov`
   - Jest: `jest --coverage`
   - Vitest: `vitest --coverage`

2. **Identify uncovered code paths**:
   - Functions with <80% coverage
   - Error cases not tested
   - Edge cases (zero values, max values, etc.)
   - State transitions
   - Boundary conditions

3. **Add new test cases**:
   - Add to existing test files in `test/` or `__tests__/`
   - Create new test files if needed
   - Follow existing test patterns
   - Use descriptive test names

4. **Focus Areas**:
   - Functions with low coverage
   - Error conditions (throws, reverts, rejects)
   - Edge cases and boundary conditions
   - State transitions
   - Access control checks

5. **Verification**:
   - Run tests to ensure all pass
   - Run coverage again to verify improvement
   - Target: Increase coverage by at least 5%

6. **Format and commit**:
   - Format code (forge fmt, prettier, etc.)
   - Commit: `git add test/ && git commit -m "test: improve coverage for [Feature]"`

## Critical Rules

- ❌ DO NOT modify any implementation code
- ❌ DO NOT refactor existing tests  
- ❌ DO NOT change production code behavior
- ✅ ONLY add new test cases
- ✅ Follow existing test patterns
- ✅ Use descriptive test names
- ✅ Add comments explaining test purpose
- ✅ Ensure all tests pass

## Project-Specific Patterns

### Foundry Projects
```solidity
// test/MyContract.t.sol
function test_RevertWhen_Unauthorized() public {
    vm.prank(unauthorizedUser);
    vm.expectRevert(MyContract.Unauthorized.selector);
    myContract.protectedFunction();
}
```

### Next.js/Jest Projects
```typescript
// __tests__/api/users.test.ts
describe('GET /api/users', () => {
  it('should return 401 when unauthorized', async () => {
    const res = await request(app).get('/api/users');
    expect(res.status).toBe(401);
  });
});
```

### React Component Tests
```typescript
// components/__tests__/Button.test.tsx
it('should call onClick when clicked', () => {
  const onClick = jest.fn();
  render(<Button onClick={onClick}>Click</Button>);
  fireEvent.click(screen.getByText('Click'));
  expect(onClick).toHaveBeenCalledTimes(1);
});
```

## Coverage Goals

Target coverage levels:
- **Critical paths**: 100%
- **Business logic**: 90%+
- **Utilities**: 80%+
- **UI components**: 70%+

Focus on:
1. Security-critical functions
2. Complex business logic
3. Error handling
4. Edge cases
