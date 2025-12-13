# Implementation Review Loop Skill

## Overview

This Claude Code skill performs systematic self-review of implementations against their specifications, iterating until the implementation fully matches the spec.

## Features

- **Spec-to-Implementation Verification**: Compares code against specification
- **Gap Detection**: Identifies missing or incorrect implementations
- **Iterative Fixes**: Loops until all requirements are met
- **Comprehensive Checklist**: Covers functionality, edge cases, errors, security
- **Detailed Reporting**: Clear gap reports with fix instructions

## How It Works

### Automatic Activation

The skill automatically activates when:
- User says "implementation is done" or "finished implementing"
- User asks to "verify implementation against spec"
- User mentions "review my implementation"
- User requests "self-review" or "feedback loop"

### Review Loop

```
Spec → Check Implementation → Gaps Found?
                                  │
                    ┌─────────────┴───────────┐
                    ▼                         ▼
                   YES                        NO
                    │                         │
              Apply Fixes                   Done!
                    │
                    └──────── Loop Back ─────┘
```

## Review Categories

| Category | What's Checked |
|----------|---------------|
| Functional Completeness | All requirements implemented |
| Edge Cases | Zero, max, boundary conditions |
| Error Handling | All error cases from spec |
| Technical Accuracy | Types, signatures, return values |
| Security | Access control, validation |
| Performance | Efficiency, resource limits |

## Usage Examples

### Example 1: After Implementation

```
User: "I've finished implementing the auth module from the spec"

Skill Response:
1. Locates auth spec
2. Reviews implementation files
3. Finds 3 gaps
4. Applies fixes
5. Re-reviews (loop)
6. Reports all clear
```

### Example 2: Explicit Request

```
User: "Review my vault implementation against specs/vault/spec.md"
```

### Example 3: Self-Review Loop

```
User: "Run the implementation review loop for the API endpoints"
```

## Output Format

### Gap Report

```markdown
# Implementation Review Report

## Summary
| Category | Status | Issues |
|----------|--------|--------|
| Functional | ⚠️ | 2 |
| Edge Cases | ❌ | 3 |

## Gaps Found

### Gap #1: Missing Validation
**Spec Says**: "Validate input before processing"
**Implementation Has**: No validation
**Fix**: Add input validation at line 45
```

### Final Report

```markdown
## Implementation Review Complete

**Status**: ✅ ALL REQUIREMENTS MET
**Iterations**: 3
**Total Fixes**: 7

Ready for code review.
```

## Review Checklist

For each spec requirement:
- [ ] Requirement implemented?
- [ ] Implementation matches spec exactly?
- [ ] Edge cases handled?
- [ ] Errors handled correctly?
- [ ] Security requirements met?

## Best Practices

### During Review
1. **Be thorough** - Check every requirement
2. **Be precise** - Note exact locations
3. **Be constructive** - Provide clear fixes

### During Fixes
1. **One at a time** - Fix gaps individually
2. **Verify immediately** - Don't batch fixes
3. **Minimal changes** - Don't refactor

### When to Stop
- ✅ All requirements implemented
- ✅ All tests pass
- ✅ No known gaps remain
- ❌ NOT "good enough"

## Project-Specific Checks

### Solidity
- Events emitted per spec
- Modifiers applied correctly
- Gas considerations met

### TypeScript
- Types match interfaces
- Async handling correct
- API formats match

### React
- Props match spec
- State management correct
- Accessibility met

## Integration

This skill works well with:
- Specification files in `specs/` directory
- Implementation in `src/` directory
- Test files for verification

## Trigger Phrases

- "Review my implementation against the spec"
- "Check if implementation matches spec"
- "Run implementation review loop"
- "Verify implementation completeness"
- "Self-review the implementation"
- "Implementation is done, please verify"

## Limitations

- Requires clear, well-structured specifications
- Cannot verify requirements not in spec
- May need human judgment for ambiguous cases
- Performance testing may need manual verification

## Contributing

To improve this skill:
1. Add project-specific review patterns
2. Improve gap detection heuristics
3. Add more detailed fix suggestions
4. Test with real spec-to-implementation workflows
