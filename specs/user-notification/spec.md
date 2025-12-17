# User Notification System Specification

## Overview

A notification system that allows the application to send notifications to users through multiple channels (email, push, in-app).

## Purpose

Enable timely communication with users about important events, updates, and actions required.

## Scope

### In Scope
- Email notifications
- Push notifications (mobile)
- In-app notifications
- Notification preferences management
- Read/unread status tracking

### Out of Scope
- SMS notifications
- Third-party messaging integrations (Slack, Teams)
- Scheduled notifications (v2)

## Functional Requirements

### 1. Send Notification

**Endpoint**: `POST /api/notifications`

**Request**:
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

**Response (201)**:
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

### 2. Get User Notifications

**Endpoint**: `GET /api/users/{userId}/notifications`

**Query Parameters**:
- `status`: filter by read/unread
- `type`: filter by notification type
- `limit`: max results (default: 20, max: 100)
- `offset`: pagination offset

**Response (200)**:
```json
{
  "data": [
    {
      "id": "uuid",
      "type": "string",
      "title": "string",
      "body": "string",
      "read": false,
      "createdAt": "ISO 8601"
    }
  ],
  "meta": {
    "total": 150,
    "limit": 20,
    "offset": 0
  }
}
```

### 3. Mark as Read

**Endpoint**: `PATCH /api/notifications/{id}/read`

**Response (200)**:
```json
{
  "data": {
    "id": "uuid",
    "read": true,
    "readAt": "ISO 8601"
  }
}
```

## Non-Functional Requirements

- Email delivery: < 30 seconds
- Push notification delivery: < 5 seconds
- In-app notification: < 1 second
- Availability: 99.9%
- Concurrent users: 10,000

## Error Codes

| HTTP | Code | Condition |
|------|------|-----------|
| 400 | INVALID_USER_ID | Invalid or missing userId |
| 400 | INVALID_TYPE | Invalid notification type |
| 400 | INVALID_TITLE | Title empty or > 100 chars |
| 400 | INVALID_BODY | Body empty or > 1000 chars |
| 404 | USER_NOT_FOUND | User does not exist |
| 404 | NOTIFICATION_NOT_FOUND | Notification does not exist |
| 429 | RATE_LIMIT_EXCEEDED | Too many notifications |

## Edge Cases

| Input | Expected |
|-------|----------|
| Empty title | 400 INVALID_TITLE |
| Title > 100 chars | 400 INVALID_TITLE |
| Invalid userId format | 400 INVALID_USER_ID |
| Non-existent user | 404 USER_NOT_FOUND |
| Already read notification | 200 (idempotent) |

## Acceptance Criteria

### AC1: Send Email Notification
**Given** valid notification request with type=email
**When** POST /api/notifications
**Then** notification queued and email sent within 30 seconds

### AC2: Get Unread Notifications
**Given** user has 5 unread notifications
**When** GET /api/users/{userId}/notifications?status=unread
**Then** returns 5 notifications with read=false

### AC3: Mark Notification as Read
**Given** unread notification exists
**When** PATCH /api/notifications/{id}/read
**Then** notification marked as read with timestamp
