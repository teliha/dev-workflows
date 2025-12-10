---
description: Run comprehensive security audit
---

# Audit Command

Run a comprehensive security audit of your codebase.

## Auto-Detection

This command automatically detects your project type and applies appropriate security checks:

### Foundry/Solidity Projects
- Smart contract vulnerability analysis
- DeFi-specific pattern checks
- Integration safety (EVK, Uniswap, etc.)
- Uses the `audit` skill for deep analysis

### Next.js/React Projects
- XSS (Cross-Site Scripting) vulnerabilities
- CSRF (Cross-Site Request Forgery) protection
- API security (authentication, authorization)
- Server-side rendering security
- Environment variable exposure
- Dependency vulnerabilities

### General Projects
- OWASP Top 10 vulnerabilities
- Code injection risks
- Authentication/Authorization issues
- Insecure dependencies
- Sensitive data exposure

## Security Categories

### 1. Authentication & Authorization
- Proper access control checks
- Session management
- JWT validation
- API key protection

### 2. Input Validation
- User input sanitization
- SQL/NoSQL injection prevention
- Command injection prevention
- Path traversal protection

### 3. Data Security
- Sensitive data encryption
- Secure storage practices
- Environment variable usage
- API key and secret management

### 4. API Security
- Rate limiting
- CORS configuration
- Request validation
- Response sanitization

### 5. Dependencies
- Known vulnerability scanning
- Outdated packages
- Malicious dependencies

### 6. Framework-Specific

**Solidity:**
- Reentrancy
- Integer overflow/underflow
- Access control
- External call safety
- Flash loan attacks

**Next.js:**
- Server component security
- API route protection
- Middleware authorization
- Client-side data exposure
- Hydration mismatches

## Usage

Simply run:
```bash
/audit
```

The audit will:
1. Detect your project type
2. Analyze all relevant source files
3. Check for security vulnerabilities
4. Generate a detailed report
5. Create issues for critical findings (optional)

## Output

The audit generates a report: `audits/[ProjectName]_audit_[DATE].md`

Report includes:
- **Executive Summary**: Overall risk level and issue count
- **Findings**: Detailed vulnerabilities with severity
- **Recommendations**: Specific fixes for each issue
- **Code Examples**: Vulnerable code and suggested fixes

## Severity Levels

- **CRITICAL**: Immediate security risk, fix ASAP
- **HIGH**: Significant vulnerability, fix soon
- **MEDIUM**: Moderate risk, should fix
- **LOW/INFO**: Best practices, consider fixing

## Example Findings

### Solidity Example
```solidity
// CRITICAL: Reentrancy vulnerability
function withdraw() external {
    msg.sender.call{value: balance[msg.sender]}("");  // ❌ External call first
    balance[msg.sender] = 0;  // ❌ State change after
}

// FIX:
function withdraw() external {
    uint256 amount = balance[msg.sender];
    balance[msg.sender] = 0;  // ✅ State change first
    msg.sender.call{value: amount}("");  // ✅ External call after
}
```

### Next.js API Route Example
```typescript
// HIGH: No authentication check
export async function GET(req: Request) {
    const users = await db.users.findMany();  // ❌ No auth
    return Response.json(users);
}

// FIX:
export async function GET(req: Request) {
    const session = await auth(req);  // ✅ Check auth
    if (!session) return Response.json({ error: 'Unauthorized' }, { status: 401 });
    
    const users = await db.users.findMany();
    return Response.json(users);
}
```

### XSS Example
```typescript
// MEDIUM: XSS vulnerability
<div dangerouslySetInnerHTML={{ __html: userInput }} />  // ❌ Unescaped user input

// FIX:
import DOMPurify from 'isomorphic-dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />  // ✅ Sanitized
```

## Integration with CI/CD

The audit can be run automatically in GitHub Actions:

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
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Best Practices

1. **Run regularly**: Weekly audits for active projects
2. **Before deployment**: Always audit before production releases
3. **After major changes**: Audit when adding new features
4. **Review findings**: Don't ignore LOW severity issues
5. **Track fixes**: Use issues to track remediation

## Project-Specific Context

For best results, create a `CLAUDE.md` in your repository root:

```markdown
## Project Overview
[Brief description of your project]

## Security Considerations
- [List security-critical features]
- [Known security patterns used]
- [External integrations]

## Architecture
- [Key components]
- [Trust boundaries]
- [External dependencies]
```

This helps the audit understand your specific context and patterns.
