# Lint Fix Expert Skill

## Overview

This Claude Code skill fixes lint and static analysis warnings one at a time, prioritizing by severity and ensuring safe, verified changes.

## Features

- **Auto-Detection**: Detects project type and appropriate linter
- **Prioritized Fixes**: Security > Correctness > Performance > Style
- **One-at-a-Time**: Controlled changes with clear commit history
- **Verification**: Confirms fix works and no regressions
- **Multi-Linter Support**: ESLint, Clippy, Solhint, and more

## How It Works

### Automatic Activation

The skill automatically activates when:
- User asks to fix lint warnings
- User mentions "lint", "linter", or "static analysis"
- Working on code quality improvements

### Process

1. **Detect**: Identify project type and linter
2. **List**: Get all current warnings
3. **Prioritize**: Select highest priority warning
4. **Fix**: Apply appropriate fix
5. **Verify**: Re-run linter and tests
6. **Commit**: Single focused commit

## Supported Linters

| Project Type | Linter | Command |
|-------------|--------|---------|
| TypeScript/JS | ESLint | `npm run lint` |
| Next.js | ESLint | `npm run lint` |
| Rust | Clippy | `cargo clippy` |
| Solidity | Forge/Solhint | `forge fmt` |

## Priority Order

1. **Security Issues** - Unsafe operations, injection risks
2. **Correctness Issues** - Type errors, logic bugs
3. **Performance Issues** - Inefficient code
4. **Code Quality** - Unused code, deprecated APIs
5. **Style Issues** - Formatting, naming

## Usage Examples

### Example 1: Fix Highest Priority

```bash
"Fix a lint warning"
```

### Example 2: Fix Specific Type

```bash
"Fix unused variable warnings"
```

### Example 3: Continuous Fixing

```bash
"Keep fixing lint warnings until clean"
```

## Common Fixes

### ESLint
- Unused variables: Remove or prefix with `_`
- Missing await: Add `await` keyword
- Prefer const: Change `let` to `const`

### Clippy
- Unnecessary clone: Remove `.clone()`
- Needless return: Remove explicit `return`

### Solidity
- State mutability: Add `view`/`pure`
- Unused imports: Remove from import statement

## Output Format

```markdown
## Lint Fix Summary

### Warning Fixed
**Type**: Quality
**Rule**: @typescript-eslint/no-unused-vars

### Location
`src/utils.ts:42`

### Fix Applied
```diff
- const x = getValue();
+ const _x = getValue();
```

### Verification
- [x] No more warnings for this issue
- [x] Tests pass
```

## Integration with CI/CD

```yaml
name: Fix Lint

on:
  workflow_dispatch:

jobs:
  fix:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - uses: anthropics/claude-code-action@v1
        with:
          prompt: /fix-lint
```

## Best Practices

1. **One warning per commit**: Clear, reviewable changes
2. **Verify always**: Re-run linter and tests
3. **Understand before fixing**: Don't blindly suppress
4. **Update config if needed**: Wrong rules should be disabled project-wide

## Related Commands

- `/fix-lint` - Explicitly invoke this skill
- `/fix-ci` - Fix all CI failures including lint

## Limitations

- Fixes one warning at a time (by design)
- Cannot fix architectural issues
- Some warnings require human judgment

## Contributing

To improve this skill:
1. Add new linter support
2. Add common fix patterns
3. Improve prioritization logic
4. Test with real lint outputs
