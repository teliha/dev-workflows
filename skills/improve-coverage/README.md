# Test Coverage Improvement Expert Skill

## Overview

This Claude Code skill analyzes test coverage and adds new test cases to improve coverage without modifying implementation code.

## Features

- **Coverage Analysis**: Runs and parses coverage reports
- **Gap Identification**: Finds uncovered code paths
- **Smart Prioritization**: Focuses on critical paths first
- **Pattern Matching**: Follows existing test conventions
- **Safety Guarantee**: Never modifies implementation code

## How It Works

### Automatic Activation

The skill automatically activates when:
- User asks about test coverage
- User mentions "coverage", "test coverage", or "add tests"
- Working on improving test quality
- User asks to increase coverage percentage

### Process

1. **Run Coverage**: Execute coverage tool for project type
2. **Analyze Gaps**: Identify uncovered functions, branches, lines
3. **Prioritize**: Focus on critical paths and security-sensitive code
4. **Write Tests**: Add new test cases following existing patterns
5. **Verify**: Confirm coverage improvement and all tests pass

## Supported Frameworks

| Framework | Coverage Command | Test Location |
|-----------|-----------------|---------------|
| Foundry | `forge coverage` | `test/*.t.sol` |
| Jest | `jest --coverage` | `__tests__/` or `*.test.ts` |
| Vitest | `vitest --coverage` | `*.test.ts` |

## Critical Rules

### DO:
- ONLY add new test cases
- Follow existing test patterns
- Use descriptive test names
- Add comments explaining test purpose
- Ensure all tests pass

### DON'T:
- Modify implementation code
- Refactor existing tests
- Change production behavior
- Skip verification

## Usage Examples

### Example 1: Improve Overall Coverage

```bash
"Please improve the test coverage"
```

### Example 2: Target Specific Area

```bash
"Add tests for the utils/ directory"
```

### Example 3: Focus on Coverage Gaps

```bash
"The API routes have low coverage, add tests"
```

## Coverage Goals

| Area | Target |
|------|--------|
| Critical paths | 100% |
| Business logic | 90%+ |
| Utilities | 80%+ |
| UI components | 70%+ |

## Test Case Categories

### 1. Success Cases
Normal operation with valid inputs

### 2. Error Cases
- Invalid inputs
- Unauthorized access
- Resource not found

### 3. Edge Cases
- Zero/empty values
- Maximum values
- Boundary conditions

### 4. State Transitions
- Multi-step processes
- State machine coverage

## Output Format

```markdown
## Coverage Improvement Summary

### Before
- Overall: 65%

### After
- Overall: 78% (+13%)

### Tests Added
1. `test/utils/format.test.ts` - 12 new tests

### Verification
- [x] All tests pass
- [x] Coverage increased
```

## Integration with CI/CD

```yaml
name: Improve Coverage

on:
  workflow_dispatch:

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - uses: teliha/dev-workflows/.github/actions/improve-coverage@main
```

## Related Commands

- `/improve-coverage` - Explicitly invoke this skill
- `/code-review` - Review code including test coverage

## Limitations

- Cannot improve coverage for code without clear specification
- Requires understanding of business logic for meaningful tests
- Should be reviewed by developers for test quality

## Contributing

To improve this skill:
1. Add new test patterns for different frameworks
2. Improve coverage gap detection
3. Add more edge case patterns
4. Test with real codebases
