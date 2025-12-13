# Specification Translation Expert Skill

## Overview

This Claude Code skill translates specification files from English to Japanese while preserving technical accuracy, code blocks, and formatting.

## Features

- **Technical Term Handling**: Keeps English terms with Japanese explanations
- **Code Preservation**: Code blocks remain unchanged
- **Format Preservation**: Maintains markdown structure perfectly
- **Consistency**: Uses standardized term translations
- **Configurable**: Source and target paths via environment variables

## How It Works

### Automatic Activation

The skill automatically activates when:
- User asks to translate documentation
- User mentions "translate", "Japanese", or "localization"
- Working with specification files
- User requests Japanese documentation

### Process

1. **Configure**: Read source/target paths from environment
2. **Identify**: Find all files to translate
3. **Translate**: Convert content following guidelines
4. **Preserve**: Keep code and formatting intact
5. **Verify**: Check consistency and completeness

## Configuration

Set via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `TRANSLATE_SOURCE_PATH` | `specs/**/*.md` | Source file pattern |
| `TRANSLATE_TARGET_BASE_PATH` | `docs/ja` | Target directory base |

## Translation Rules

### Technical Terms

Keep in English with Japanese on first use:
```markdown
Vault(ボールト)は資産を管理します。
```

### Code Blocks

Preserve exactly, translate comments only:
```solidity
// アカウントの健全性をチェック
function checkHealth() external view returns (bool);
```

### Formatting

- Markdown structure preserved
- Links updated for translated paths
- Tables, lists, headers unchanged

## Common Translations

| English | Japanese |
|---------|----------|
| Vault | ボールト |
| Collateral | 担保 |
| Liquidation | 清算 |
| Controller | コントローラー |

## Usage Examples

### Example 1: Translate All Specs

```bash
"Translate the specifications to Japanese"
```

### Example 2: Specific Directory

```bash
"Translate docs/api/ to Japanese"
```

### Example 3: With Custom Config

Set environment variables then:
```bash
"Run translate-specs command"
```

## Output Format

```markdown
## Translation Summary

### Files Translated
- `specs/vault/spec.md` -> `docs/ja/specs/vault/spec.md`

### Statistics
- Total files: 5
- Lines translated: 1,234

### Glossary
| Term | Translation | Count |
|------|-------------|-------|
| Vault | ボールト | 23 |
```

## Integration with CI/CD

```yaml
name: Translate Specs

on:
  push:
    paths:
      - 'specs/**/*.md'

jobs:
  translate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          prompt: /translate-specs
        env:
          TRANSLATE_SOURCE_PATH: "specs/**/*.md"
          TRANSLATE_TARGET_BASE_PATH: "docs/ja"
```

## Quality Checklist

- [ ] All files translated
- [ ] Code blocks unchanged
- [ ] Links working
- [ ] Terms consistent
- [ ] Formatting preserved

## Best Practices

1. **Consistency**: Same term = same translation
2. **Technical accuracy**: Prioritize over fluency
3. **Preserve formatting**: Don't alter structure
4. **Review**: Native speakers should review

## Related Commands

- `/translate-specs` - Explicitly invoke this skill

## Limitations

- Japanese target only (currently)
- Requires well-structured markdown
- Cannot translate images or diagrams
- May need review for domain-specific terms

## Contributing

To improve this skill:
1. Add more term translations to glossary
2. Add support for other languages
3. Improve consistency checking
4. Test with real documentation
