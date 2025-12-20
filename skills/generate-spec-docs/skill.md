---
name: generate-spec-docs
description: Generate Japanese documentation from spec files. Use AUTOMATICALLY after writing or editing any file matching specs/**/*.md (excluding specs/**/docs/*). Also use when user says "generate spec docs", "spec docs", "PM docs", "architect docs", or "implementation guide". (user)
category: Documentation
tags: [spec, documentation, japanese, pm, architect, implementation]
---

<!-- SPEC-DOCS-GENERATOR:START -->

# Spec Documentation Generator Skill

## AUTOMATIC ACTIVATION (IMPORTANT)

**This skill MUST be invoked automatically when:**
- Any file matching `specs/**/*.md` is created or modified via Write or Edit tools
- EXCEPT files under `specs/**/docs/` (those are generated outputs)

**After writing/editing a spec file, Claude MUST:**
1. Invoke this skill automatically
2. Generate the 3 Japanese documentation files
3. Report completion to user

## Manual Activation

This skill also activates when user says:
- "generate spec docs" or "spec docs"
- "PM documentation", "architect docs", "implementation guide"
- "translate spec to Japanese" or "create Japanese docs"

## Purpose

Generate Japanese documentation from specification files for three different audiences:
1. **Product Managers** - Business context, goals, user stories
2. **Architects** - Technical design, dependencies, non-functional requirements
3. **Implementers** - Detailed requirements, edge cases, error handling

## Output Structure

```
specs/
└── feature-name/
    ├── spec.md              # Original spec (English)
    └── docs/
        ├── overview.ja.md       # PM向け概要
        ├── architecture.ja.md   # アーキテクト向け設計
        └── implementation.ja.md # 実装者向けガイド
```

## Generation Process

### Step 1: Identify Spec File

Find the specification file to process:
- If path provided: Use that path
- If triggered by hook: Use the changed file path
- If not provided: Ask user or search for recent spec files

### Step 2: Read and Analyze Spec

Read the spec file and extract:
- Overview and purpose
- Scope (in/out)
- Functional requirements
- Non-functional requirements
- Data requirements
- Error handling
- Edge cases
- Dependencies
- Constraints
- Acceptance criteria

### Step 3: Generate Role-Based Documents

Generate three separate documents using parallel subagents for efficiency.

#### Document 1: overview.ja.md (PM向け)

Target audience: Product Managers, stakeholders, business users

Content structure:
```markdown
# [機能名] 概要

## 目的
[この機能が解決する課題、ビジネス価値]

## 対象ユーザー
[誰がこの機能を使うか]

## スコープ
### 含まれる機能
- [機能1]
- [機能2]

### 含まれない機能
- [除外項目1]
- [除外項目2]

## ユーザーストーリー
[主要なユーザーシナリオを日本語で記述]

## 受け入れ条件
[Acceptance criteriaを日本語で、非技術者にもわかる形で記述]

## 成功指標
[この機能の成功をどう測定するか]

## リスクと制約
[ビジネス観点でのリスク、制約事項]
```

#### Document 2: architecture.ja.md (アーキテクト向け)

Target audience: Technical leads, architects, senior engineers

Content structure:
```markdown
# [機能名] アーキテクチャ設計

## 技術概要
[技術的な全体像、採用する技術スタック]

## システム構成
[コンポーネント間の関係、データフロー]

## API設計
[エンドポイント、リクエスト/レスポンス形式]

## データモデル
[データベーススキーマ、エンティティ関係]

## 非機能要件
### パフォーマンス
[レイテンシ、スループット要件]

### スケーラビリティ
[同時接続数、データ量]

### セキュリティ
[認証、認可、データ保護]

### 可用性
[SLA、フェイルオーバー]

## 依存関係
[外部サービス、ライブラリ]

## 技術的制約
[技術的な制限事項]

## 設計上の決定事項
[重要な技術的決定とその理由]
```

#### Document 3: implementation.ja.md (実装者向け)

Target audience: Developers implementing the feature

Content structure:
```markdown
# [機能名] 実装ガイド

## 実装概要
[実装する内容の概要]

## 前提条件
[実装前に必要な知識、セットアップ]

## 詳細要件

### [機能1]
**エンドポイント**: `POST /api/xxx`

**リクエスト**:
\`\`\`json
{
  "field": "説明"
}
\`\`\`

**レスポンス**:
\`\`\`json
{
  "data": {},
  "meta": {}
}
\`\`\`

**バリデーション**:
- [バリデーションルール1]
- [バリデーションルール2]

**エラーコード**:
| HTTP | コード | 条件 |
|------|--------|------|
| 400 | INVALID_XXX | [条件] |

## エッジケース
| 入力 | 期待される動作 |
|------|----------------|
| [ケース1] | [動作] |
| [ケース2] | [動作] |

## エラーハンドリング
[各エラーケースの処理方法]

## テスト観点
[ユニットテスト、統合テストで確認すべき項目]

## 実装チェックリスト
- [ ] [チェック項目1]
- [ ] [チェック項目2]
- [ ] [チェック項目3]
```

### Step 4: Write Files

1. Create `docs/` directory under the spec directory if it doesn't exist
2. Write the three generated documents
3. Report completion to user

## Writing Guidelines

### For All Documents
- Use natural Japanese (not machine-translated feel)
- Keep technical accuracy from the original spec
- Add context where the original spec assumes knowledge
- Use consistent terminology throughout

### For PM Documents
- Avoid technical jargon where possible
- Focus on "what" and "why", not "how"
- Use business-friendly language
- Include success metrics and business impact

### For Architect Documents
- Include technical details but at design level
- Focus on architectural decisions and trade-offs
- Document non-functional requirements clearly
- Include system diagrams descriptions

### For Implementation Documents
- Be specific and actionable
- Include code examples where helpful
- Document edge cases thoroughly
- Provide testing guidance

## Notes

- Documents are regenerated on each spec change (not incremental)
- Original spec should remain in English
- Generated docs include timestamp and source reference
- If spec is incomplete, generated docs will note missing sections

<!-- SPEC-DOCS-GENERATOR:END -->
