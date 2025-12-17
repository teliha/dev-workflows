# Sample Feature Specification

## Overview

This is a sample feature for testing the spec documentation generator skill.

## Purpose

Automatically generate Japanese documentation from English specification files for different stakeholders.

## Scope

### In Scope
- Basic CRUD operations
- User authentication
- Data validation

### Out of Scope
- Real-time notifications
- Third-party integrations

## Functional Requirements

### 1. Create Item

**Endpoint**: `POST /api/items`

**Request**:
```json
{
  "name": "string (1-100 chars, required)",
  "description": "string (max 1000 chars, optional)"
}
```

**Response (201)**:
```json
{
  "data": {
    "id": "uuid",
    "name": "string",
    "createdAt": "ISO 8601"
  }
}
```

## Non-Functional Requirements

- Response time: < 200ms (p95)
- Availability: 99.9%
- Concurrent users: 1000

## Edge Cases

| Input | Expected |
|-------|----------|
| Empty name | 400 INVALID_NAME |
| Name > 100 chars | 400 INVALID_NAME |

## Acceptance Criteria

### AC1: Create Item Success
**Given** valid input
**When** POST /api/items
**Then** 201 with item data
