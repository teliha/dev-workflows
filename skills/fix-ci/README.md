# CI/CD Fix Expert Skill

## Overview

This Claude Code skill automatically analyzes and fixes CI/CD pipeline failures, including build errors, test failures, linting issues, and formatting problems across different project types.

## Features

- **Auto-Detection**: Automatically detects project type (Foundry, Next.js, Node.js, Rust, etc.)
- **Error Analysis**: Parses CI logs to identify root causes
- **Targeted Fixes**: Applies minimal, focused fixes
- **Verification**: Confirms fixes work before committing
- **Safety First**: Maintains existing patterns and security

## How It Works

### Automatic Activation

The skill automatically activates when:
- CI/CD pipeline fails
- User mentions "CI failure", "build error", or "test failure"
- User asks to fix failing workflows
- Analyzing GitHub Actions logs

### Fix Process

1. **Identify Failure**: Get and parse CI logs
2. **Detect Project**: Determine project type and tools
3. **Categorize Error**: Build, test, lint, format, or type error
4. **Apply Fix**: Make minimal targeted changes
5. **Verify**: Run local checks before committing

## Supported Project Types

| Project Type | Build | Test | Lint | Format |
|-------------|-------|------|------|--------|
| Foundry/Solidity | `forge build` | `forge test` | `solhint` | `forge fmt` |
| Next.js | `next build` | `jest` | `eslint` | `prettier` |
| Node.js/TS | `npm build` | `npm test` | `eslint` | `prettier` |
| Rust | `cargo build` | `cargo test` | `clippy` | `cargo fmt` |

## Error Categories

### Build Errors
- Missing imports
- Type mismatches
- Syntax errors
- Missing dependencies

### Test Failures
- Assertion failures
- Timeout issues
- Setup problems

### Linting Issues
- Code style violations
- Unused variables
- Import ordering

### Formatting Issues
- Whitespace
- Indentation
- Line endings

### Type Errors
- Missing types
- Wrong types
- Nullable handling

## Usage Examples

### Example 1: Fix Failed CI

```bash
"The CI is failing, can you fix it?"
```

The skill will:
1. Get the failure logs from the latest run
2. Identify the error type
3. Apply appropriate fixes
4. Verify and commit

### Example 2: Specific Run

```bash
"Fix the CI failure in run #12345"
```

### Example 3: After Viewing Logs

```bash
"I see type errors in the build, please fix them"
```

## Maintenance Rules

### DO:
- Fix only what's broken
- Maintain existing patterns
- Keep security best practices
- Verify fixes locally
- Make minimal changes

### DON'T:
- Refactor unrelated code
- Add new features
- Change test behavior unnecessarily
- Introduce breaking changes
- Skip verification

## Output Format

After fixing, provides a summary:

```markdown
## CI Fix Summary

### Error Type
[Build/Test/Lint/Format/Type]

### Root Cause
[Explanation]

### Fixes Applied
1. [Fix 1]
2. [Fix 2]

### Files Modified
- file1.ts
- file2.ts

### Verification
- [x] Build passes
- [x] Tests pass
```

## Integration with CI/CD

Use as composite action for automatic fixes:

```yaml
name: Auto-fix CI

on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]

jobs:
  fix:
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4  # Caller provides setup
      - run: npm ci
      - uses: teliha/dev-workflows/.github/actions/fix-ci@main
        with:
          failed_run_id: ${{ github.event.workflow_run.id }}
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Safety Checks

Before pushing:
- [ ] All tests pass locally
- [ ] Build succeeds
- [ ] Linting passes
- [ ] Formatting correct
- [ ] No new warnings
- [ ] Minimal changes
- [ ] No security issues

## Related Commands

- `/fix-ci` - Explicitly invoke this skill
- `/fix-lint` - Focus on linting issues only

## Limitations

- Cannot fix fundamental design issues
- May not understand complex business logic
- Requires access to CI logs
- Should be reviewed before merge

## Contributing

To improve this skill:
1. Add new error patterns to recognize
2. Add project-specific fixes
3. Improve verification steps
4. Test with real CI failures
