# ユーザー通知システム 実装ガイド

> このドキュメントは実装者向けです。
> 生成元: `specs/user-notification/spec.md`

## 実装概要

通知送信、通知一覧取得、既読マークの3つのAPIエンドポイントを実装する。

## 前提条件

### 必要な知識

- Node.js / TypeScript
- Express フレームワーク
- PostgreSQL / SQL
- Redis / Bull (ジョブキュー)
- SendGrid API
- Firebase Cloud Messaging

### 環境セットアップ

```bash
# 依存関係インストール
pnpm install

# 環境変数設定
cp .env.example .env
# SENDGRID_API_KEY, FCM_SERVER_KEY などを設定

# データベースマイグレーション
pnpm run db:migrate

# 開発サーバー起動
pnpm run dev
```

## 詳細要件

### 1. Send Notification API

**エンドポイント**: `POST /api/notifications`

**リクエスト**:
```json
{
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "type": "push",
  "title": "New message",
  "body": "You have a new message from John",
  "priority": "high",
  "metadata": { "messageId": "abc123" }
}
```

**レスポンス (201)**:
```json
{
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "type": "push",
    "title": "New message",
    "status": "pending",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

### バリデーションルール

| フィールド | ルール | エラーコード |
|-----------|--------|-------------|
| userId | 必須、UUID形式 | INVALID_USER_ID |
| type | 必須、email/push/in_app | INVALID_TYPE |
| title | 必須、1-100文字 | INVALID_TITLE |
| body | 必須、1-1000文字 | INVALID_BODY |
| priority | low/normal/high | (デフォルト: normal) |

### エラーコード一覧

| HTTP | コード | 条件 | メッセージ例 |
|------|--------|------|-------------|
| 400 | INVALID_USER_ID | userIdが空またはUUID形式でない | "Invalid user ID format" |
| 400 | INVALID_TYPE | typeが不正 | "Type must be email, push, or in_app" |
| 400 | INVALID_TITLE | titleが空または100文字超 | "Title must be 1-100 characters" |
| 400 | INVALID_BODY | bodyが空または1000文字超 | "Body must be 1-1000 characters" |
| 404 | USER_NOT_FOUND | ユーザーが存在しない | "User not found" |
| 429 | RATE_LIMIT_EXCEEDED | レート制限超過 | "Too many requests" |

## エッジケース

| 入力 | 期待される動作 |
|------|----------------|
| `title: ""` (空文字) | 400 INVALID_TITLE |
| `title: "a".repeat(101)` (101文字) | 400 INVALID_TITLE |
| `userId: "invalid-uuid"` | 400 INVALID_USER_ID |
| `userId: "550e8400-..."` (存在しない) | 404 USER_NOT_FOUND |
| `type: "sms"` (未対応) | 400 INVALID_TYPE |
| 同じ通知を2回既読マーク | 200 (冪等、readAt更新なし) |

## エラーハンドリング

### バリデーションエラー

```typescript
// 400 Bad Request
res.status(400).json({
  error: {
    code: 'INVALID_TITLE',
    message: 'Title must be 1-100 characters'
  },
  meta: { requestId }
});
```

### 存在しないリソース

```typescript
// 404 Not Found
res.status(404).json({
  error: {
    code: 'USER_NOT_FOUND',
    message: 'User not found'
  },
  meta: { requestId }
});
```

### レート制限

```typescript
// 429 Too Many Requests
res.status(429).json({
  error: {
    code: 'RATE_LIMIT_EXCEEDED',
    message: 'Too many requests'
  },
  meta: { requestId }
});
```

## Worker実装

### メール配信Worker

```typescript
// workers/email.worker.ts
import { Job } from 'bull';
import sgMail from '@sendgrid/mail';

export async function processEmailJob(job: Job) {
  const { userId, title, body, metadata } = job.data;

  const user = await getUserById(userId);

  await sgMail.send({
    to: user.email,
    from: 'noreply@example.com',
    subject: title,
    text: body,
  });

  await updateNotificationStatus(job.data.notificationId, 'sent');
}
```

### プッシュ通知Worker

```typescript
// workers/push.worker.ts
import { Job } from 'bull';
import admin from 'firebase-admin';

export async function processPushJob(job: Job) {
  const { userId, title, body, metadata } = job.data;

  const tokens = await getUserPushTokens(userId);

  await admin.messaging().sendMulticast({
    tokens,
    notification: { title, body },
    data: metadata,
  });

  await updateNotificationStatus(job.data.notificationId, 'sent');
}
```

## テスト観点

### ユニットテスト

- [ ] バリデーション関数のテスト
  - 有効な入力で true を返す
  - 各エラーケースで適切なエラーを返す
- [ ] 通知ステータス更新のテスト
- [ ] 既読マークの冪等性テスト

### 統合テスト

- [ ] 正常系: 有効なリクエストで201が返る
- [ ] 正常系: 通知一覧取得でページネーションが動作する
- [ ] 異常系: 無効なuserIdで400が返る
- [ ] 異常系: 存在しないユーザーで404が返る
- [ ] 異常系: レート制限超過で429が返る

### E2Eテスト

- [ ] メール配信が30秒以内に完了する
- [ ] プッシュ通知が5秒以内に完了する

## 実装チェックリスト

### API実装

- [ ] POST /api/notifications エンドポイント作成
- [ ] GET /api/users/{userId}/notifications エンドポイント作成
- [ ] PATCH /api/notifications/{id}/read エンドポイント作成
- [ ] 入力バリデーション実装
- [ ] エラーハンドリング実装

### Worker実装

- [ ] メール配信Worker
- [ ] プッシュ通知Worker
- [ ] アプリ内通知処理

### データベース

- [ ] notificationsテーブル作成
- [ ] インデックス作成
- [ ] マイグレーションスクリプト

### 外部連携

- [ ] SendGrid連携設定
- [ ] Firebase連携設定

### テスト

- [ ] ユニットテスト作成
- [ ] 統合テスト作成
- [ ] カバレッジ 80%以上

### ドキュメント

- [ ] API仕様書更新
- [ ] CHANGELOG更新
