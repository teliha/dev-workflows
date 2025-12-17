# ユーザー通知システム アーキテクチャ設計

> このドキュメントはアーキテクト・テックリード向けです。
> 生成元: `specs/user-notification/spec.md`

## 技術概要

複数チャネル（メール、プッシュ、アプリ内）に対応した通知配信システム。非同期処理によるスケーラブルな設計。

## システム構成

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │────▶│  API Server │────▶│   Queue     │
│             │◀────│             │     │  (Redis)    │
└─────────────┘     └─────────────┘     └──────┬──────┘
                           │                    │
                           ▼                    ▼
                    ┌─────────────┐     ┌─────────────┐
                    │  Database   │     │   Workers   │
                    │ (PostgreSQL)│     │             │
                    └─────────────┘     └──────┬──────┘
                                               │
                    ┌──────────────────────────┼──────────────────────────┐
                    ▼                          ▼                          ▼
             ┌─────────────┐           ┌─────────────┐           ┌─────────────┐
             │   Email     │           │    Push     │           │   In-App    │
             │  Provider   │           │  Provider   │           │   Store     │
             │  (SendGrid) │           │    (FCM)    │           │             │
             └─────────────┘           └─────────────┘           └─────────────┘
```

## API設計

### エンドポイント一覧

| Method | Path | 説明 |
|--------|------|------|
| POST | /api/notifications | 通知送信 |
| GET | /api/users/{userId}/notifications | ユーザーの通知一覧取得 |
| PATCH | /api/notifications/{id}/read | 既読マーク |

### 1. Send Notification API

**エンドポイント**: `POST /api/notifications`

**リクエスト**:
```json
{
  "userId": "uuid (required)",
  "type": "email | push | in_app (required)",
  "title": "string (1-100 chars, required)",
  "body": "string (1-1000 chars, required)",
  "priority": "low | normal | high (default: normal)",
  "metadata": "object (optional)"
}
```

**レスポンス (201)**:
```json
{
  "data": {
    "id": "uuid",
    "userId": "uuid",
    "type": "string",
    "title": "string",
    "status": "pending | sent | failed",
    "createdAt": "ISO 8601"
  }
}
```

### 2. Get User Notifications API

**エンドポイント**: `GET /api/users/{userId}/notifications`

**クエリパラメータ**:
| パラメータ | 型 | デフォルト | 説明 |
|-----------|-----|----------|------|
| status | string | - | read/unreadでフィルタ |
| type | string | - | 通知タイプでフィルタ |
| limit | number | 20 | 最大取得件数（max: 100） |
| offset | number | 0 | ページネーションオフセット |

**レスポンス (200)**:
```json
{
  "data": [...],
  "meta": {
    "total": 150,
    "limit": 20,
    "offset": 0
  }
}
```

### 3. Mark as Read API

**エンドポイント**: `PATCH /api/notifications/{id}/read`

**レスポンス (200)**:
```json
{
  "data": {
    "id": "uuid",
    "read": true,
    "readAt": "ISO 8601"
  }
}
```

## データモデル

### notifications テーブル

| カラム | 型 | 制約 | 説明 |
|--------|-----|------|------|
| id | UUID | PRIMARY KEY | 一意識別子 |
| user_id | UUID | NOT NULL, FK | 宛先ユーザー |
| type | VARCHAR(20) | NOT NULL | email/push/in_app |
| title | VARCHAR(100) | NOT NULL | 通知タイトル |
| body | TEXT | NOT NULL | 通知本文 |
| priority | VARCHAR(10) | DEFAULT 'normal' | 優先度 |
| status | VARCHAR(20) | DEFAULT 'pending' | 配信状態 |
| read | BOOLEAN | DEFAULT false | 既読フラグ |
| read_at | TIMESTAMP | NULLABLE | 既読日時 |
| metadata | JSONB | NULLABLE | 追加情報 |
| created_at | TIMESTAMP | NOT NULL | 作成日時 |

### インデックス

```sql
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_user_status ON notifications(user_id, read);
CREATE INDEX idx_notifications_created_at ON notifications(created_at DESC);
```

## 非機能要件

### パフォーマンス

| 指標 | 要件 |
|------|------|
| メール配信 | < 30秒 |
| プッシュ通知配信 | < 5秒 |
| アプリ内通知 | < 1秒 |
| API レスポンス | < 200ms (p95) |

### スケーラビリティ

| 指標 | 要件 |
|------|------|
| 同時接続ユーザー数 | 10,000 |
| 日次通知送信数 | 100万件 |
| 水平スケール | Worker数で調整可能 |

### 可用性

| 指標 | 要件 |
|------|------|
| SLA | 99.9% |
| フェイルオーバー | キュー永続化で再試行 |

### セキュリティ

- 認証: JWT Bearer Token
- 認可: ユーザーは自分の通知のみアクセス可能
- メール: SPF/DKIM/DMARC設定
- プッシュ: サーバーキーの安全な管理

## 依存関係

### 外部サービス

| サービス | 用途 |
|---------|------|
| SendGrid | メール配信 |
| Firebase Cloud Messaging | プッシュ通知 |

### インフラ

| コンポーネント | 用途 |
|---------------|------|
| PostgreSQL | データ永続化 |
| Redis | ジョブキュー、キャッシュ |

## 設計上の決定事項

### 決定1: 非同期処理の採用

**理由**: 配信遅延をAPIレスポンスから分離し、スケーラビリティを確保

**代替案**: 同期処理
**不採用理由**: メール配信等で30秒以上かかる可能性があり、APIタイムアウトの原因になる

### 決定2: チャネル別Worker分離

**理由**: チャネルごとに異なるSLAに対応、障害影響を局所化

**代替案**: 統合Worker
**不採用理由**: プッシュ通知（5秒）とメール（30秒）で要件が異なる
