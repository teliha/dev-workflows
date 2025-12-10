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

## Reusable GitHub Actions Workflows

This plugin provides reusable GitHub Actions workflows for automated CI/CD across any project type.

### Available Reusable Workflows

#### 1. Security Audit Workflow

```yaml
name: Security Audit

on:
  schedule:
    - cron: "0 3 * * 1"  # Weekly
  workflow_dispatch:

jobs:
  audit:
    uses: teliha/dev-workflows/.github/workflows/security-audit.yml@main
    with:
      foundry_version: nightly  # Optional, for Foundry projects
      create_issue_on_critical: true
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Works with:** Foundry, Next.js, any project type

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
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Paths filter examples:**
- Foundry: `src/**/*.sol`
- Next.js: `src/**/*.{ts,tsx}` or `app/**`
- General: `**/*.{js,ts,py}`

---

#### 3. Auto Fix CI Workflow

```yaml
name: Auto Fix CI

on:
  workflow_run:
    workflows: ["CI"]  # Your CI workflow name
    types: [completed]

jobs:
  auto-fix:
    if: github.event.workflow_run.conclusion == 'failure'
    uses: teliha/dev-workflows/.github/workflows/fix-ci.yml@main
    with:
      foundry_version: nightly  # Optional, for Foundry
      failed_run_id: ${{ github.event.workflow_run.id }}
      failed_branch: ${{ github.event.workflow_run.head_branch }}
      pr_number: ${{ github.event.workflow_run.pull_requests[0].number }}
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

#### 4. Improve Coverage Workflow

```yaml
name: Improve Coverage

on:
  schedule:
    - cron: "0 */8 * * *"
  workflow_dispatch:

jobs:
  coverage:
    uses: teliha/dev-workflows/.github/workflows/improve-coverage.yml@main
    with:
      foundry_version: nightly  # Optional, for Foundry only
      test_match_path: "test/unit/*"  # Customize per project
      target_coverage_increase: "5"
      base_branch: main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Test path examples:**
- Foundry: `test/unit/*`
- Next.js: `__tests__/**`, `src/**/*.test.ts`
- General: Depends on your test structure

---

#### 5. Spec Check Workflow

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
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
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

| Command | Foundry | Next.js | General |
|---------|---------|---------|---------|
| `/audit` | ✅ Smart contract security | ✅ Web security | ✅ Code security |
| `/code-review` | ✅ Solidity best practices | ✅ React/TS patterns | ✅ General review |
| `/fix-ci` | ✅ forge test/build | ✅ npm test/build | ✅ Any CI |
| `/improve-coverage` | ✅ forge coverage | ✅ jest/vitest | ✅ Auto-detect |
| `/check-spec-contradictions` | ✅ | ✅ | ✅ |

## Skills

### Audit Skill

Automatically activates for security-focused tasks:
- **Foundry**: Solidity vulnerability patterns (reentrancy, access control, etc.)
- **Next.js**: Web security (XSS, CSRF, API security)
- **General**: OWASP Top 10, secure coding practices

## Examples

### Example 1: Foundry Project

```bash
# Local audit
/audit

# Output: Detailed report in audits/ContractName_audit_2024-12-10.md
```

### Example 2: Next.js Project

```bash
# Review PR
/code-review

# Output: Comments on PR with security and quality feedback
```

### Example 3: Mixed Monorepo

```bash
# Check specs across all packages
/check-spec-contradictions

# Output: Report identifying cross-package inconsistencies
```

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
