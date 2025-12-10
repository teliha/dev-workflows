Fix one important lint or static analysis warning.

## Detection Strategy

Auto-detect the project type and use the appropriate linter:

1. **Foundry (Solidity)**:
   - Check for `foundry.toml`
   - Run: `forge lint` to find linting issues (if available)
   - Run: `forge fmt --check` to find formatting issues
   - Run: `forge build` to find compilation warnings
   - If solhint is available: `npx solhint 'contracts/**/*.sol'`

2. **Rust**:
   - Check for `Cargo.toml`
   - Run: `cargo clippy --all-targets --all-features -- -W clippy::all`

3. **TypeScript/JavaScript**:
   - Check for `package.json`
   - Run: `npm run lint` (if available in scripts)
   - Or: `eslint .` (if eslint is installed)
   - Or: `tsc --noEmit` (for TypeScript type errors)

4. **Next.js**:
   - Check for `next.config.js` or `next.config.ts`
   - Run: `npm run lint` (Next.js has built-in ESLint)

## Priority Levels

When selecting which warning to fix, prioritize in this order:

1. **Security issues** (e.g., unsafe operations, SQL injection risks)
2. **Correctness issues** (e.g., type errors, logic bugs)
3. **Performance issues** (e.g., unnecessary allocations, inefficient loops)
4. **Code quality** (e.g., unused variables, deprecated APIs)
5. **Style issues** (e.g., formatting, naming conventions)

## Fix Process

1. Run the linter to get the full list of warnings
2. Select ONE important warning based on priority
3. Analyze the code and understand the warning
4. Fix the issue by modifying the source code
5. Run the linter again to verify the fix
6. Commit with format: `fix(lint): [description]`
7. Push to the current branch

## Examples

### Foundry Example
```bash
# Run linter (if available)
forge lint

# Check formatting
forge fmt --check

# Fix one formatting issue
forge fmt contracts/MyContract.sol

# Verify
forge build
```

### Rust Example
```bash
# Get warnings
cargo clippy --all-targets --all-features

# Fix one warning (e.g., unused variable)
# Edit the file to remove or prefix with underscore

# Verify
cargo clippy --all-targets --all-features
```

### TypeScript Example
```bash
# Get warnings
npm run lint

# Fix one warning (e.g., missing await)
# Edit the file to add await

# Verify
npm run lint
```

## Output

Generate a concise commit message that describes:
- What warning was fixed
- Which file was modified
- Brief explanation if the fix isn't obvious

Keep changes minimal and focused on fixing just ONE warning.
