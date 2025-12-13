---
name: Lint Fix Expert
description: Fix lint and static analysis warnings one at a time with proper prioritization
category: Quality
tags: [lint, static-analysis, code-quality, eslint, clippy, solhint]
---

<!-- FIX-LINT:START -->

# Lint Fix Expert Skill

## When to Use This Skill

This skill automatically activates when:
- User asks to fix lint warnings
- User mentions "lint", "linter", or "static analysis"
- Working on code quality improvements
- User asks to fix ESLint, Clippy, or similar warnings

## Lint Fix Process

### Step 1: Detect Project Type and Linter

#### Foundry/Solidity
- Indicator: `foundry.toml`
- Format check: `forge fmt --check`
- Build warnings: `forge build`
- Solhint (if available): `npx solhint 'contracts/**/*.sol'`

#### Rust
- Indicator: `Cargo.toml`
- Linter: `cargo clippy --all-targets --all-features -- -W clippy::all`
- Format: `cargo fmt --check`

#### TypeScript/JavaScript
- Indicator: `package.json`
- ESLint: `npm run lint` or `eslint .`
- TypeScript: `tsc --noEmit`

#### Next.js
- Indicator: `next.config.js` or `next.config.ts`
- Lint: `npm run lint` (Next.js built-in ESLint)

### Step 2: Get Warnings List

Run the linter to get all warnings:

```bash
# ESLint
npm run lint 2>&1

# Clippy
cargo clippy --all-targets 2>&1

# Foundry
forge build 2>&1
forge fmt --check 2>&1
```

### Step 3: Prioritize Warnings

Fix warnings in this priority order:

1. **Security Issues** (Highest)
   - Unsafe operations
   - SQL injection risks
   - Command injection
   - Memory safety issues

2. **Correctness Issues**
   - Type errors
   - Logic bugs
   - Null/undefined risks

3. **Performance Issues**
   - Unnecessary allocations
   - Inefficient loops
   - Redundant operations

4. **Code Quality**
   - Unused variables
   - Deprecated APIs
   - Dead code

5. **Style Issues** (Lowest)
   - Formatting
   - Naming conventions
   - Import ordering

### Step 4: Fix ONE Warning

**Important**: Fix only ONE warning per invocation for controlled changes.

## Common Fixes by Linter

### ESLint

#### Unused Variables
```typescript
// Warning: 'x' is defined but never used
const x = getValue();  // Remove if truly unused

// Or prefix with underscore if intentionally unused
const _x = getValue();
```

#### Missing Await
```typescript
// Warning: Promises must be awaited
async function process() {
    fetchData();  // Missing await
}

// Fix
async function process() {
    await fetchData();
}
```

#### Prefer Const
```typescript
// Warning: 'x' is never reassigned
let x = 5;

// Fix
const x = 5;
```

### Clippy (Rust)

#### Unnecessary Clone
```rust
// Warning: redundant clone
let s = str.clone();

// Fix: Remove clone if not needed
let s = str;
```

#### Unused Variables
```rust
// Warning: unused variable
let x = 5;

// Fix: Prefix with underscore
let _x = 5;
```

#### Needless Return
```rust
// Warning: unneeded `return` statement
fn foo() -> i32 {
    return 5;
}

// Fix
fn foo() -> i32 {
    5
}
```

### Solidity/Foundry

#### Unused Imports
```solidity
// Warning: Imported name "Foo" is not used
import {Foo, Bar} from "./contracts.sol";

// Fix: Remove unused
import {Bar} from "./contracts.sol";
```

#### State Mutability
```solidity
// Warning: Function state mutability can be restricted to view
function getValue() public returns (uint256) {
    return value;
}

// Fix
function getValue() public view returns (uint256) {
    return value;
}
```

#### Visibility
```solidity
// Warning: Function visibility not set
function foo() {
    // ...
}

// Fix
function foo() internal {
    // ...
}
```

## Verification

After fixing:

```bash
# Re-run the linter
npm run lint  # ESLint
cargo clippy  # Rust
forge build   # Solidity

# Verify the specific warning is gone
# Run tests to ensure no regression
npm test
cargo test
forge test
```

## Output Format

After fixing a warning:

```markdown
## Lint Fix Summary

### Warning Fixed
**Type**: [Security/Correctness/Performance/Quality/Style]
**Linter**: [ESLint/Clippy/Solhint/Forge]
**Rule**: [rule-name]

### Location
**File**: `path/to/file.ts`
**Line**: 42

### Original Warning
```
[Exact warning message from linter]
```

### Fix Applied
```diff
- const x = getValue();
+ const _x = getValue();
```

### Explanation
[Why this fix is appropriate]

### Verification
- [x] Linter no longer reports this warning
- [x] Tests still pass
- [x] No new warnings introduced

### Commit
```bash
git add path/to/file.ts
git commit -m "fix(lint): [description of fix]"
```
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

## Best Practices

1. **One at a time**: Fix one warning per commit for clear history
2. **Prioritize wisely**: Security > Correctness > Performance > Style
3. **Verify after fix**: Always re-run linter and tests
4. **Don't suppress blindly**: Understand warnings before disabling
5. **Update configs**: If a rule is wrong for your project, update linter config

## When to Suppress Instead of Fix

Sometimes suppression is appropriate:

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any
const data: any = externalLib.getData();  // Third-party returns any
```

Only suppress when:
- External library limitations
- Intentional behavior that linter doesn't understand
- Legacy code that can't be changed safely
- Always add comment explaining why

<!-- FIX-LINT:END -->
