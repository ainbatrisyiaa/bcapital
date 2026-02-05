# 04 — API Specifications

This document defines the complete API contract between the React frontend and the FastAPI backend.

System goals:

- Streaming chat responses (ChatGPT-like)
- Strict RAG grounding with citations
- Session-based memory (no login)
- Feedback collection
- Public-safe behavior

Base path example: `/api`

---

## 1. Conventions

### 1.1 Content Types

- Request: `application/json`
- Streaming response: `text/event-stream` (SSE)

### 1.2 Session ID

- Frontend generates a UUID called `session_id`
- Stored in browser localStorage
- Sent with every chat request

### 1.3 Language Handling

- Backend detects BM / English automatically
- Response must match user language

---

## 2. Data Models

### 2.1 Chat Request

```json
{
  "session_id": "uuid-string",
  "message": "string",
  "client_ts": "2026-02-04T10:00:00+08:00"
}
```

---

### 2.2 Source Object (Citation)

PDF example:

```json
{
  "source_type": "pdf",
  "title": "Project_Brochure_ABC.pdf",
  "locator": "page 7",
  "url": null,
  "snippet": "Eligible applicants must ...",
  "score": 0.82
}
```

Web example:

```json
{
  "source_type": "web",
  "title": "Project ABC - Pricing",
  "locator": "https://bcapital.com.my/projects/abc/pricing",
  "url": "https://bcapital.com.my/projects/abc/pricing",
  "snippet": "Price starts from ...",
  "score": 0.79
}
```

---

### 2.3 Final Chat Response Payload

Sent in the final SSE event.

```json
{
  "message_id": "uuid-string",
  "session_id": "uuid-string",
  "answer": "string",
  "sources": [],
  "language": "bm",
  "status": "ok"
}
```

---

### 2.4 Error Payload

```json
{
  "status": "error",
  "error_code": "NOT_FOUND_IN_SOURCES",
  "message": "Maklumat tidak dijumpai dalam sumber yang tersedia."
}
```

---

## 3. Endpoints

---

## 3.1 POST `/chat/stream`

Streams assistant response using Server-Sent Events (SSE).

### Request

Headers:
```
Content-Type: application/json
```

Body:

```json
{
  "session_id": "uuid-string",
  "message": "string"
}
```

---

### Response

Status: `200 OK`  
Content-Type: `text/event-stream`

---

### SSE Events

#### Event: `token`

```text
event: token
data: {"delta":"Hello "}
```

Sent multiple times as tokens are generated.

---

#### Event: `final`

```text
event: final
data: {"message_id":"...","session_id":"...","answer":"...","sources":[...],"language":"bm","status":"ok"}
```

Sent once when answer is complete.

---

#### Event: `error`

```text
event: error
data: {"status":"error","error_code":"NOT_FOUND_IN_SOURCES","message":"Maklumat tidak dijumpai dalam sumber yang tersedia."}
```

---

### Behavioral Rules

- Must enforce strict source grounding
- If retrieval insufficient → return error or refusal
- Must store user and assistant messages
- Must include citations in normal answers

---

## 3.2 POST `/feedback`

Submit feedback for a specific assistant message.

### Request

```json
{
  "message_id": "uuid-string",
  "session_id": "uuid-string",
  "rating": 1,
  "comment": "Helpful answer"
}
```

### Response

```json
{
  "status": "ok"
}
```

---

## 3.3 POST `/session/clear`

Clears session memory for privacy.

### Request

```json
{
  "session_id": "uuid-string"
}
```

### Response

```json
{
  "status": "ok"
}
```

---

## 3.4 GET `/health`

Health check endpoint.

### Response

```json
{
  "status": "ok",
  "service": "ai-chat-agent"
}
```

---

## 3.5 (Admin) POST `/ingest/pdf`

Upload or trigger PDF ingestion (Admin only).

### Request

```json
{
  "file_path": "data/pdfs/projectA.pdf"
}
```

### Response

```json
{
  "status": "ok",
  "chunks_indexed": 124
}
```

---

## 3.6 (Admin) POST `/ingest/web`

Trigger website ingestion from approved URL list.

### Request

```json
{
  "source": "data/urls/approved_urls.txt"
}
```

### Response

```json
{
  "status": "ok",
  "pages_indexed": 32
}
```

---

## 4. Error Codes

| Code | Meaning |
|------|---------|
| NOT_FOUND_IN_SOURCES | No relevant chunks found |
| PROMPT_INJECTION_DETECTED | Attempt to override rules |
| SENSITIVE_REQUEST_BLOCKED | Internal/confidential info requested |
| RATE_LIMITED | Too many requests |
| INTERNAL_ERROR | Unexpected backend error |

---

## 5. Frontend Responsibilities

Frontend must:

- Store `session_id` in localStorage
- Handle SSE events (`token`, `final`, `error`)
- Render citations after `final`
- Provide "Clear Chat" button
- Provide feedback buttons
