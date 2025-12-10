# Dev Workflows Plugin

Universal automated development workflows for any project type - security audits, coverage improvement, CI fixes, and spec validation.

## Overview

This Claude Code plugin provides essential development workflow commands that work across different project types. Whether you're building smart contracts with Foundry, web apps with Next.js, or any other project, these workflows help maintain code quality and automate repetitive tasks.

## Supported Project Types

- **Foundry/Solidity**: Smart contract development with security audits
- **Next.js/React**: Frontend development with TypeScript support
- **General Projects**: Any codebase with tests and CI/CD

## Features

### 🔒 Security Audit (`/audit`)
- Comprehensive code security analysis
- **Foundry**: Smart contract vulnerability checks (reentrancy, access control, etc.)
- **Next.js**: XSS, CSRF, API security, authentication issues
- **General**: Common security anti-patterns
- Automated audit report generation

### 📊 Code Review (`/code-review`)
- Pull request code quality analysis
- Security vulnerability detection
- Best practices verification
- Performance optimization suggestions
- Framework-specific checks

### 🔧 CI Auto-Fix (`/fix-ci`)
- Automatic CI failure diagnosis
- Test failure resolution
- Build error fixes
- Formatting and linting corrections
- Works with any test framework

### 🧪 Coverage Improvement (`/improve-coverage`)
- Test coverage analysis
- Automated test case generation
- Edge case identification
- Coverage report generation
- **Foundry**: `forge coverage`
- **Next.js/Jest**: `jest --coverage`
- **General**: Auto-detect test framework

### 📋 Spec Validation (`/check-spec-contradictions`)
- Specification consistency checking
- Cross-file contradiction detection
- Documentation gap identification
- Ambiguity detection

## Installation

### Add to Your Project

1. **Add Plugin Marketplace** to your `.claude/settings.json`:

```json
{
  "pluginMarketplaces": [
    {
      "name": "teliha-workflows",
      "url": "https://github.com/teliha/dev-workflows"
    }
  ],
  "plugins": [
    {
      "name": "dev-workflows",
      "marketplace": "teliha-workflows",
      "enabled": true
    }
  ]
}
```

2. **Install via Claude Code**:

```bash
# Interactive installation
/plugin

# Or direct installation
/plugin marketplace add https://github.com/teliha/dev-workflows
/plugin install dev-workflows@teliha-workflows
```

### Local Development/Testing

```bash
# Clone the plugin repository
git clone https://github.com/teliha/dev-workflows

# Add local marketplace
/plugin marketplace add ./dev-workflows

# Install from local
/plugin install dev-workflows@local
```

## Usage

### Security Audit

Run a comprehensive security audit:

```bash
/audit
```

**Foundry Projects:**
- Analyzes all Solidity files in `src/`
- Checks for reentrancy, access control, integer overflow, etc.
- Verifies DeFi integration patterns (EVK, Uniswap, etc.)

**Next.js Projects:**
- Analyzes API routes, server components, middleware
- Checks for XSS, CSRF, injection vulnerabilities
- Reviews authentication/authorization logic
- Validates environment variable usage

**General Projects:**
- Code security best practices
- Dependency vulnerability scanning
- Common anti-patterns

### Code Review

Review a pull request (run from PR branch):

```bash
/code-review
```

Provides feedback on:
- Security vulnerabilities
- Code quality and maintainability
- Performance optimization opportunities
- Test coverage gaps
- Framework-specific best practices

### Fix CI Failures

Automatically diagnose and fix CI failures:

```bash
/fix-ci
```

Works with:
- **Foundry**: `forge test`, `forge build`, `forge fmt`
- **Next.js**: `npm test`, `npm run build`, `eslint`, `prettier`
- **General**: Any CI framework

### Improve Test Coverage

Add tests to improve coverage:

```bash
/improve-coverage
```

**Foundry:**
```bash
forge coverage --report summary
```

**Next.js/Jest:**
```bash
npm run test:coverage
# or
jest --coverage
```

**Vitest:**
```bash
vitest --coverage
```

### Check Specification Contradictions

Analyze specification files for inconsistencies:

```bash
/check-spec-contradictions
```

Scans `specs/`, `docs/`, `README.md` for:
- Contradictions between specs
- Ambiguous requirements
- Missing critical information

### Fix Lint Warning

Fix one important lint or static analysis warning:

```bash
/fix-lint
```

**Supported Languages:**
- **TypeScript/JavaScript**: ESLint, TypeScript compiler
- **Rust**: Clippy warnings
- **Foundry/Solidity**: Forge formatting, Solhint

Automatically:
1. Detects project type
2. Runs appropriate linter
3. Fixes ONE important warning (prioritizes security > correctness > performance)
4. Commits and pushes the fix

## GitHub Actions Components

This plugin provides both **reusable workflows** and **composite actions** for automated CI/CD.

### Component Categories

**Universal Workflows (No build required):**
- Security Audit - Static code analysis
- Code Review - PR review
- Spec Check - Documentation analysis

**Composite Actions (Requires build environment):**
- Auto Fix CI - You provide the build environment
- Improve Coverage - You provide the build environment
- Fix Lint Warning - You provide the build environment

### Available Reusable Workflows

#### 1. Security Audit Workflow

**Universal workflow** - Works with any project type without setup.

```yaml
name: Security Audit

on:
  schedule:
    - cron: "0 3 * * 1"  # Weekly
  workflow_dispatch:

jobs:
  audit:
    uses: teliha/dev-workflows/.github/workflows/security-audit.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

**Works with:** All project types (Foundry, Next.js, Node.js, Python, etc.)  
**Setup required:** None - uses static analysis only

---

#### 2. Code Review Workflow

```yaml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize, ready_for_review]

jobs:
  review:
    if: github.event.pull_request.draft == false
    uses: teliha/dev-workflows/.github/workflows/code-review.yml@main
    with:
      pr_number: ${{ github.event.pull_request.number }}
      paths_filter: "src/**"  # Customize for your project
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

**Paths filter examples:**
- Foundry: `src/**/*.sol`
- Next.js: `src/**/*.{ts,tsx}` or `app/**`
- General: `**/*.{js,ts,py}`

---

#### 3. Auto Fix CI (Composite Action)

**Universal action - works with any project type. You provide the build environment.**

**For Foundry projects:**
```yaml
name: Auto Fix CI

on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]

jobs:
  auto-fix:
    if: github.event.workflow_run.conclusion == 'failure'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.workflow_run.head_branch }}
      
      # Setup your build environment
      - uses: foundry-rs/foundry-toolchain@v1
        with:
          version: nightly
      - run: forge install
      
      # Use the composite action
      - uses: teliha/dev-workflows/.github/actions/fix-ci@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          failed_run_id: ${{ github.event.workflow_run.id }}
          failed_branch: ${{ github.event.workflow_run.head_branch }}
          pr_number: ${{ github.event.workflow_run.pull_requests[0].number }}
```

**For Node.js/Next.js projects:**
```yaml
name: Auto Fix CI

on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]

jobs:
  auto-fix:
    if: github.event.workflow_run.conclusion == 'failure'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.workflow_run.head_branch }}
      
      # Setup your build environment
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      
      # Use the composite action
      - uses: teliha/dev-workflows/.github/actions/fix-ci@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          failed_run_id: ${{ github.event.workflow_run.id }}
          failed_branch: ${{ github.event.workflow_run.head_branch }}
          pr_number: ${{ github.event.workflow_run.pull_requests[0].number }}
```

---

#### 4. Improve Coverage (Composite Action)

**Universal action - works with any project type. You provide the build environment.**

**For Foundry projects:**
```yaml
name: Improve Coverage

on:
  schedule:
    - cron: "0 */8 * * *"
  workflow_dispatch:

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      # Setup your build environment
      - uses: foundry-rs/foundry-toolchain@v1
        with:
          version: nightly
      - run: forge install
      
      # Use the composite action
      - uses: teliha/dev-workflows/.github/actions/improve-coverage@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          target_coverage_increase: "5"
          base_branch: main
```

**For Node.js/Next.js projects:**
```yaml
name: Improve Coverage

on:
  schedule:
    - cron: "0 */8 * * *"
  workflow_dispatch:

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      # Setup your build environment
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      
      # Use the composite action
      - uses: teliha/dev-workflows/.github/actions/improve-coverage@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          target_coverage_increase: "5"
          base_branch: main
```

---

#### 5. Fix Lint Warning (Composite Action)

**Universal action - works with any project type. You provide the build environment.**

**For Foundry projects:**
```yaml
name: Fix Lint

on:
  schedule:
    - cron: "0 4 * * *"  # Daily at 4 AM
  workflow_dispatch:

jobs:
  fix-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Setup your build environment
      - uses: foundry-rs/foundry-toolchain@v1
        with:
          version: nightly
      - run: forge install
      
      # Use the composite action
      - uses: teliha/dev-workflows/.github/actions/fix-lint@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          base_branch: main
```

**For TypeScript/Node.js projects:**
```yaml
name: Fix Lint

on:
  schedule:
    - cron: "0 4 * * *"  # Daily at 4 AM
  workflow_dispatch:

jobs:
  fix-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Setup your build environment
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      
      # Use the composite action
      - uses: teliha/dev-workflows/.github/actions/fix-lint@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          base_branch: main
```

**For Rust projects:**
```yaml
name: Fix Lint

on:
  schedule:
    - cron: "0 4 * * *"  # Daily at 4 AM
  workflow_dispatch:

jobs:
  fix-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Setup your build environment
      - uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          components: clippy
      
      # Use the composite action
      - uses: teliha/dev-workflows/.github/actions/fix-lint@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          base_branch: main
```

**Features:**
- Fixes ONE important lint warning per run
- Prioritizes: Security > Correctness > Performance > Style
- Creates focused, reviewable commits
- Auto-detects: TypeScript/JavaScript (ESLint), Rust (Clippy), Solidity (Forge fmt)

---

#### 6. Spec Check Workflow

```yaml
name: Spec Check

on:
  schedule:
    - cron: "0 */8 * * *"
  workflow_dispatch:

jobs:
  check:
    uses: teliha/dev-workflows/.github/workflows/spec-check.yml@main
    with:
      specs_directory: "specs/"
      docs_directory: "docs/"
      create_issue_on_findings: true
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

---

## Project-Specific Configuration

### Foundry/Solidity Projects

Create `CLAUDE.md` in your repository:

```markdown
## Project Overview
Foundry-based DeFi protocol with EVK integration.

## Security Patterns
- All operators use `callThroughEVC` modifier
- EVC authentication required
- Health checks via `evc.requireAccountStatusCheck()`

## Testing
- Use `forge test` for unit tests
- Coverage: `forge coverage`
- Format: `forge fmt`
```

### Next.js Projects

Create `CLAUDE.md`:

```markdown
## Project Overview
Next.js 14 app with App Router and TypeScript.

## Security Patterns
- API routes use authentication middleware
- Server actions validate input with Zod
- Environment variables properly scoped

## Testing
- Jest for unit tests
- Playwright for E2E
- Coverage: `npm run test:coverage`
```

### General Projects

Provide context in `CLAUDE.md`:

```markdown
## Project Overview
[Brief description]

## Tech Stack
- Language: [TypeScript/Python/etc.]
- Framework: [Express/Django/etc.]
- Testing: [Jest/Pytest/etc.]

## Conventions
[Coding standards, patterns, etc.]
```

## Requirements

### Common Requirements
- Git repository
- GitHub CLI (optional, for PR/issue management)
- Project-specific tooling (see below)

### Foundry Projects
- [Foundry](https://book.getfoundry.sh/getting-started/installation)
- Solidity 0.8.x

### Next.js Projects
- Node.js 18+
- npm/yarn/pnpm
- TypeScript (recommended)

### Permissions (GitHub Actions)

```yaml
permissions:
  contents: write
  pull-requests: write
  issues: write
  id-token: write
```

## Commands Reference

All slash commands **automatically detect your project type** and adapt accordingly.

| Command | Description | Setup Required |
|---------|-------------|----------------|
| `/audit` | Security audit (auto-detects: Solidity, Next.js, Node.js, general) | None |
| `/code-review` | PR code review with best practices | None |
| `/fix-ci` | Auto-fix CI failures (runs tests/build) | Project tools (forge/npm) |
| `/improve-coverage` | Add tests to improve coverage (runs tests) | Project tools (forge/npm) |
| `/check-spec-contradictions` | Find spec inconsistencies | None |

**Analysis commands** (`/audit`, `/code-review`, `/check-spec-contradictions`) work without any setup.  
**Build commands** (`/fix-ci`, `/improve-coverage`) require project build tools to be installed.

## Skills

### Audit Skill

Automatically activates for security-focused tasks:
- **Foundry**: Solidity vulnerability patterns (reentrancy, access control, etc.)
- **Next.js**: Web security (XSS, CSRF, API security)
- **General**: OWASP Top 10, secure coding practices

## Examples

### Example 1: Security Audit (Universal - No Setup Required)

**Local command:**
```bash
/audit
```

**GitHub Actions:**
```yaml
jobs:
  audit:
    uses: teliha/dev-workflows/.github/workflows/security-audit.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

**What it does:**
- Automatically detects project type (Foundry, Next.js, Node.js, etc.)
- Performs static code analysis
- Generates security audit report
- No build tools required

### Example 2: Auto-Fix CI Failures (Foundry)

**GitHub Actions - Composite Action:**
```yaml
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]

jobs:
  auto-fix:
    if: github.event.workflow_run.conclusion == 'failure'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # You setup your build environment
      - uses: foundry-rs/foundry-toolchain@v1
      - run: forge install
      
      # Then use the action
      - uses: teliha/dev-workflows/.github/actions/fix-ci@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          failed_run_id: ${{ github.event.workflow_run.id }}
          failed_branch: ${{ github.event.workflow_run.head_branch }}
```

**What it does:**
- Analyzes CI failure logs
- Fixes code issues using your build environment
- Runs `forge fmt`, `forge build`, `forge test`
- Creates PR with fixes

### Example 3: Improve Coverage (Node.js)

**GitHub Actions - Composite Action:**
```yaml
on:
  schedule:
    - cron: "0 */8 * * *"

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # You setup your build environment
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      
      # Then use the action
      - uses: teliha/dev-workflows/.github/actions/improve-coverage@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          target_coverage_increase: "5"
```

**What it does:**
- Analyzes test coverage using your build environment
- Adds new test cases
- Runs `npm test -- --coverage`
- Creates PR with new tests

## Contributing

Contributions welcome! To add support for new frameworks:

1. Fork the repository
2. Add framework-specific patterns to commands
3. Update documentation
4. Test with sample projects
5. Submit pull request

## Roadmap

- [ ] Python/Django support
- [ ] Rust/Cargo support
- [ ] Go modules support
- [ ] Framework auto-detection
- [ ] Custom command templates
- [ ] Integration with more CI providers

## License

GPL-2.0-or-later

## Support

- Issues: [GitHub Issues](https://github.com/teliha/dev-workflows/issues)
- Documentation: [Claude Code Docs](https://code.claude.com/docs)

## Changelog

### v1.0.0 (Initial Release)
- Security audit command (Foundry, Next.js, general)
- Code review command
- CI auto-fix command
- Coverage improvement command
- Spec contradiction checking
- Audit skill integration
- Reusable GitHub Actions workflows
