# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **Claude Code plugin** that provides reusable development workflow commands and GitHub Actions for automated code quality, security auditing, CI/CD fixes, and test coverage improvement. The plugin supports multiple project types including Foundry/Solidity, Next.js/React, and general codebases.

## Architecture

### Plugin Structure

```
.
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata and configuration
├── .github/workflows/        # Reusable GitHub Actions workflows
│   ├── security-audit.yml
│   ├── code-review.yml
│   ├── fix-ci.yml
│   ├── improve-coverage.yml
│   └── spec-check.yml
├── commands/                 # Claude Code slash commands
│   ├── audit.md
│   ├── code-review.md
│   ├── fix-ci.md
│   ├── improve-coverage.md
│   └── check-spec-contradictions.md
└── skills/                   # Auto-activating skills
    └── audit/
        ├── README.md
        └── skill.md         # Smart contract security audit skill

```

### Key Components

1. **Commands** (`commands/*.md`): Markdown files that define slash commands (`/audit`, `/fix-ci`, etc.). These expand into prompts when invoked by users.

2. **Skills** (`skills/*/skill.md`): Auto-activating specialized agents that trigger based on context (e.g., the audit skill activates when reviewing Solidity files).

3. **GitHub Actions Workflows** (`.github/workflows/*.yml`): Reusable workflows that other repositories can call to automate development tasks in CI/CD pipelines.

4. **Plugin Metadata** (`.claude-plugin/plugin.json`): Defines plugin name, version, author, and repository information.

## Command Design Patterns

### Auto-Detection Pattern

All commands follow an **auto-detection pattern** to support multiple project types:

1. Detect project type (Foundry, Next.js, or general)
2. Apply framework-specific checks and commands
3. Generate appropriate reports

Example from `/audit`:
- **Foundry**: Analyzes `.sol` files for smart contract vulnerabilities
- **Next.js**: Checks for XSS, CSRF, API security issues
- **General**: OWASP Top 10 and common security patterns

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

### Reusable Workflow Design

All workflows use `workflow_call` to make them reusable:

```yaml
on:
  workflow_call:
    inputs:
      foundry_version:
        description: 'Foundry version to use'
        required: false
        type: string
        default: 'nightly'
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN:
        required: true
      GITHUB_TOKEN:
        required: true
```

### Common Workflow Pattern

1. **Checkout**: Clone repository with submodules
2. **Setup**: Install Foundry/Node.js as needed
3. **Run Claude Code**: Execute command via `anthropics/claude-code-action@v1`
4. **Post-process**: Parse results, create issues/PRs if needed
5. **Upload artifacts**: Save reports for later review

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

3. Add corresponding GitHub Actions workflow if needed

### Adding New Skills

1. Create `skills/skill-name/skill.md`
2. Define activation triggers in frontmatter
3. Include systematic analysis steps
4. Provide output format specifications

### Modifying Workflows

When updating `.github/workflows/*.yml`:

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

Reference workflows:
```yaml
jobs:
  audit:
    uses: teliha/dev-workflows/.github/workflows/security-audit.yml@main
    with:
      foundry_version: nightly
      create_issue_on_critical: true
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Contribution Guidelines

When modifying this plugin:

1. **Maintain backwards compatibility** in workflow interfaces
2. **Test across project types** (Foundry, Next.js, general)
3. **Document new features** in command files and README
4. **Keep commands focused** - one command, one purpose
5. **Preserve auto-detection** logic for multi-framework support
6. **Update version** in `plugin.json` when releasing changes

## Repository Metadata

- **License**: GPL-2.0-or-later
- **Repository**: https://github.com/teliha/dev-workflows
- **Plugin Name**: dev-workflows
- **Version**: 1.0.0
- **Author**: PREX Trade Team
