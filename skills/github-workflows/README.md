# GitHub Actions & Reusable Workflows Expert Skill

## Overview

This skill provides expert guidance for creating, reviewing, and maintaining GitHub Actions workflows and reusable workflows with a focus on security, performance, and best practices.

## What This Skill Does

### Automatic Activation

This skill automatically activates when Claude detects:
- Work with `.github/workflows/*.yml` files
- Work with `.github/actions/*/action.yml` files
- Questions about GitHub Actions, CI/CD, or workflows
- Keywords like "workflow", "reusable workflow", "github actions"

### Key Capabilities

1. **Architecture Guidance**
   - Helps decide between universal workflows vs composite actions
   - Prevents over-engineering with conditional project detection
   - Recommends appropriate patterns for different use cases

2. **Security Review**
   - Audits permissions (principle of least privilege)
   - Validates secret handling
   - Checks tool access restrictions in Claude Code Actions
   - Ensures proper token usage (GITHUB_TOKEN vs PAT)

3. **Best Practices Enforcement**
   - Input/output validation and documentation
   - Error handling and reliability patterns
   - Performance optimization (caching, concurrency)
   - Artifact management

4. **Debugging Support**
   - Common issue identification
   - Debug techniques and logging strategies
   - Step-by-step troubleshooting

## Core Principles

### Universal vs Project-Specific

**Universal Workflows** (for analysis tasks):
- No build tools required
- Works across all project types
- Examples: security-audit.yml, code-review.yml, spec-check.yml

**Composite Actions** (for build/test tasks):
- Caller provides build environment
- Universal - works with any project type
- Examples: fix-ci, improve-coverage

### Security First

- ✅ Minimal required permissions
- ✅ Secrets via `secrets:` (not `inputs:`)
- ✅ Restricted bash tool access with `claude_args`
- ✅ Action version pinning
- ❌ Never `permissions: write-all`
- ❌ Never unrestricted bash access

### Simplicity Over Complexity

- ✅ One workflow/action per purpose
- ✅ Caller controls setup (no auto-detection)
- ✅ Clear, explicit configuration
- ❌ No complex conditional logic
- ❌ No project-type detection in workflows

## Usage Examples

### Example 1: Reviewing a Workflow

When you open a workflow file, the skill will automatically analyze:

```yaml
# .github/workflows/security-audit.yml
name: Security Audit

on:
  workflow_call:
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN:
        required: true

permissions:
  contents: read
  issues: write

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          claude_args: '--allowed-tools "Bash(gh issue create:*)"'
          prompt: /audit
```

The skill will check:
- ✅ Minimal permissions (contents: read, issues: write)
- ✅ Restricted tool access (only `gh issue create`)
- ✅ Proper secret handling
- ✅ Universal workflow pattern (no project-specific setup)

### Example 2: Creating a New Workflow

Ask: "I need a workflow to run code reviews on PRs"

The skill will guide you to:
1. Identify this as an analysis task → Universal workflow
2. Define required inputs (pr_number)
3. Set minimal permissions (contents: read, pull-requests: write)
4. Restrict tools to PR operations
5. Structure the prompt properly

### Example 3: Converting Project-Specific to Universal

**Before** (❌ Bad pattern):
```yaml
# Multiple similar workflows
fix-ci-foundry.yml
fix-ci-nodejs.yml
fix-ci-python.yml
```

**After** (✅ Good pattern):
```yaml
# One composite action
.github/actions/fix-ci/action.yml
```

Callers provide their own setup:
```yaml
# Foundry caller
- uses: foundry-rs/foundry-toolchain@v1
- run: forge install
- uses: ./.github/actions/fix-ci

# Node.js caller
- uses: actions/setup-node@v4
- run: npm ci
- uses: ./.github/actions/fix-ci
```

## Security Checklist

The skill enforces this security checklist for every workflow:

- [ ] Minimal required permissions specified
- [ ] Secrets passed via `secrets:` (not `inputs:`)
- [ ] Third-party actions pinned to versions/SHAs
- [ ] `claude_args` restricts bash tool access
- [ ] `GITHUB_TOKEN` used instead of PAT where possible
- [ ] No secrets logged or exposed in outputs

## Common Issues Fixed

### Issue 1: Over-Permissive Workflows

**Problem**:
```yaml
permissions: write-all
```

**Fix**:
```yaml
permissions:
  contents: read
  issues: write
```

### Issue 2: Unrestricted Bash Access

**Problem**:
```yaml
- uses: anthropics/claude-code-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    # No claude_args = unrestricted!
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Fix**:
```yaml
- uses: anthropics/claude-code-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    claude_args: '--allowed-tools "Bash(gh issue create:*),Bash(gh issue list:*)"'
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Issue 4: Using github.token in Composite Actions

**Problem**:
```yaml
# In composite action (.github/actions/*/action.yml)
inputs:
  github_token:
    required: false
    default: ${{ github.token }}  # Does NOT work!
```

**Fix**:
```yaml
# In composite action
inputs:
  github_token:
    description: "GitHub token for API access"
    required: true  # Caller must pass explicitly

# In caller workflow
- uses: ./.github/actions/my-action
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Issue 3: Complex Project Detection

**Problem**:
```yaml
- name: Detect project
  run: |
    if [ -f "foundry.toml" ]; then
      echo "type=foundry" >> $GITHUB_OUTPUT
    elif [ -f "package.json" ]; then
      echo "type=nodejs" >> $GITHUB_OUTPUT
    fi

- name: Setup Foundry
  if: steps.detect.outputs.type == 'foundry'
  uses: foundry-rs/foundry-toolchain@v1
```

**Fix**: Use composite action, let caller provide setup:
```yaml
# Caller's workflow
- uses: foundry-rs/foundry-toolchain@v1  # Caller chooses
- uses: ./.github/actions/universal-action
```

## Integration with Other Skills

This skill works alongside:
- **Audit Skill**: Workflows that trigger security audits
- **Commands**: Workflows that invoke `/audit`, `/fix-ci`, etc.
- **General Development**: CI/CD setup and automation

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Security Hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

## Contributing

When updating this skill:
1. Keep principles simple and actionable
2. Add real-world examples from this repository
3. Update checklists based on common issues
4. Test with actual workflow files
