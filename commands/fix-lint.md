---
description: Fix one important lint or static analysis warning
---

# Fix Lint Command

Fix one important lint or static analysis warning with proper prioritization.

This command uses the [Lint Fix Expert skill](../skills/fix-lint/skill.md) for targeted fixes.

## Usage

```bash
/fix-lint
```

## What It Does

The skill automatically:

1. **Detects linter** - ESLint, Clippy, Solhint, etc.
2. **Gets warnings** - Runs linter to list all issues
3. **Prioritizes** - Selects highest priority warning
4. **Fixes ONE** - Applies targeted fix
5. **Verifies** - Re-runs linter and tests

## Priority Order

| Priority | Type | Examples |
|----------|------|----------|
| 1 | Security | Unsafe operations, injection risks |
| 2 | Correctness | Type errors, logic bugs |
| 3 | Performance | Inefficient loops, allocations |
| 4 | Quality | Unused code, deprecated APIs |
| 5 | Style | Formatting, naming |

## Supported Linters

| Project | Linter | Command |
|---------|--------|---------|
| TypeScript/JS | ESLint | `npm run lint` |
| Next.js | ESLint | `npm run lint` |
| Rust | Clippy | `cargo clippy` |
| Solidity | Forge/Solhint | `forge fmt` |

## Why One at a Time?

- Clear, reviewable commits
- Easy to track what changed
- Safe to revert if needed
- Better git history

## Example Fix

```markdown
## Lint Fix Summary

### Warning Fixed
**Rule**: @typescript-eslint/no-unused-vars
**File**: `src/utils.ts:42`

### Fix Applied
```diff
- const x = getValue();
+ const _x = getValue();
```

### Verification
- [x] Linter passes
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
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          prompt: /fix-lint
```

## Related

- [Lint Fix Expert Skill](../skills/fix-lint/skill.md) - Detailed fix process
- [Fix CI Command](./fix-ci.md) - Fix all CI failures including lint
