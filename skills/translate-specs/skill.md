---
name: Specification Translation Expert
description: Translate specification files to Japanese while preserving technical accuracy and formatting
category: Documentation
tags: [translation, japanese, documentation, i18n, localization]
---

<!-- TRANSLATE-SPECS:START -->

# Specification Translation Expert Skill

## When to Use This Skill

This skill automatically activates when:
- User asks to translate documentation
- User mentions "translate", "Japanese", or "localization"
- Working with specification files that need Japanese versions
- User requests documentation in Japanese

## Translation Process

### Step 0: Read Configuration

Check environment variables for configuration:

```bash
# Source path pattern (default: specs/**/*.md)
$TRANSLATE_SOURCE_PATH

# Target base directory (default: docs/ja)
$TRANSLATE_TARGET_BASE_PATH
```

### Step 1: Identify Files to Translate

Find all markdown files matching the source pattern:

```bash
# Default
find specs/ -name "*.md" -type f

# Custom path from environment
find ${TRANSLATE_SOURCE_PATH} -type f
```

### Step 2: Create Target Structure

Map source paths to target paths:

| Source | Target |
|--------|--------|
| `specs/vault/spec.md` | `docs/ja/specs/vault/spec.md` |
| `docs/architecture.md` | `docs/ja/docs/architecture.md` |
| `README.md` | `docs/ja/README.md` |

### Step 3: Translation Guidelines

## Translation Rules

### Technical Terms

**Keep in English (with Japanese explanation on first use):**
- Smart contract terminology: vault, collateral, liability, liquidation
- Programming terms: interface, modifier, function, event
- Protocol-specific terms: EVC, EVK, controller

**Example:**
```markdown
# English
The vault implements the IVault interface.

# Japanese
Vault(ボールト)はIVaultインターフェースを実装します。
```

### Code and Comments

- **Keep all code blocks unchanged**
- **Translate code comments** to Japanese
- **Keep function/variable names** in English

**Example:**
```solidity
// English
// Check if the account is healthy
function checkAccountHealth(address account) external view returns (bool);

// Japanese
// アカウントが健全かチェックする
function checkAccountHealth(address account) external view returns (bool);
```

### Formatting Preservation

- Maintain all markdown formatting (headers, lists, tables, links)
- Preserve internal links with correct paths
- Keep external URLs unchanged
- Maintain code fence languages

### Mathematical Expressions

Translate labels, keep formulas:

```markdown
# English
Collateral Value = Balance × Price × Collateral Factor

# Japanese
担保価値 = 残高 × 価格 × 担保係数
```

### Writing Style

- Use polite form (です・ます体)
- Be consistent with technical term translations
- Use Japanese punctuation (、。)
- Maintain consistent terminology across files

## Common Term Translations

| English | Japanese |
|---------|----------|
| Vault | ボールト |
| Collateral | 担保 |
| Liability | 負債 |
| Liquidation | 清算 |
| Controller | コントローラー |
| Account | アカウント |
| Health Factor | ヘルスファクター |
| Oracle | オラクル |
| Governor | ガバナー |
| Interest Rate | 金利 |
| Deposit | 預け入れ |
| Withdraw | 引き出し |
| Borrow | 借り入れ |
| Repay | 返済 |
| Position | ポジション |
| Margin | マージン |
| Leverage | レバレッジ |
| Slippage | スリッページ |
| Transaction | トランザクション |
| Block | ブロック |
| Gas | ガス |
| Fee | 手数料 |

## Step 4: Translate Each File

For each file:

1. **Read source file** completely
2. **Translate content** following guidelines
3. **Preserve formatting** exactly
4. **Check technical terms** for consistency
5. **Write to target location**

## Step 5: Update Internal Links

Ensure links work in translated structure:

```markdown
# English
See [Vault Spec](./vault/spec.md)

# Japanese (same relative path works)
[Vault仕様](./vault/spec.md)を参照してください
```

## Step 6: Verification

After translation:

- [ ] All files translated
- [ ] Directory structure matches source
- [ ] Links work correctly
- [ ] Code blocks preserved exactly
- [ ] Technical terms consistent
- [ ] Formatting intact
- [ ] No English text remains (except code/terms)

## Output Format

After translation:

```markdown
## Translation Summary

### Configuration
- Source pattern: `specs/**/*.md`
- Target base: `docs/ja`

### Files Translated
| Source | Target | Status |
|--------|--------|--------|
| `specs/vault/spec.md` | `docs/ja/specs/vault/spec.md` | Done |
| `specs/api/spec.md` | `docs/ja/specs/api/spec.md` | Done |

### Statistics
- Total files: 5
- Total lines translated: 1,234
- Technical terms standardized: 45

### Glossary Used
| English | Japanese | Count |
|---------|----------|-------|
| Vault | ボールト | 23 |
| Collateral | 担保 | 18 |

### Notes
- [Any special considerations or decisions made]
```

## Quality Checklist

Before completing:

- [ ] All markdown files translated
- [ ] Directory structure preserved
- [ ] All code blocks unchanged
- [ ] Internal links work
- [ ] Technical terms consistent
- [ ] Formatting intact
- [ ] Mathematical expressions preserved
- [ ] No remaining untranslated text (except intentional English)

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

## Best Practices

1. **Consistency**: Use the same translation for the same term
2. **Context**: Understand the context before translating
3. **Technical accuracy**: Don't sacrifice accuracy for fluency
4. **Formatting**: Preserve all original formatting
5. **Review**: Have native speakers review important translations

<!-- TRANSLATE-SPECS:END -->
