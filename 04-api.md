# 04 — API Specifications

## 1. Purpose

This document defines the backend API contract for the Internal AI Chat Agent.

The API supports authentication, AI chat interactions, conversation memory handling, website content ingestion control, feedback submission, and system health checks.

All APIs are consumed by the Next.js frontend and enforced at the backend.

---

## 2. General API Rules

- All endpoints except `/health` require authentication
- Authentication uses secure tokens (JWT or session token)
- Role-based access control (RBAC) is enforced server-side
- Request and response payloads use JSON unless stated otherwise
- Streaming responses use Server-Sent Events (SSE)

---

## 3. Authentication APIs

### 3.1 POST /auth/login

Authenticate a user and start a session.

**Request**
```json
{
  "email": "user@company.com",
  "password": "********"
}
```

**Response**
```json
{
  "access_token": "jwt_token_here",
  "user": {
    "id": "uuid",
    "name": "User Name",
    "role": "agent"
  }
}
```

### 3.2 POST /auth/logout

Invalidate the current session.

**Request Headers**

```Authorization: Bearer <token>```


**Response**
```
{
  "success": true
}
```

## 4. Chat APIs

### 4.1 POST /chat/stream

Send a user message and receive a streamed AI response.

**Authentication**

```Required```

**Request**

```
{
  "message": "Apakah lokasi projek A?"
}


Response (Server-Sent Events)

event: token
data: "Projek A terletak di ..."

event: token
data: " Lokasinya berhampiran ..."

event: done
data: {
  "confidence": "high"
}
```

### 4.2 GET /chat/history

Retrieve past conversations for the logged-in user.

**Authentication**

```Required```

**Response**

```
[
  {
    "conversation_id": "uuid",
    "updated_at": "timestamp"
  }
]
```

### 4.3 GET /chat/history/{conversation_id}

Retrieve messages for a specific conversation.

**Authentication**

Required

**Response**

```
{
  "conversation_id": "uuid",
  "messages": [
    {
      "sender": "user",
      "content": "Soalan..."
    },
    {
      "sender": "assistant",
      "content": "Jawapan..."
    }
  ]
}
```

### 4.4 POST /chat/clear

Clear the current conversation context.

**Authentication**

Required

**Response**

```
{
  "success": true
}
```

## 5. Website Content Management APIs (Admin Only)

### 5.1 GET /content/pages

Retrieve a list of ingested website pages.

**Authentication**

Required

**Role**

Admin only

```
Response

[
  {
    "id": "uuid",
    "url": "https://example.com/project-a",
    "status": "active",
    "last_crawled_at": "timestamp"
  }
]
```

## 5.2 POST /content/reindex

Trigger re-indexing of approved website content.

**Authentication**

Required

**Role**

Admin only

**Request**

```
{
  "mode": "full"
}
```

**Response**

```
{
  "status": "started"
}
```

### 5.3 POST /content/exclude

Exclude a website page from the knowledge base.

**Authentication**

Required

**Role**

Admin only

**Request**

```
{
  "url": "https://example.com/page-to-exclude"
}
```

**Response**

```
{
  "success": true
}
```

## 6. Feedback APIs

### 6.1 POST /feedback

Submit feedback on an assistant response.

**Authentication**

Required

**Request**

```
{
  "message_id": "uuid",
  "rating": "up",
  "comment": "Jawapan jelas"
}
```

**Response**

```
{
  "success": true
}
```

## 7. System APIs
### 7.1 GET /health

Check system health.

**Authentication**

Not required

**Response**

```
{
  "status": "ok"
}
```

## 8. Error Handling

All error responses follow this format:

```
{
  "error": "Error message"
}
```

| Status Code | Meaning |
|------------|---------|
| 400 | Bad request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not found |
| 500 | Internal server error |


## 9. Security Enforcement

- Authentication required for all protected endpoints

- RBAC enforced at backend level

- No trust in frontend role claims

- Admin-only access for content management APIs

- All actions logged for audit purposes
