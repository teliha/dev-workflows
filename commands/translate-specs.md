---
description: Translate specification files to Japanese
---

# Translate Specifications to Japanese

You are tasked with translating specification files from English to Japanese while preserving technical accuracy and formatting.

## Task Overview

Translate markdown files to Japanese, maintaining the same directory structure.

**Path Configuration:**
- Source path: Check `$TRANSLATE_SOURCE_PATH` environment variable (default: `specs/**/*.md`)
- Target base path: Check `$TRANSLATE_TARGET_BASE_PATH` environment variable (default: `docs/ja`)

**Examples:**
- `specs/vault/spec.md` → `docs/ja/specs/vault/spec.md`
- `docs/architecture.md` → `docs/ja/docs/architecture.md`
- `README.md` → `docs/ja/README.md`

## Translation Guidelines

### 1. Technical Terms

**Keep in English (with Japanese explanation on first use):**
- Smart contract terminology: vault, collateral, liability, liquidation
- Programming terms: interface, modifier, function, event
- Protocol-specific terms: EVC, EVK, controller, account status check

**Example:**
```markdown
# English
The vault implements the IVault interface.

# Japanese
Vault（ボールト）はIVaultインターフェースを実装します。
```

### 2. Code and Code Comments

- **Keep all code blocks unchanged** (code is language-agnostic)
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

### 3. Formatting Preservation

- Maintain all markdown formatting (headers, lists, tables, links)
- Preserve internal links: `[リンク](../other-spec.md)`
- Keep external URLs unchanged
- Maintain code fence languages: ```solidity, ```typescript

### 4. Mathematical Expressions

Keep mathematical notation unchanged:
```markdown
# English
Collateral Value = Balance × Price × Collateral Factor

# Japanese  
担保価値 = 残高 × 価格 × 担保係数
```

### 5. Consistency Rules

- Use polite form (です・ます体)
- Be consistent with technical term translations
- Maintain consistent terminology across all files
- Use Japanese punctuation (、。) instead of English (, .)

## Common Technical Term Translations

Create a glossary as you translate. Common terms:

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

## Step-by-Step Process

### Step 0: Read Configuration

1. Read `$TRANSLATE_SOURCE_PATH` environment variable (default: `specs/**/*.md`)
2. Read `$TRANSLATE_TARGET_BASE_PATH` environment variable (default: `docs/ja`)

### Step 1: Identify Files to Translate

Use the source path pattern to find all matching markdown files.

**Example with default:**
```bash
# Finds: specs/**/*.md
```

**Example with custom path:**
```bash
# If TRANSLATE_SOURCE_PATH="docs/**/*.md"
# Finds: docs/**/*.md
```

### Step 2: Create Target Directory Structure

For each source file, determine the target path:
- Source: `{source_file_path}`
- Target: `{target_base_path}/{source_file_path}`

**Examples:**
- Source: `specs/vault/spec.md` → Target: `docs/ja/specs/vault/spec.md`
- Source: `docs/architecture.md` → Target: `docs/ja/docs/architecture.md`
- Source: `README.md` → Target: `docs/ja/README.md`

### Step 3: Translate Each File

For each markdown file:

1. **Read the source file** completely
2. **Translate content** following the guidelines above
3. **Verify formatting** is preserved
4. **Check technical terms** for consistency
5. **Write to target location** with the same filename

### Step 4: Update Internal Links

Update internal markdown links to point to translated versions:
- `[link](../other-spec.md)` → Keep relative paths working
- Ensure all internal links resolve correctly in the translated structure

### Step 5: Verify Translation

- All files translated
- Directory structure matches source
- Links work correctly
- Code blocks are preserved
- Technical terms are consistent

## Output Format

After translation, provide a summary:

```markdown
## Translation Summary

### Configuration
- Source pattern: `{TRANSLATE_SOURCE_PATH or default}`
- Target base: `{TRANSLATE_TARGET_BASE_PATH or default}`

### Files Translated
- `{source_file_1}` → `{target_file_1}`
- `{source_file_2}` → `{target_file_2}`
- ... (list all files)

### Statistics
- Total files: X
- Total lines translated: Y
- Technical terms standardized: Z

### Glossary
(Provide the glossary of technical terms used)

### Notes
- Any special considerations or decisions made during translation
```

## Quality Checklist

Before completing:

- [ ] All markdown files matching source pattern are translated
- [ ] Directory structure is preserved under target base path
- [ ] All code blocks are preserved exactly
- [ ] All internal links work correctly in translated structure
- [ ] Technical terms are consistent across all files
- [ ] Formatting is intact (headers, lists, tables)
- [ ] Mathematical expressions are preserved
- [ ] No English text remains except for code and technical terms

## Integration with CI/CD

This command can be used in GitHub Actions:

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

## Example Translation

**Source (`specs/example/spec.md`):**
```markdown
# Vault Specification

## Overview

A vault is a contract that manages user deposits and withdrawals.

### Functions

- `deposit(uint256 amount)` - Deposit tokens
- `withdraw(uint256 amount)` - Withdraw tokens

## Security Considerations

The vault must check account health before withdrawals.
```

**Target (`docs/ja/specs/example/spec.md`):**
```markdown
# Vault仕様

## 概要

Vault（ボールト）は、ユーザーの預け入れと引き出しを管理するコントラクトです。

### 関数

- `deposit(uint256 amount)` - トークンを預け入れる
- `withdraw(uint256 amount)` - トークンを引き出す

## セキュリティに関する考慮事項

Vaultは引き出し前にアカウントの健全性をチェックする必要があります。
```

## Notes

- This is an automated translation task - focus on accuracy and consistency
- When in doubt about technical terms, keep them in English with Japanese explanation
- Preserve all original formatting and structure
- Do not modify code or code examples
- Create a glossary for future reference and consistency
