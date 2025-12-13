---
name: CI/CD Fix Expert
description: Analyze and fix CI/CD pipeline failures including build errors, test failures, and linting issues
category: DevOps
tags: [ci-cd, build, test, lint, fix, automation]
---

<!-- FIX-CI:START -->

# CI/CD Fix Expert Skill

## When to Use This Skill

This skill automatically activates when:
- CI/CD pipeline fails
- User mentions "CI failure", "build error", or "test failure"
- User asks to fix failing workflows
- Analyzing GitHub Actions logs
- User mentions "pipeline" or "workflow" in error context

## Fix Process

### Step 1: Identify the Failure

1. **Get failure logs**
   ```bash
   # Get the failed workflow run
   gh run view <RUN_ID>

   # Get detailed logs
   gh run view <RUN_ID> --log-failed
   ```

2. **Categorize the failure**
   - Compilation/Build errors
   - Test failures
   - Linting issues
   - Formatting issues
   - Type errors
   - Dependency issues

### Step 2: Auto-Detect Project Type

Detect project type to apply appropriate fixes:

#### Foundry/Solidity
- Indicator: `foundry.toml` exists
- Build: `forge build`
- Test: `forge test`
- Format: `forge fmt`
- Lint: `solhint` (if available)

#### Next.js/TypeScript
- Indicator: `next.config.js` or `next.config.ts`
- Build: `npm run build` / `next build`
- Test: `npm test` / `jest`
- Lint: `eslint` / `next lint`
- Type check: `tsc --noEmit`
- Format: `prettier`

#### Node.js/TypeScript
- Indicator: `package.json` with TypeScript
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`
- Type check: `tsc --noEmit`

#### Rust
- Indicator: `Cargo.toml`
- Build: `cargo build`
- Test: `cargo test`
- Lint: `cargo clippy`
- Format: `cargo fmt`

#### General
- Detect from CI logs and config files
- Support most common build tools

### Step 3: Apply Fixes by Error Type

## Error Categories and Fixes

### 1. Compilation/Build Errors

**Common causes:**
- Missing imports
- Type errors
- Syntax errors
- Missing dependencies

**Fix approach:**
1. Read the error message carefully
2. Identify the file and line number
3. Fix the specific issue
4. Verify with build command

**TypeScript example:**
```typescript
// Error: Property 'foo' does not exist on type 'Bar'

// Fix options:
// 1. Add the property
interface Bar {
    foo: string;
}

// 2. Type assertion (if intentional)
const x = (obj as any).foo;

// 3. Optional chaining (if might not exist)
const x = obj.foo ?? undefined;
```

**Solidity example:**
```solidity
// Error: Undeclared identifier 'bar'

// Fix: Check if function exists, correct spelling
function foo() external {
    _bar();  // Use correct function name
}
```

### 2. Test Failures

**Common causes:**
- Assertion failures
- Timeout issues
- Setup/teardown problems
- Environment differences

**Fix approach:**
1. Identify which test is failing
2. Understand expected vs actual values
3. Determine if test or code is wrong
4. Fix appropriately

**Important:** Do NOT blindly change test expectations. Understand WHY it's failing.

**Example:**
```typescript
// If the test expectation is wrong:
expect(result).toBe(10);  // Fails with actual: 11
// Fix: Update if 11 is actually correct
expect(result).toBe(11);

// If the code is wrong:
// Fix the implementation, not the test
```

### 3. Linting Issues

**Common issues:**
- Code style violations
- Unused variables
- Missing semicolons
- Import ordering

**Fix approach:**
1. Run lint with fix option
2. Address remaining issues manually
3. Verify no new issues introduced

**Commands:**
```bash
# ESLint
npm run lint -- --fix
# or
eslint --fix .

# Prettier
npm run format
# or
prettier --write .

# Solidity
forge fmt
```

### 4. Formatting Issues

**Fix automatically:**
```bash
# Foundry
forge fmt

# Prettier
prettier --write .

# Rust
cargo fmt
```

### 5. Type Errors

**Common fixes:**
```typescript
// Missing type
const x = getData();  // Error: implicit any
const x: DataType = getData();  // Fix: add type

// Wrong type
const x: string = 123;  // Error: not assignable
const x: number = 123;  // Fix: correct type

// Nullable
const len = str.length;  // Error: possibly undefined
const len = str?.length ?? 0;  // Fix: handle null
```

### 6. Import Errors

**Common fixes:**
```typescript
// Wrong path
import { Foo } from './foo';  // Error: not found
import { Foo } from './foo.js';  // Fix: add extension
// or
import { Foo } from '@/lib/foo';  // Fix: use alias

// Missing package
import { z } from 'zod';  // Error: not found
// Fix: npm install zod
```

## Maintenance Rules

### DO:
- Fix only what's broken
- Maintain existing patterns
- Keep security best practices
- Verify fixes work locally
- Make minimal changes

### DON'T:
- Refactor unrelated code
- Add new features
- Change test behavior (unless fixing the test itself)
- Introduce breaking changes
- Skip verification

## Verification

Before committing fixes:

**Foundry:**
```bash
forge fmt --check
forge build
forge test
```

**Next.js/TypeScript:**
```bash
npm run lint
npm run type-check  # or tsc --noEmit
npm test
npm run build
```

**General:**
```bash
# Run the same CI commands locally
```

## Output Format

After fixing, provide a summary:

```markdown
## CI Fix Summary

### Error Type
[Build/Test/Lint/Format/Type]

### Root Cause
[Brief explanation of what caused the failure]

### Fixes Applied
1. [Description of fix 1]
2. [Description of fix 2]

### Files Modified
- `path/to/file1.ts`
- `path/to/file2.ts`

### Verification
- [x] Build passes
- [x] Tests pass
- [x] Lint passes
- [x] Format check passes

### Commit
```bash
git add .
git commit -m "fix: resolve CI failures - [brief description]"
```
```

## Safety Checks

Before pushing fixes:
- [ ] All tests pass locally
- [ ] Build succeeds
- [ ] Linting passes
- [ ] Formatting is correct
- [ ] No new warnings introduced
- [ ] Changes are minimal and focused
- [ ] No security issues introduced

## Integration with CI/CD

Use as a composite action in failing workflows:

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

      # Caller provides build environment
      - uses: actions/setup-node@v4  # or foundry-toolchain, etc.
      - run: npm ci

      - uses: teliha/dev-workflows/.github/actions/fix-ci@main
        with:
          failed_run_id: ${{ github.event.workflow_run.id }}
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

<!-- FIX-CI:END -->
