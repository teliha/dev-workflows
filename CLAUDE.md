# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **Claude Code plugin** that provides reusable development workflow commands and GitHub Actions for automated code quality, security auditing, CI/CD fixes, and test coverage improvement. The plugin supports multiple project types including Foundry/Solidity, Next.js/React, and general codebases.

## Architecture

### Plugin Structure

```
.
├── .claude-plugin/
│   └── plugin.json                # Plugin metadata and configuration
├── .github/
│   ├── workflows/                 # Reusable workflows (no build required)
│   │   ├── security-audit.yml     # Universal security audit
│   │   ├── code-review.yml        # Universal code review
│   │   └── spec-check.yml         # Universal spec validation
│   └── actions/                   # Composite actions (build required)
│       ├── fix-ci/
│       │   └── action.yml         # Universal CI fix (caller provides build env)
│       └── improve-coverage/
│           └── action.yml         # Universal coverage improvement (caller provides build env)
├── commands/                      # Claude Code slash commands (auto-detect in command)
│   ├── audit.md
│   ├── code-review.md
│   ├── fix-ci.md
│   ├── improve-coverage.md
│   └── check-spec-contradictions.md
└── skills/                        # Auto-activating skills
    └── audit/
        ├── README.md
        └── skill.md               # Smart contract security audit skill

```

### Key Components

1. **Commands** (`commands/*.md`): Markdown files that define slash commands (`/audit`, `/fix-ci`, etc.). These expand into prompts when invoked by users.

2. **Skills** (`skills/*/skill.md`): Auto-activating specialized agents that trigger based on context (e.g., the audit skill activates when reviewing Solidity files).

3. **GitHub Actions Workflows** (`.github/workflows/*.yml`): Reusable workflows that other repositories can call to automate development tasks in CI/CD pipelines.

4. **Plugin Metadata** (`.claude-plugin/plugin.json`): Defines plugin name, version, author, and repository information.

## Architecture Design

### Commands vs Workflows vs Composite Actions

**Commands** (slash commands like `/audit`, `/fix-ci`):
- Auto-detect project type internally
- Claude decides what to do based on codebase
- Used locally via Claude Code CLI or in Claude Code Action

**Reusable Workflows** (for analysis tasks):
- No build tools required
- Work universally across all project types
- Examples: `security-audit.yml`, `code-review.yml`

**Composite Actions** (for build/test tasks):
- Caller provides the build environment
- Action runs in the caller's environment
- Universal - works with any project type
- Examples: `fix-ci`, `improve-coverage`

### Why Composite Actions for Build Tasks?

**Previous approach (project-specific workflows):**
```yaml
# fix-ci-foundry.yml
- uses: foundry-rs/foundry-toolchain@v1
- run: forge install
- uses: claude-code-action@v1

# fix-ci-nodejs.yml
- uses: actions/setup-node@v4
- run: npm ci
- uses: claude-code-action@v1
```
❌ Duplicate workflows for each project type

**New approach (composite action):**
```yaml
# Caller's workflow (Foundry)
- uses: foundry-rs/foundry-toolchain@v1
- run: forge install
- uses: teliha/dev-workflows/.github/actions/fix-ci@main

# Caller's workflow (Node.js)
- uses: actions/setup-node@v4
- run: npm ci
- uses: teliha/dev-workflows/.github/actions/fix-ci@main
```
✅ One universal action, caller controls setup

### Benefits

1. **Caller controls the build environment** - You decide which versions, tools, etc.
2. **Truly universal** - Works with any project type (Foundry, Node.js, Python, Go, Rust...)
3. **Simpler maintenance** - One action instead of N workflows
4. **Flexible** - Caller can add custom setup steps before the action
5. **No duplication** - DRY principle

### Command Structure

Each command file (`commands/*.md`) includes:

- **Front matter**: Description field for CLI display
- **Auto-detection logic**: How the command detects project type
- **Task instructions**: Step-by-step guidance for Claude
- **Output format**: Expected report structure
- **Examples**: Code examples showing vulnerabilities and fixes
- **Integration**: How to use in CI/CD

### Critical Rules for Commands

Commands must follow these principles:

1. **Minimal changes**: Only fix what's broken, don't refactor unrelated code
2. **No feature creep**: Don't add features beyond what's requested
3. **Security first**: Never introduce security vulnerabilities
4. **Verification**: Always verify fixes work before committing
5. **Clear output**: Generate structured reports with actionable recommendations

## GitHub Actions Workflow Patterns

### Two Workflow Categories

**1. Universal Workflows (No Setup)**

These workflows work for all project types without build tools:

```yaml
# security-audit.yml
on:
  workflow_call:
    inputs:
      create_issue_on_critical:
        required: false
        type: boolean
        default: true
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN:
        required: true
      GITHUB_TOKEN:
        required: true

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          prompt: /audit
```

**No detection, no conditional setup** - just checkout and run Claude.

**2. Project-Specific Workflows (With Setup)**

These workflows require build tools and are separated by project type:

```yaml
# fix-ci-foundry.yml
on:
  workflow_call:
    inputs:
      foundry_version:
        required: false
        type: string
        default: 'nightly'
      failed_run_id:
        required: true
        type: string

jobs:
  auto-fix:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: foundry-rs/foundry-toolchain@v1
        with:
          version: ${{ inputs.foundry_version }}
      - run: forge install
      - uses: anthropics/claude-code-action@v1
```

```yaml
# fix-ci-nodejs.yml
on:
  workflow_call:
    inputs:
      node_version:
        required: false
        type: string
        default: '20'

jobs:
  auto-fix:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node_version }}
      - run: npm ci
      - uses: anthropics/claude-code-action@v1
```

**No conditionals needed** - each workflow knows exactly what to setup.

### Common Workflow Patterns

**Universal Workflows:**
1. Checkout repository
2. Run Claude Code Action with command
3. Post-process results (parse audit reports, etc.)
4. Upload artifacts

**Project-Specific Workflows:**
1. Checkout repository
2. Setup project-specific tooling (Foundry or Node.js)
3. Install dependencies (`forge install` or `npm ci`)
4. Run Claude Code Action with command
5. Post-process results
6. Upload artifacts

**Key difference:** Project-specific workflows have dedicated setup steps, no conditionals needed.

### Authentication Requirements

- `CLAUDE_CODE_OAUTH_TOKEN`: For Claude Code API access
- `GITHUB_TOKEN`: For GitHub API operations (issues, PRs, comments)

## Skills System

### Audit Skill (`skills/audit/skill.md`)

The audit skill is a **context-aware agent** that automatically activates when:
- Reviewing `.sol` files
- User mentions "audit" or "security"
- User requests code review of smart contracts

**Key Features:**
- Systematic vulnerability scanning across 10+ categories
- Project-specific pattern recognition (EVK/EVC integration)
- Generates structured audit reports in `audits/` directory
- Distinguishes legitimate patterns from vulnerabilities

**Critical Context (EVK/EVC Projects):**
- `callThroughEVC` modifier is a legitimate auth pattern, not a vulnerability
- `evc.call(vault, account, 0, calldata)` is the correct way to perform vault operations
- Single swap per operation is a design requirement, not a limitation

## Development Workflow

### Adding New Commands

1. Create `commands/new-command.md`:
   ```markdown
   ---
   description: Brief description for CLI
   ---
   
   # Command Name
   
   [Detailed instructions for Claude...]
   ```

2. Test locally by invoking `/new-command` in a project

3. Decide if a workflow is needed:
   - **Analysis only?** → Create one universal workflow
   - **Needs build tools?** → Create separate workflows per project type

### Adding New Skills

1. Create `skills/skill-name/skill.md`
2. Define activation triggers in frontmatter
3. Include systematic analysis steps
4. Provide output format specifications

### Adding Support for New Project Types

**For analysis workflows (like security-audit.yml):**
- No changes needed - they already work universally

**For build workflows (like fix-ci, improve-coverage):**
1. Create new workflow file: `fix-ci-[projecttype].yml`
2. Add setup steps for that project type
3. Install dependencies
4. Run Claude Code Action
5. Update README with usage example

Example for Python:
```yaml
# fix-ci-python.yml
- uses: actions/checkout@v4
- uses: actions/setup-python@v4
  with:
    python-version: '3.11'
- run: pip install -r requirements.txt
- uses: anthropics/claude-code-action@v1
```

### Modifying Workflows

When updating `.github/workflows/*.yml`:

- **Universal workflows**: Keep them simple, no conditional logic
- **Project-specific workflows**: Keep them focused on one project type
- Maintain `workflow_call` interface for backwards compatibility
- Keep input parameters with sensible defaults
- Document all inputs in description fields
- Test with sample repositories before releasing

## Testing Strategy

### Local Testing

Test commands in various project types:

```bash
# In a Foundry project
/audit
/improve-coverage

# In a Next.js project
/audit
/fix-ci

# In any project
/check-spec-contradictions
```

### CI/CD Testing

Reference workflows from test repositories:

```yaml
jobs:
  audit:
    uses: teliha/dev-workflows/.github/workflows/security-audit.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Project-Specific Patterns

### Foundry/Solidity Projects

Commands should:
- Run `forge fmt` before committing
- Use `forge test -vvv` for detailed output
- Generate coverage with `forge coverage --report summary`
- Check for Solidity-specific vulnerabilities (reentrancy, access control, etc.)

### Next.js Projects

Commands should:
- Check `next.config.js` for build settings
- Verify environment variables
- Run `npm run build` to catch build-time errors
- Check for web security issues (XSS, CSRF, etc.)

### EVK/EVC Integration (DeFi)

Legitimate patterns to recognize (NOT vulnerabilities):
- `callThroughEVC` modifier for authentication
- `evc.getCurrentOnBehalfOfAccount(address(0))` for user identification
- `evc.requireAccountStatusCheck(account)` after position changes
- Single swap operations (design requirement)

## Security Considerations

### Audit Severity Levels

- **CRITICAL**: Immediate security risk (reentrancy, access control bypass)
- **HIGH**: Significant vulnerability (unchecked calls, flash loan attacks)
- **MEDIUM**: Moderate risk (gas issues, missing validation)
- **LOW/INFO**: Best practices, optimization opportunities

### Safe Patterns

When fixing CI or improving coverage:
- Never modify implementation code (coverage improvement)
- Only fix what's broken (CI fixes)
- Maintain existing security patterns
- Verify all tests pass before committing

## Common Commands

### Running Commands Locally

```bash
# Security audit
/audit

# Review pull request
/code-review

# Fix CI failures
/fix-ci

# Improve test coverage
/improve-coverage

# Check specification contradictions
/check-spec-contradictions
```

### Verification Commands

**Foundry:**
```bash
forge fmt --check
forge build
forge test
forge coverage
```

**Next.js:**
```bash
npm run lint
npm run type-check
npm test
npm run build
```

## Output Conventions

### Audit Reports

Save to: `audits/[ContractName]_audit_[YYYY-MM-DD].md`

Structure:
- Executive Summary (risk level, issue count)
- Findings (severity, location, impact, fix)
- Gas Optimizations
- Code Quality Issues
- Architecture Review
- Conclusion

### Coverage Reports

Include:
- Current coverage percentage
- New tests added
- Coverage increase achieved
- Files modified

### CI Fix Reports

Document:
- Errors encountered
- Fixes applied
- Verification results
- Commands used

## Integration Examples

### Using in Another Repository

Add to `.claude/settings.json`:
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

### Using GitHub Actions

**Universal workflows (no project type needed):**
```yaml
jobs:
  audit:
    uses: teliha/dev-workflows/.github/workflows/security-audit.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Project-specific workflows (choose the right one):**
```yaml
# For Foundry projects
jobs:
  fix-ci:
    uses: teliha/dev-workflows/.github/workflows/fix-ci-foundry.yml@main
    with:
      failed_run_id: ${{ github.event.workflow_run.id }}
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}

# For Node.js projects
jobs:
  fix-ci:
    uses: teliha/dev-workflows/.github/workflows/fix-ci-nodejs.yml@main
    with:
      failed_run_id: ${{ github.event.workflow_run.id }}
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

## Contribution Guidelines

When modifying this plugin:

1. **Maintain backwards compatibility** in workflow interfaces
2. **Follow the separation principle**:
   - Analysis workflows → Universal (no build tools)
   - Build/test workflows → Project-specific (dedicated setup)
3. **Test with real projects** (Foundry, Next.js, etc.)
4. **Document new features** in command files and README
5. **Keep workflows simple** - avoid complex conditional logic
6. **Update version** in `plugin.json` when releasing changes

### Adding New Project Type Support

**Don't modify existing workflows** - create new ones:
- ✅ Create `fix-ci-python.yml` for Python support
- ❌ Don't add Python conditionals to `fix-ci-foundry.yml`

This keeps workflows maintainable and fast.

## Repository Metadata

- **License**: GPL-2.0-or-later
- **Repository**: https://github.com/teliha/dev-workflows
- **Plugin Name**: dev-workflows
- **Version**: 1.0.0
- **Author**: PREX Trade Team
