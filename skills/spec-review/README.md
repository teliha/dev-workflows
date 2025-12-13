# Specification Review Expert Skill

## Overview

This Claude Code skill reviews specifications for completeness, clarity, testability, and quality BEFORE implementation begins. Catching issues in specs is much cheaper than fixing them in code.

## Features

- **Completeness Check**: Are all necessary requirements included?
- **Clarity Analysis**: Are there ambiguous terms or vague requirements?
- **Testability Review**: Can acceptance criteria be objectively verified?
- **Edge Case Detection**: Are boundary conditions considered?
- **Consistency Verification**: Is the spec internally consistent?
- **Feasibility Assessment**: Is the spec technically achievable?

## How It Works

### Automatic Activation

The skill automatically activates when:
- User finishes writing a specification
- User says "review this spec" or "check my specification"
- User asks "is this spec complete?"
- User creates or modifies files in `specs/` directory

### Review Process

1. **Read** - Understand the entire specification
2. **Review** - Check against all quality criteria
3. **Report** - Generate detailed findings with recommendations
4. **Iterate** - Author fixes issues, re-review if needed

## Review Categories

| Category | What's Checked |
|----------|---------------|
| Completeness | All necessary sections present |
| Clarity | No ambiguous terms |
| Testability | Measurable acceptance criteria |
| Edge Cases | Boundary conditions covered |
| Consistency | No internal contradictions |
| Feasibility | Technically achievable |
| Security | Security requirements defined |
| Structure | Well-organized format |

## Usage Examples

### Example 1: Review New Spec

```
User: "I just wrote specs/auth/spec.md, please review it"
```

### Example 2: Check Completeness

```
User: "Is this spec complete enough to implement?"
```

### Example 3: After Editing

```
User: "I updated the API spec, does it look good now?"
```

## Ambiguous Patterns Detected

| Ambiguous | Should Be |
|-----------|-----------|
| "respond quickly" | "< 200ms response time" |
| "large files" | "files > 100MB" |
| "appropriate access" | "access to owned resources" |
| "as needed" | "when X condition is met" |

## Output Format

```markdown
# Specification Review Report

## Summary
| Category | Status |
|----------|--------|
| Completeness | ⚠️ |
| Clarity | ❌ |
| Testability | ✅ |

## Critical Issues
### Issue #1: Missing error handling
**Problem**: No error cases defined
**Recommendation**: Add error handling section

## Questions for Clarification
1. What is the maximum file size?
2. Who can access admin functions?
```

## Severity Levels

| Level | Meaning |
|-------|---------|
| ❌ Critical | Must fix before implementation |
| ⚠️ Warning | Should fix |
| 💡 Suggestion | Nice to have |
| ✅ Pass | Good to go |

## Completeness Checklist

A good spec should include:
- [ ] Purpose/Overview
- [ ] Scope (included/excluded)
- [ ] Functional requirements
- [ ] Non-functional requirements
- [ ] Data formats (inputs/outputs)
- [ ] Error handling
- [ ] Edge cases
- [ ] Acceptance criteria
- [ ] Dependencies

## Best Practices for Specs

1. **Use RFC 2119 keywords**: MUST, SHOULD, MAY
2. **Quantify everything**: Replace vague terms with numbers
3. **Include examples**: Sample inputs/outputs
4. **Document decisions**: Why this approach?
5. **Consider implementers**: What will they need to know?

## Workflow Integration

```
Write Spec
    ↓
Spec Review (this skill)  ←── Fix Issues
    ↓
Check Contradictions (vs other specs)
    ↓
Implement
    ↓
Implementation Review (vs spec)
```

## Related Skills

- `check-spec-contradictions` - Compare specs against each other
- `implementation-review` - Verify implementation matches spec

## Trigger Phrases

- "Review this spec"
- "Check my specification"
- "Is this spec complete?"
- "Does this spec look good?"
- "Review specs/feature/spec.md"

## Limitations

- Cannot verify business correctness (only structure/clarity)
- May not catch domain-specific issues
- Human review still recommended for critical specs
- Cannot predict all implementation challenges

## Contributing

To improve this skill:
1. Add domain-specific review patterns
2. Improve ambiguity detection
3. Add more example findings
4. Test with real specifications
