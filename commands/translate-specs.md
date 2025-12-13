---
description: Translate specification files to Japanese
---

# Translate Specifications Command

Translate specification files from English to Japanese while preserving technical accuracy and formatting.

This command uses the [Specification Translation Expert skill](../skills/translate-specs/skill.md) for high-quality translation.

## Usage

```bash
/translate-specs
```

## Configuration

Configure via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `TRANSLATE_SOURCE_PATH` | `specs/**/*.md` | Source file pattern |
| `TRANSLATE_TARGET_BASE_PATH` | `docs/ja` | Target directory base |

## What It Does

The skill automatically:

1. **Finds files** - Matches source pattern
2. **Translates content** - Preserves code and formatting
3. **Maintains structure** - Same directory hierarchy
4. **Ensures consistency** - Standardized terminology

## Translation Rules

| Content Type | Handling |
|--------------|----------|
| Text | Translate to Japanese |
| Code blocks | Keep unchanged |
| Code comments | Translate to Japanese |
| Technical terms | Keep English + Japanese on first use |
| Links | Preserve relative paths |
| Formatting | Maintain exactly |

## Common Terms

| English | Japanese |
|---------|----------|
| Vault | ボールト |
| Collateral | 担保 |
| Liquidation | 清算 |
| Controller | コントローラー |
| Account | アカウント |

## Example

**Source:**
```markdown
# Vault Specification

A vault manages user deposits.
```

**Target:**
```markdown
# Vault仕様

Vault(ボールト)はユーザーの預け入れを管理します。
```

## Output

After translation, provides:
- Files translated list
- Statistics (files, lines, terms)
- Glossary used
- Any notes or decisions made

## Integration with CI/CD

```yaml
name: Translate Specs

on:
  push:
    paths:
      - 'specs/**/*.md'
  workflow_dispatch:

jobs:
  translate:
    uses: teliha/dev-workflows/.github/workflows/translate-specs.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Quality Checklist

- [ ] All files translated
- [ ] Code blocks unchanged
- [ ] Links working
- [ ] Terms consistent
- [ ] Formatting preserved

## Related

- [Specification Translation Expert Skill](../skills/translate-specs/skill.md) - Detailed translation process
