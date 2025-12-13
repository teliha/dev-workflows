---
description: Analyze and fix CI/CD pipeline failures
---

# Fix CI Command

Analyze CI failure logs and fix failing tests, builds, or linting issues.

This command uses the [CI/CD Fix Expert skill](../skills/fix-ci/skill.md) for detailed analysis and fixes.

## Usage

```bash
/fix-ci
```

## What It Does

The fix-ci skill automatically:

1. **Detects project type** - Foundry, Next.js, Node.js, Rust, etc.
2. **Analyzes failure logs** - Identifies root cause
3. **Applies targeted fixes** - Minimal, focused changes
4. **Verifies fixes** - Runs local checks before committing

## Supported Error Types

| Error Type | Examples |
|------------|----------|
| Build errors | Missing imports, type errors, syntax |
| Test failures | Assertions, timeouts, setup issues |
| Lint errors | ESLint, Clippy, Solhint |
| Format issues | Prettier, forge fmt, cargo fmt |
| Type errors | TypeScript strict mode violations |

## Project Support

| Project | Build | Test | Lint | Format |
|---------|-------|------|------|--------|
| Foundry | `forge build` | `forge test` | `solhint` | `forge fmt` |
| Next.js | `next build` | `jest` | `eslint` | `prettier` |
| Node.js | `npm build` | `npm test` | `eslint` | `prettier` |
| Rust | `cargo build` | `cargo test` | `clippy` | `cargo fmt` |

## Maintenance Rules

- Fix only what's broken
- Maintain existing patterns
- Keep security best practices
- Don't refactor unrelated code
- Don't add new features

## Safety Checks

Before committing, the skill verifies:
- [ ] All tests pass
- [ ] Build succeeds
- [ ] Linting passes
- [ ] Formatting is correct
- [ ] Changes are minimal

## Integration with CI/CD

Use as a composite action (caller provides build environment):

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

      # Caller provides setup (example for Node.js)
      - uses: actions/setup-node@v4
      - run: npm ci

      - uses: teliha/dev-workflows/.github/actions/fix-ci@main
        with:
          failed_run_id: ${{ github.event.workflow_run.id }}
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Related

- [CI/CD Fix Expert Skill](../skills/fix-ci/skill.md) - Detailed fix process
- [Fix Lint Command](./fix-lint.md) - Focus on lint issues only
