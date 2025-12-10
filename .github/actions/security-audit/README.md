# Security Audit Composite Action

Perform comprehensive security audits using Claude Code. This composite action allows callers to control their build environment and dependencies while leveraging Claude's security analysis capabilities.

## When to Use

Use this composite action when:
- You need to run security audits with custom build tools installed
- You want to combine the audit with other project-specific setup
- You need fine-grained control over the execution environment

For simple security audits without custom setup, use the [reusable workflow](../../workflows/security-audit.yml) instead.

## Usage

### Basic Usage (Foundry Project)

```yaml
name: Security Audit

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday

permissions:
  contents: read
  issues: write
  id-token: write

jobs:
  audit:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: recursive
      
      # Setup Foundry toolchain
      - name: Install Foundry
        uses: foundry-rs/foundry-toolchain@v1
        with:
          version: nightly
      
      - name: Install dependencies
        run: forge install
      
      # Run security audit
      - name: Run security audit
        uses: teliha/dev-workflows/.github/actions/security-audit@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced Usage (Node.js Project)

```yaml
name: Security Audit

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read
  issues: write

jobs:
  audit:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      # Setup Node.js
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      
      # Run security audit
      - uses: teliha/dev-workflows/.github/actions/security-audit@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          create_issue_on_critical: true
          report_retention_days: 180
```

### With Private Submodules

```yaml
steps:
  - uses: actions/checkout@v4
    with:
      submodules: recursive
      token: ${{ secrets.GH_PAT }}  # PAT with submodule access
  
  - uses: foundry-rs/foundry-toolchain@v1
  - run: forge install
  
  - uses: teliha/dev-workflows/.github/actions/security-audit@main
    with:
      claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `claude_code_oauth_token` | Claude Code OAuth token for authentication | Yes | - |
| `github_token` | GitHub token for API access (creating issues, etc.) | Yes | - |
| `create_issue_on_critical` | Create GitHub issue if critical findings are discovered | No | `true` |
| `report_retention_days` | Number of days to retain audit reports as artifacts | No | `90` |

## Outputs

This action uploads artifacts:
- **Name**: `security-audit-report-{run_number}`
- **Contents**:
  - `audit-report-YYYYMMDD.md` - Main audit report
  - `audits/**/*.md` - Contract-specific audit reports (if applicable)
- **Retention**: Configurable via `report_retention_days` input

## Required Permissions

```yaml
permissions:
  contents: read        # To checkout code
  issues: write        # To create issues on critical findings (if enabled)
  id-token: write      # For OIDC authentication (if used)
```

## Required Secrets

### CLAUDE_CODE_OAUTH_TOKEN

Generate this token from your Claude account:
1. Visit https://claude.ai/settings
2. Navigate to API section
3. Create new OAuth token for GitHub Actions
4. Add to repository secrets as `CLAUDE_CODE_OAUTH_TOKEN`

### GITHUB_TOKEN

The default `GITHUB_TOKEN` is automatically available in GitHub Actions. Pass it explicitly:

```yaml
github_token: ${{ secrets.GITHUB_TOKEN }}
```

For cross-repository operations or private submodules, use a Personal Access Token:

```yaml
github_token: ${{ secrets.GH_PAT }}
```

## Features

- ✅ **Auto-detection**: Automatically detects project type (Solidity, JavaScript, Rust, etc.)
- ✅ **Comprehensive analysis**: Checks for security vulnerabilities, code quality issues, and best practices
- ✅ **Structured reports**: Generates detailed Markdown reports with severity levels
- ✅ **Issue creation**: Optionally creates GitHub issues for critical findings
- ✅ **Artifact upload**: Reports are always uploaded as artifacts for review
- ✅ **Step summary**: Provides a preview of findings in the GitHub Actions summary

## Example Report Structure

```markdown
# Smart Contract Security Audit Report

**Contract**: VaultOperator
**Date**: 2024-01-15
**Auditor**: Claude Code Audit Skill

## Executive Summary

- **Overall Risk Level**: MEDIUM
- **Total Issues Found**: 7
  - Critical: 0
  - High: 2
  - Medium: 3
  - Low/Info: 2

## Findings

### [HIGH] Finding #1: Reentrancy Risk
**Location**: `src/VaultOperator.sol:142`
**Impact**: Potential fund drainage...
**Recommendation**: Implement checks-effects-interactions pattern...

[...]
```

## Comparison: Composite Action vs Reusable Workflow

| Feature | Composite Action | Reusable Workflow |
|---------|------------------|-------------------|
| **Build tools** | Caller provides | Not available |
| **Setup control** | Full control | Limited |
| **Simplicity** | More steps | One step |
| **Use case** | Custom environment | Standard audits |
| **Dependencies** | Caller installs | None needed |

**When to use each:**

- **Composite Action** (this): When you need custom build tools or dependencies
- **Reusable Workflow**: For simple, zero-setup security audits

## Troubleshooting

### "Resource not accessible by integration"

Add required permissions to your workflow:

```yaml
permissions:
  contents: read
  issues: write
```

### "No audit report generated"

Ensure Claude has access to your code and dependencies are installed before running the action.

### "CLAUDE_CODE_OAUTH_TOKEN not found"

Add the secret to your repository settings or organization secrets.

## Related

- [Reusable Workflow Version](../../workflows/security-audit.yml)
- [Code Review Action](../code-review/)
- [Fix CI Action](../fix-ci/)
- [Improve Coverage Action](../improve-coverage/)

## License

GPL-2.0-or-later
