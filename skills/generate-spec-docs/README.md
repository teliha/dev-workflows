# Spec Documentation Generator

Specファイルから日本語ドキュメントを自動生成するスキルです。

## 概要

英語で書かれたSpecファイルを、以下の3種類の日本語ドキュメントに変換します：

| ファイル | 対象者 | 内容 |
|---------|--------|------|
| `overview.ja.md` | PM・ステークホルダー | ビジネス概要、スコープ、受け入れ条件 |
| `architecture.ja.md` | アーキテクト・テックリード | 技術設計、依存関係、非機能要件 |
| `implementation.ja.md` | 実装者 | 詳細要件、エッジケース、実装チェックリスト |

## 出力構造

```
specs/
└── feature-name/
    ├── spec.md              # 元のSpec（英語）
    └── docs/
        ├── overview.ja.md       # PM向け概要
        ├── architecture.ja.md   # アーキテクト向け設計
        └── implementation.ja.md # 実装者向けガイド
```

## 使い方

### 自動実行

`specs/**/*.md` ファイルを作成・編集すると、Claude Codeが自動的にこのスキルを実行してドキュメントを生成します。

### 手動実行

```
generate spec docs for specs/feature-name/spec.md
```

または単に「spec docs」と言えば、パスを聞かれます。

## トリガーフレーズ

- `generate spec docs for [path]`
- `spec docs`
- `create Japanese documentation`
- `PM documentation`
- `architect docs`
- `implementation guide`

## 注意事項

- 元のSpecファイルは英語のまま維持されます
- ドキュメントは変更のたびに再生成されます（差分更新ではない）
- `specs/**/docs/` 配下のファイルは生成対象外（無限ループ防止）
- Specが不完全な場合、生成されるドキュメントにもその旨が記載されます
