# Smart Contract Security Audit Skill

## Overview

This Claude Code skill provides automated deep security analysis for Solidity smart contracts, with specific understanding of DeFi protocols including Euler Vault Kit (EVK) and Uniswap integrations.

## Features

- **Comprehensive Vulnerability Detection**: Checks for common smart contract vulnerabilities including reentrancy, access control issues, integer operations, external call safety, and more
- **DeFi-Specific Analysis**: Understands EVK/EVC patterns, vault operations, liquidation logic, and swap integrations
- **Gas Optimization**: Identifies opportunities to reduce gas costs
- **Code Quality**: Flags code quality issues and best practice violations
- **Context-Aware**: Recognizes legitimate project-specific patterns vs actual vulnerabilities
- **Detailed Reports**: Generates comprehensive Markdown audit reports with severity ratings, impact analysis, and fix recommendations

## How It Works

### Automatic Activation

The skill automatically activates when:
- You open or review a `.sol` (Solidity) file
- You ask about security or vulnerabilities
- You request a code review of smart contracts
- You mention "audit" in the context of smart contracts

### Audit Process

1. **Initial Analysis**: Identifies contract type, dependencies, and trust boundaries
2. **Complete Code Reading**: Reads the entire contract before analysis
3. **Systematic Vulnerability Scan**: Checks each vulnerability category methodically
4. **Context-Aware Analysis**: Understands project-specific architecture and patterns
5. **Report Generation**: Creates a comprehensive audit report in `audits/` directory

## Vulnerability Categories Checked

### Common Vulnerabilities
- Reentrancy attacks
- Access control issues
- Integer overflow/underflow and precision loss
- External call safety
- Token handling issues
- Flash loan attack vectors

### DeFi-Specific Checks
- EVK/EVC integration patterns
- Vault operation safety
- Liquidation logic correctness
- Swap integration and slippage protection

### Additional Checks
- Gas optimization opportunities
- Code quality issues
- Architecture and design patterns

## Project-Specific Context

This skill understands your project's architecture:

### Vault Structure
- **quoteVault**: Quote yield vault (can be collateral or debt)
- **assetVault**: Asset yield vault (can be collateral or debt)

**Note**: The two vaults use different underlying tokens (e.g., USDC and WETH)

### Recognized Patterns (Not Flagged as Vulnerabilities)

✅ **Legitimate Patterns**:
- `callThroughEVC` modifier for EVC authentication
- `evc.getCurrentOnBehalfOfAccount()` for getting the actual user
- `evc.call()` for vault operations on behalf of users
- `evc.requireAccountStatusCheck()` at the end of risky operations
- External swap router calls with slippage protection
- EVC-based borrowing followed by swaps for margin positions

❌ **Anti-Patterns** (Will Be Flagged):
- Direct vault operations without EVC authentication
- Missing health checks after position changes
- Swap operations without slippage protection
- Missing authorization checks on operator functions
- Unchecked external call return values
- Unnecessary multiple swaps

## Audit Report Format

Reports are saved to: `audits/[ContractName]_audit_[YYYY-MM-DD].md`

### Report Structure

1. **Executive Summary**: Risk level and issue counts
2. **Findings**: Detailed vulnerability reports with:
   - Severity rating (CRITICAL/HIGH/MEDIUM/LOW)
   - Location in code
   - Category
   - Description and impact
   - Recommendations and code examples
3. **Gas Optimization Opportunities**: Specific gas-saving suggestions
4. **Code Quality Issues**: Best practice violations
5. **Architecture Review**: High-level design observations
6. **Positive Findings**: Security strengths
7. **Conclusion**: Overall assessment and next steps

## Usage Examples

### Example 1: Review a Contract

```bash
# Simply ask Claude to review a contract
"Can you audit src/operators/PositionOpenOperator.sol?"
```

The skill will automatically activate and generate a comprehensive audit report.

### Example 2: Security Check During Development

```bash
# When working on a new contract
"I've just finished implementing LiquidationOperator.sol. Can you do a security review?"
```

### Example 3: Specific Vulnerability Check

```bash
# Ask about specific concerns
"Does this contract have any reentrancy vulnerabilities?"
```

The skill will focus on reentrancy but still perform a comprehensive check.

## Best Practices

1. **Use Early and Often**: Run audits during development, not just at the end
2. **Review All Findings**: Even LOW severity issues can indicate design problems
3. **Understand the Context**: The skill understands EVK/EVC patterns, but review its reasoning
4. **Combine with Professional Audits**: This tool augments but doesn't replace human security experts
5. **Check Gas Optimizations**: Even if security is solid, gas savings can be significant

## Limitations

- **Automated Analysis**: While comprehensive, automated tools can miss sophisticated attacks
- **No Formal Verification**: Does not provide mathematical proofs of correctness
- **Context Dependent**: Some issues depend on how contracts are used together
- **High-Value Contracts**: Always get professional audits for production contracts handling significant funds

## Integration with Development Workflow

This skill integrates seamlessly with your existing Claude Code workflow:

1. **During Development**: Get real-time feedback on security as you code
2. **Before Commits**: Quick security check before committing changes
3. **Code Reviews**: Automated first-pass review before human review
4. **Pre-Deployment**: Final security check before deploying to testnet/mainnet

## Report Interpretation

### Severity Levels

- **CRITICAL**: Immediate fix required. Funds at risk, core functionality broken
- **HIGH**: Important fix needed. Significant risk or impact
- **MEDIUM**: Should be fixed. Moderate risk or quality issue
- **LOW/INFO**: Nice to fix. Minor issues or informational findings

### Priority Order

1. Fix all CRITICAL issues immediately
2. Address HIGH severity issues before deployment
3. Review MEDIUM issues and plan fixes
4. Consider LOW/INFO issues for code quality improvements

## Contributing

This skill is part of your project's Claude Code configuration. To improve it:

1. Edit `.claude/skills/audit/skill.md` to add new checks or patterns
2. Update project-specific context as your architecture evolves
3. Add examples of good/bad patterns you discover

## Support

For issues or questions:
- Check the skill.md file for detailed audit logic
- Review the audit reports in `audits/` directory
- Consult your team's security experts for complex issues

---

**Disclaimer**: This automated audit skill provides comprehensive analysis but should not replace professional security audits by human experts, especially for production contracts handling significant value. Always seek multiple security reviews for high-value contracts.
