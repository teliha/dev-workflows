---
name: GitHub Actions & Reusable Workflows Expert
description: Expert guidance for creating and maintaining GitHub Actions workflows and reusable workflows with security best practices
category: DevOps
tags: [github-actions, ci-cd, workflows, automation, security, reusable-workflows]
---

<!-- GITHUB-WORKFLOWS:START -->

# GitHub Actions & Reusable Workflows Expert Skill

## When to Use This Skill

This skill automatically activates when:
- Creating or modifying `.github/workflows/*.yml` files
- Working with `action.yml` files (composite actions)
- User asks about GitHub Actions, CI/CD, or workflow automation
- User mentions "workflow", "reusable workflow", or "github actions"
- Reviewing or debugging workflow files

## Core Principles

### 1. Workflow Architecture Design

**Decision Tree: What Type of Workflow Should I Create?**

```
Is build/test tooling required?
├─ NO → Create Universal Reusable Workflow
│   └─ Works for ALL project types
│   └─ Example: security-audit.yml, code-review.yml
│
└─ YES → Create Composite Action
    └─ Caller provides build environment
    └─ Example: fix-ci, improve-coverage
```

**Universal Reusable Workflows (Analysis Tasks)**
- ✅ No project-specific setup
- ✅ Works across all project types
- ✅ Only needs repository checkout
- Examples: Security audits, code reviews, spec checking

**Composite Actions (Build/Test Tasks)**
- ✅ Caller controls build environment
- ✅ Runs in caller's job context
- ✅ Universal - works with any setup
- Examples: CI fixing, coverage improvement

### 2. Architecture Patterns to AVOID

❌ **Don't create project-specific variations of the same workflow**
```yaml
# BAD: Multiple similar workflows
fix-ci-foundry.yml
fix-ci-nodejs.yml
fix-ci-python.yml
```

✅ **Instead: Create one composite action, let caller setup environment**
```yaml
# GOOD: One universal action
.github/actions/fix-ci/action.yml  # Used by all project types
```

## Systematic Workflow Review Process

### Step 1: Read Complete Workflow File

**CRITICAL**: You MUST read the entire workflow file before making recommendations.

### Step 2: Workflow Type Classification

Identify what type of workflow this is:

- [ ] **Reusable Workflow** (`workflow_call` trigger)
- [ ] **Direct Workflow** (push/pull_request/schedule triggers)
- [ ] **Composite Action** (`action.yml` in `.github/actions/`)
- [ ] **Event-triggered** (issue_comment, workflow_run, etc.)

### Step 3: Security & Permissions Audit

#### Permission Best Practices

**Principle of Least Privilege**: Only grant permissions that are actually needed.

```yaml
# ✅ GOOD: Minimal required permissions
permissions:
  contents: read        # Only if reading code
  issues: write        # Only if creating/updating issues
  pull-requests: write # Only if commenting on PRs
  id-token: write      # Only if using OIDC

# ❌ BAD: Over-permissive
permissions: write-all
```

#### Secret Handling

**Check for:**
- [ ] Are secrets passed via `secrets:` section (not `inputs:`)?
- [ ] Are sensitive values properly masked in logs?
- [ ] Is `GITHUB_TOKEN` used instead of PAT where possible?
- [ ] Are third-party actions pinned to commit SHA (not tags)?

```yaml
# ✅ GOOD: Secrets are secrets
secrets:
  CLAUDE_CODE_OAUTH_TOKEN:
    required: true
  GH_PAT:
    required: false

# ❌ BAD: Secrets as inputs (logged in plain text!)
inputs:
  api_token:
    required: true
```

#### Token Security

**GITHUB_TOKEN vs Personal Access Token (PAT):**

```yaml
# ✅ GOOD: Use GITHUB_TOKEN when possible (explicit form)
- uses: actions/checkout@v4
  with:
    token: ${{ secrets.GITHUB_TOKEN }}

# ⚠️ USE PAT ONLY WHEN NEEDED: For submodules or cross-repo access
- uses: actions/checkout@v4
  with:
    token: ${{ secrets.GH_PAT || secrets.GITHUB_TOKEN }}
    submodules: recursive
```

**Note on token syntax:**
- **`${{ secrets.GITHUB_TOKEN }}`** - Explicit, recommended for security clarity
- **`${{ github.token }}`** - Context reference (works but less explicit)
- Both are valid, but `secrets.GITHUB_TOKEN` is the official best practice

**When to use PAT:**
- Accessing private submodules
- Triggering workflows from workflows
- Cross-repository operations
- Pushing commits that should trigger workflows

### Step 4: Input & Output Validation

#### Reusable Workflow Inputs

**Required fields for each input:**
```yaml
inputs:
  parameter_name:
    description: "Clear description of what this does"  # REQUIRED
    required: true/false                                 # REQUIRED
    type: string/number/boolean                         # REQUIRED
    default: "value"                                    # Optional
```

**Common Issues:**
- [ ] Missing `description` fields
- [ ] Missing `type` specifications
- [ ] No sensible defaults for optional inputs
- [ ] Boolean inputs without default values

```yaml
# ✅ GOOD: Well-defined inputs
inputs:
  node_version:
    description: "Node.js version to use"
    required: false
    type: string
    default: "20"
  
  create_issue:
    description: "Create GitHub issue on failure"
    required: false
    type: boolean
    default: true

# ❌ BAD: Poorly defined inputs
inputs:
  version:
    required: false  # Missing description and type!
```

#### Composite Action Inputs

**For composite actions, inputs work differently:**
```yaml
# In action.yml
inputs:
  failed_run_id:
    description: "ID of the failed workflow run"
    required: true
  
  github_token:
    description: "GitHub token for API access"
    required: true  # Must be passed from caller
  
  max_attempts:
    description: "Maximum fix attempts"
    required: false
    default: "3"

# Access via ${{ inputs.parameter_name }}
runs:
  using: "composite"
  steps:
    - run: echo "Failed run: ${{ inputs.failed_run_id }}"
      shell: bash
      env:
        GITHUB_TOKEN: ${{ inputs.github_token }}
```

**IMPORTANT**: Composite actions **cannot** use `${{ github.token }}` or `${{ secrets.* }}` directly.
- ❌ `default: ${{ github.token }}` - Does NOT work in composite actions
- ✅ Caller must pass token explicitly as input
- ✅ Use `${{ inputs.github_token }}` inside the composite action

### Step 5: Job & Step Structure Analysis

#### Job Configuration

**Check for:**
- [ ] Appropriate `runs-on` value (ubuntu-latest for universal workflows)
- [ ] Proper `needs:` dependencies between jobs
- [ ] Appropriate `timeout-minutes` (default 360min often too long)
- [ ] Conditional execution with `if:` where needed

```yaml
# ✅ GOOD: Explicit timeout and conditions
jobs:
  audit:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    if: github.event_name == 'pull_request'
```

#### Step Best Practices

**Action Version Pinning:**
```yaml
# ✅ GOOD: Pin to major version (gets patches)
- uses: actions/checkout@v4

# ✅ BETTER: Pin to exact SHA for security-critical workflows
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1

# ❌ BAD: No version specified
- uses: actions/checkout
```

**Step Naming:**
```yaml
# ✅ GOOD: Clear, descriptive names
- name: Install Foundry toolchain
  uses: foundry-rs/foundry-toolchain@v1

- name: Run security audit with Claude
  uses: anthropics/claude-code-action@v1

# ❌ BAD: Generic or missing names
- uses: foundry-rs/foundry-toolchain@v1  # No name

- name: Run action  # Too generic
  uses: anthropics/claude-code-action@v1
```

### Step 6: Claude Code Action Integration

#### Required Parameters

```yaml
- name: Run Claude Code Action
  uses: anthropics/claude-code-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}  # REQUIRED
    prompt: |                                                         # REQUIRED
      Your detailed prompt here
    claude_args: '--allowed-tools "..."'                             # Optional but recommended
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}                        # For gh CLI
```

#### Tool Restrictions with claude_args

**Security Best Practice**: Restrict bash commands to only what's needed.

```yaml
# ✅ GOOD: Minimal tool access for audit workflow
claude_args: '--allowed-tools "Bash(gh issue create:*),Bash(gh issue list:*)"'

# ✅ GOOD: Read-only access for spec checking
claude_args: '--allowed-tools "Read,Write,Glob,Grep,Bash(gh:*),Bash(find:*),Bash(cat:*),Bash(date:*)"'

# ✅ GOOD: PR review access
claude_args: '--allowed-tools "Bash(gh pr comment:*),Bash(gh pr diff:*),Bash(gh pr view:*)"'

# ❌ BAD: No restrictions (Claude can run arbitrary bash)
# (omitting claude_args entirely)
```

**Tool restriction patterns:**
- `Bash(gh:*)` - All gh CLI commands
- `Bash(gh issue create:*)` - Only creating issues
- `Bash(git:*)` - All git commands
- `Read,Write,Glob,Grep` - File operations only
- `mcp__github_inline_comment__create_inline_comment` - Inline PR comments

#### Prompt Design for Workflows

**Effective prompts include:**
1. **Command to run**: `/audit`, `/fix-ci`, etc.
2. **Context variables**: PR number, repo name, run ID
3. **Specific instructions**: What to analyze, where to save output
4. **Success criteria**: What to do after completion
5. **Error handling**: What to do on failure

```yaml
# ✅ GOOD: Comprehensive prompt
prompt: |
  /audit

  Please perform a comprehensive security audit.
  
  Generate a detailed audit report and save it to `audit-report-$(date +%Y%m%d).md`.

  After completing the audit:
  1. If you find any CRITICAL severity issues, create a GitHub issue using the `gh` CLI
  2. Include link to this workflow run: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
  3. Label with: security, critical, audit

# ❌ BAD: Vague prompt
prompt: "Please audit the code"
```

### Step 7: Error Handling & Reliability

#### Conditional Steps

```yaml
# ✅ GOOD: Upload artifacts even on failure
- name: Upload audit report
  if: always()  # Run even if previous steps failed
  uses: actions/upload-artifact@v4

# ✅ GOOD: Only run on success
- name: Post success comment
  if: success()
  run: gh pr comment ${{ inputs.pr_number }} --body "✅ Audit passed!"

# ✅ GOOD: Only run on failure
- name: Create failure issue
  if: failure()
  run: gh issue create --title "Workflow failed" --label bug
```

#### Artifact Management

**Best practices:**
```yaml
- name: Upload artifacts
  uses: actions/upload-artifact@v4
  with:
    name: audit-report-${{ github.run_number }}  # Unique name with run number
    path: |                                       # Multiple paths supported
      audit-report-*.md
      audits/**/*.md
    retention-days: 90                           # Don't keep forever
    if-no-files-found: warn                      # Don't fail if missing
```

### Step 8: Performance Optimization

#### Caching Strategies

```yaml
# ✅ GOOD: Cache dependencies
- name: Cache Node modules
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

# ✅ GOOD: Cache Foundry installation
- name: Cache Foundry
  uses: actions/cache@v4
  with:
    path: ~/.foundry
    key: ${{ runner.os }}-foundry-${{ hashFiles('foundry.toml') }}
```

#### Concurrency Control

```yaml
# ✅ GOOD: Cancel in-progress runs for PR updates
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# ⚠️ USE CAREFULLY: For deployments, don't cancel
concurrency:
  group: production-deploy
  cancel-in-progress: false
```

## Common Workflow Patterns

### Pattern 1: Universal Analysis Workflow

**Use case**: Security audits, code reviews, spec checking

```yaml
name: Security Audit

on:
  workflow_call:
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN:
        required: true

permissions:
  contents: read
  issues: write
  id-token: write

jobs:
  audit:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: recursive
          token: ${{ secrets.GH_PAT || github.token }}

      - name: Run Claude Code Audit
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          claude_args: '--allowed-tools "Bash(gh issue create:*),Bash(gh issue list:*)"'
          prompt: |
            /audit
            [Detailed instructions...]
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Upload audit report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: audit-report-${{ github.run_number }}
          path: audit-report-*.md
          retention-days: 90
```

### Pattern 2: Composite Action (Universal Build/Test)

**Use case**: CI fixes, coverage improvement

```yaml
# .github/actions/fix-ci/action.yml
name: 'Fix CI Failures'
description: 'Automatically fix CI failures using Claude Code'

inputs:
  failed_run_id:
    description: 'ID of the failed workflow run'
    required: true
  
  claude_code_oauth_token:
    description: 'Claude Code OAuth token'
    required: true

runs:
  using: "composite"
  steps:
    - name: Run Claude Code to fix CI
      uses: anthropics/claude-code-action@v1
      with:
        claude_code_oauth_token: ${{ inputs.claude_code_oauth_token }}
        prompt: |
          /fix-ci
          
          Failed run ID: ${{ inputs.failed_run_id }}
          
          [Instructions for fixing CI...]
      shell: bash
```

**Caller workflow (Foundry project):**
```yaml
name: Auto-fix CI (Foundry)

on:
  workflow_run:
    workflows: ["Tests"]
    types: [completed]

permissions:
  contents: write
  pull-requests: write

jobs:
  fix-ci:
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      # Caller provides Foundry setup
      - uses: foundry-rs/foundry-toolchain@v1
        with:
          version: nightly
      
      - run: forge install
      
      # Universal action runs in this environment
      - uses: ./.github/actions/fix-ci
        with:
          failed_run_id: ${{ github.event.workflow_run.id }}
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          github_token: ${{ secrets.GITHUB_TOKEN }}  # Must pass explicitly
```

### Pattern 3: PR Review Workflow

```yaml
name: Code Review

on:
  workflow_call:
    inputs:
      pr_number:
        description: "Pull request number"
        required: true
        type: string
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN:
        required: true

permissions:
  contents: read
  pull-requests: write

jobs:
  review:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Review PR
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          claude_args: '--allowed-tools "Bash(gh pr:*),mcp__github_inline_comment__create_inline_comment"'
          prompt: |
            /code-review
            
            PR #${{ inputs.pr_number }}
            Repository: ${{ github.repository }}
            
            Review the changes and post inline comments using gh CLI or MCP tools.
```

## Checklist for Workflow Review

When reviewing a workflow, systematically check:

### Security
- [ ] Minimal required permissions specified
- [ ] Secrets passed via `secrets:` (not `inputs:`)
- [ ] Third-party actions pinned to versions/SHAs
- [ ] `claude_args` restricts bash tool access
- [ ] `GITHUB_TOKEN` used instead of PAT where possible
- [ ] No secrets logged or exposed in outputs

### Architecture
- [ ] Correct workflow type for the task (universal vs composite action)
- [ ] No unnecessary project-specific conditionals
- [ ] Clear separation: analysis (universal) vs build (composite)
- [ ] Reusable where appropriate

### Inputs/Outputs
- [ ] All inputs have `description`, `required`, `type`
- [ ] Sensible defaults for optional inputs
- [ ] Secrets properly typed as secrets
- [ ] Output artifacts properly named and retained

### Reliability
- [ ] Appropriate timeout values
- [ ] `if: always()` for artifact uploads
- [ ] Error handling for expected failures
- [ ] Concurrency control if needed

### Performance
- [ ] Caching for dependencies
- [ ] Minimal required steps
- [ ] Parallel jobs where possible
- [ ] Appropriate runner size

### Maintainability
- [ ] Clear step names
- [ ] Well-documented prompts
- [ ] Consistent naming conventions
- [ ] Comments for complex logic

## Common Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: Project Detection in Workflows

```yaml
# BAD: Conditional setup based on project detection
- name: Detect project type
  id: detect
  run: |
    if [ -f "foundry.toml" ]; then
      echo "type=foundry" >> $GITHUB_OUTPUT
    elif [ -f "package.json" ]; then
      echo "type=nodejs" >> $GITHUB_OUTPUT
    fi

- name: Setup Foundry
  if: steps.detect.outputs.type == 'foundry'
  uses: foundry-rs/foundry-toolchain@v1

- name: Setup Node
  if: steps.detect.outputs.type == 'nodejs'
  uses: actions/setup-node@v4
```

**Why it's bad:**
- Complex conditional logic
- Hard to maintain and extend
- Slower execution (evaluates all conditions)

**Better approach:**
- Analysis tasks: Make universal (no setup needed)
- Build tasks: Use composite action, caller provides setup

### ❌ Anti-Pattern 2: Hardcoded Values

```yaml
# BAD: Hardcoded versions and paths
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
  with:
    node-version: 18  # Hardcoded!
- run: npm ci
  working-directory: ./frontend  # Hardcoded path!
```

**Better:**
```yaml
# GOOD: Parameterized
inputs:
  node_version:
    description: "Node.js version"
    type: string
    default: "20"
  
  working_directory:
    description: "Working directory"
    type: string
    default: "."

# Use in steps
- uses: actions/setup-node@v4
  with:
    node-version: ${{ inputs.node_version }}
- run: npm ci
  working-directory: ${{ inputs.working_directory }}
```

### ❌ Anti-Pattern 3: Overly Permissive Access

```yaml
# BAD: Too many permissions
permissions: write-all

# BAD: Unrestricted bash access
- uses: anthropics/claude-code-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    # No claude_args = unrestricted bash
```

**Better:**
```yaml
# GOOD: Minimal permissions
permissions:
  contents: read
  issues: write

# GOOD: Restricted tool access
- uses: anthropics/claude-code-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    claude_args: '--allowed-tools "Bash(gh issue create:*)"'
```

### ❌ Anti-Pattern 4: No Error Handling

```yaml
# BAD: No artifact upload on failure
- name: Run tests
  run: npm test

- name: Upload coverage  # Never runs if tests fail!
  uses: actions/upload-artifact@v4
  with:
    name: coverage
    path: coverage/
```

**Better:**
```yaml
# GOOD: Always upload artifacts
- name: Run tests
  run: npm test
  continue-on-error: true  # Or remove this to fail workflow

- name: Upload coverage
  if: always()  # Always run, even on failure
  uses: actions/upload-artifact@v4
  with:
    name: coverage
    path: coverage/
    if-no-files-found: warn
```

## Debugging Workflows

### Common Issues and Solutions

**Issue: "Resource not accessible by integration"**
- Missing required permission in `permissions:`
- Solution: Add the specific permission needed

**Issue: "Secret not found"**
- Secret not passed from caller to reusable workflow
- Solution: Ensure caller passes secret explicitly

**Issue: Claude Code Action timeout**
- Task too complex or repository too large
- Solution: Increase `timeout-minutes`, break into smaller tasks

**Issue: Artifacts not uploaded**
- Path doesn't match any files
- Step didn't run (missing `if: always()`)
- Solution: Check paths, add conditional execution

**Issue: "Invalid workflow file"**
- YAML syntax error
- Missing required fields
- Solution: Validate with `yamllint` or GitHub's schema

### Debug Techniques

```yaml
# Enable debug logging
- name: Debug information
  run: |
    echo "Event name: ${{ github.event_name }}"
    echo "Ref: ${{ github.ref }}"
    echo "Repository: ${{ github.repository }}"
    echo "Runner OS: ${{ runner.os }}"
    echo "Working directory: $(pwd)"
    ls -la

# Add step summaries
- name: Generate summary
  run: |
    echo "## Audit Results" >> $GITHUB_STEP_SUMMARY
    echo "- Critical issues: 2" >> $GITHUB_STEP_SUMMARY
    echo "- High issues: 5" >> $GITHUB_STEP_SUMMARY
```

## Output Format for Workflow Reviews

When reviewing a workflow, provide feedback in this structure:

```markdown
# Workflow Review: [workflow-name.yml]

**Type**: [Universal Reusable Workflow / Composite Action / Direct Workflow]
**Purpose**: [Brief description]

---

## ✅ Strengths

- [What the workflow does well]
- [Good patterns observed]

---

## ⚠️ Issues Found

### 🔴 Critical

**Issue**: [Description]
**Location**: `line X-Y`
**Impact**: [What could go wrong]
**Fix**:
```yaml
# Current (problematic)
[current code]

# Recommended
[fixed code]
```

### 🟡 Improvements

**Suggestion**: [Description]
**Benefit**: [Why this improves the workflow]
**Implementation**:
```yaml
[suggested code]
```

---

## 📋 Checklist Results

- [x] Security: Minimal permissions
- [ ] Security: Tool access restricted (MISSING)
- [x] Inputs: Well-documented
- [x] Error handling: Artifacts uploaded on failure
- [ ] Performance: Caching configured (OPTIONAL)

---

## 🎯 Recommended Actions

1. [Prioritized list of changes]
2. [Most important fixes first]
3. [Nice-to-have improvements last]

---

## 📚 Additional Notes

[Any context-specific guidance or references]
```

<!-- GITHUB-WORKFLOWS:END -->
