---
description: Analyze and fix CI/CD pipeline failures
---

# Fix CI Command

Analyze CI failure logs and fix failing tests, builds, or linting issues.

## Auto-Detection

This command automatically detects your project type and applies appropriate fixes:

### Foundry/Solidity
- Compilation errors: `forge build`
- Test failures: `forge test`
- Formatting: `forge fmt`

### Next.js/TypeScript
- Build errors: `npm run build` / `next build`
- Test failures: `npm test` / `jest`
- Linting: `eslint` / `next lint`
- Type errors: `tsc --noEmit`
- Formatting: `prettier`

### General Projects
- Will detect from CI logs and package.json
- Supports most common test frameworks and build tools

## Task

### 1. Analyze Failure Logs

Identify the root cause:
- **Compilation/Build errors**: Missing imports, type errors, syntax errors
- **Test failures**: Assertion failures, timeout issues, setup problems
- **Linting issues**: ESLint errors, code style violations
- **Formatting issues**: Prettier, forge fmt

### 2. Apply Fixes

**Foundry Projects:**
```bash
# Fix formatting
forge fmt

# Rebuild
forge build

# Run tests
forge test -vvv
```

**Next.js Projects:**
```bash
# Fix formatting
npm run format
# or
prettier --write .

# Fix linting
npm run lint:fix
# or
eslint --fix .

# Type check
tsc --noEmit

# Run tests
npm test

# Build
npm run build
```

**General:**
- Fix based on error type
- Run appropriate commands
- Verify fixes work

### 3. Maintain Code Quality

While fixing:
- ✅ Fix only what's broken
- ✅ Maintain existing patterns
- ✅ Keep security best practices
- ❌ Don't refactor unrelated code
- ❌ Don't add new features
- ❌ Don't change test behavior (unless fixing the test itself)

### 4. Verification

Before committing:
```bash
# Foundry
forge fmt --check
forge build
forge test

# Next.js
npm run lint
npm run type-check
npm test
npm run build

# General
# Run your CI commands locally
```

### 5. Commit and Push

```bash
git add .
git commit -m "fix: resolve CI failures - [brief description]"
git push origin <branch>
```

## Common Fixes

### TypeScript Type Errors
```typescript
// Before (error: Property 'foo' does not exist)
const x = obj.foo;

// After
const x = obj.foo as string;
// or
const x = 'foo' in obj ? obj.foo : undefined;
```

### Solidity Compilation Errors
```solidity
// Before (error: Undeclared identifier)
function foo() external {
    bar();
}

// After
function foo() external {
    _bar();  // Call the correct function
}
```

### Test Assertion Failures
```typescript
// Before (failing test)
expect(result).toBe(10);

// After (fix the expectation or the code)
expect(result).toBe(11);  // If the result is actually 11
```

### Import Errors
```typescript
// Before
import { Foo } from './foo';

// After (check file extension, path)
import { Foo } from './foo.js';
// or
import { Foo } from '@/lib/foo';
```

## Framework-Specific Notes

### Foundry
- Always run `forge fmt` before committing
- Use `forge test -vvv` for detailed error output
- Check `foundry.toml` for configuration issues

### Next.js
- Check `next.config.js` for build settings
- Verify environment variables are set
- Use `next build` to catch build-time errors
- Check `tsconfig.json` for TypeScript settings

### General
- Read error messages carefully
- Check recent commits for breaking changes
- Verify dependencies are installed
- Check for environment-specific issues

## Safety Checks

Before pushing fixes:
- [ ] All tests pass
- [ ] Build succeeds
- [ ] Linting passes
- [ ] Formatting is correct
- [ ] No new warnings introduced
- [ ] Changes are minimal and focused
