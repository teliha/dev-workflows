# Sample Feature 実装ガイド

> このドキュメントは実装者向けです。
> 生成元: `specs/sample-feature/spec.md`

## 実装概要

アイテム作成APIのエンドポイントを実装する。入力バリデーション、データベース操作、レスポンス生成を含む。

## 前提条件

### 必要な知識

- Node.js / TypeScript
- Express フレームワーク
- PostgreSQL / SQL
- RESTful API設計

### 環境セットアップ

```bash
# 依存関係インストール
pnpm install

# データベースマイグレーション
pnpm run db:migrate

# 開発サーバー起動
pnpm run dev
```

## 詳細要件

### 1. Create Item API

**エンドポイント**: `POST /api/items`

**リクエスト**:
```json
{
  "name": "string (1-100 chars, required)",
  "description": "string (max 1000 chars, optional)"
}
```

**レスポンス (201)**:
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Sample Item",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

### バリデーションルール

| フィールド | ルール | エラーコード |
|-----------|--------|-------------|
| name | 必須 | INVALID_NAME |
| name | 1-100文字 | INVALID_NAME |
| name | 文字列型 | INVALID_NAME |
| description | 最大1000文字 | INVALID_DESCRIPTION |
| description | 文字列型（任意） | INVALID_DESCRIPTION |

### エラーコード一覧

| HTTP | コード | 条件 | メッセージ例 |
|------|--------|------|-------------|
| 400 | INVALID_NAME | 名前が空 | "Name is required" |
| 400 | INVALID_NAME | 名前が100文字超 | "Name must be 1-100 characters" |
| 400 | INVALID_DESCRIPTION | 説明が1000文字超 | "Description must be max 1000 characters" |
| 401 | UNAUTHORIZED | 認証トークンなし | "Authentication required" |
| 500 | INTERNAL_ERROR | サーバーエラー | "Internal server error" |

## エッジケース

| 入力 | 期待される動作 |
|------|----------------|
| `name: ""` (空文字) | 400 INVALID_NAME |
| `name: null` | 400 INVALID_NAME |
| `name: undefined` (キーなし) | 400 INVALID_NAME |
| `name: "a".repeat(101)` (101文字) | 400 INVALID_NAME |
| `name: "a".repeat(100)` (100文字) | 201 成功 |
| `name: "a"` (1文字) | 201 成功 |
| `description: null` | 201 成功 (nullは許可) |
| `description: undefined` | 201 成功 (省略可) |
| `description: "a".repeat(1001)` | 400 INVALID_DESCRIPTION |
| Content-Type: text/plain | 400 INVALID_REQUEST |

## エラーハンドリング

### バリデーションエラー

```typescript
// 400 Bad Request
res.status(400).json({
  error: {
    code: 'INVALID_NAME',
    message: 'Name is required and must be 1-100 characters'
  },
  meta: { requestId }
});
```

### 認証エラー

```typescript
// 401 Unauthorized
res.status(401).json({
  error: {
    code: 'UNAUTHORIZED',
    message: 'Authentication required'
  },
  meta: { requestId }
});
```

### サーバーエラー

```typescript
// 500 Internal Server Error
// 注意: 詳細なエラー情報はログに記録し、クライアントには返さない
res.status(500).json({
  error: {
    code: 'INTERNAL_ERROR',
    message: 'Internal server error'
  },
  meta: { requestId }
});
```

## テスト観点

### ユニットテスト

- [ ] バリデーション関数のテスト
  - 有効な入力で true を返す
  - 無効な入力で適切なエラーを返す
- [ ] UUID生成のテスト
- [ ] 日時フォーマットのテスト

### 統合テスト

- [ ] 正常系: 有効なリクエストで201が返る
- [ ] 異常系: 無効なnameで400が返る
- [ ] 異常系: 認証なしで401が返る
- [ ] 境界値: 100文字のnameで成功
- [ ] 境界値: 101文字のnameで失敗

### パフォーマンステスト

- [ ] p95レスポンスタイム < 200ms
- [ ] 同時1000リクエストで正常動作

## 実装チェックリスト

### 基本実装

- [ ] エンドポイント `POST /api/items` の作成
- [ ] リクエストボディのパース
- [ ] 入力バリデーション実装
- [ ] UUIDの生成
- [ ] データベースへの保存
- [ ] レスポンスの返却

### バリデーション

- [ ] name: 必須チェック
- [ ] name: 文字数チェック (1-100)
- [ ] description: 文字数チェック (max 1000)
- [ ] Content-Type: application/json チェック

### エラーハンドリング

- [ ] バリデーションエラー (400)
- [ ] 認証エラー (401)
- [ ] サーバーエラー (500)
- [ ] エラーログ出力

### テスト

- [ ] ユニットテスト作成
- [ ] 統合テスト作成
- [ ] テストカバレッジ 80%以上

### ドキュメント

- [ ] API仕様書更新
- [ ] CHANGELOG更新
