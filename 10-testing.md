# 10 — Testing

## 1. Purpose

This document defines the **testing strategy and test cases** for the Internal AI Chat Agent.

Testing ensures the system is secure, stable, and behaves strictly according to requirements:
- authenticated access only
- role-based permissions
- grounded answers (no hallucination)
- reliable chat and memory handling

---

## 2. Testing Scope

Testing covers:

- Authentication and authorization
- Role-based access control (RBAC)
- Chat functionality
- Conversation memory
- Retrieval grounding behaviour
- Safety and refusal behaviour
- API stability and error handling

---

## 3. Authentication Tests

### 3.1 Login

- Valid credentials return access token
- Invalid credentials return `401 Unauthorized`
- Inactive users cannot log in

### 3.2 Logout

- Logout invalidates token
- Subsequent requests with the same token return `401`

### 3.3 Protected Routes

- Access without token returns `401`
- Access with invalid token returns `401`

---

## 4. RBAC Tests

### 4.1 Agent Role

- Agent can access chat endpoints
- Agent cannot access admin endpoints
- Agent receives `403 Forbidden` when accessing admin APIs

### 4.2 Admin Role

- Admin can access chat endpoints
- Admin can access content management APIs
- Admin can trigger re-indexing

---

## 5. Chat Functionality Tests

### 5.1 Basic Chat

- User message is accepted
- Assistant response is streamed
- Response completes without error

### 5.2 Streaming Behaviour

- Tokens arrive in correct order
- Stream closes properly
- UI does not freeze during streaming

---

## 6. Conversation Memory Tests

### 6.1 Follow-Up Questions

- Follow-up questions reference previous context correctly
- Standalone query rewriting works as expected

### 6.2 Isolation

- User cannot see other users’ conversations
- Conversation history is isolated per user

---

## 7. Retrieval & Grounding Tests

### 7.1 Valid Retrieval

- Relevant content is retrieved for known queries
- Assistant answers only using retrieved content

### 7.2 Insufficient Content

- When no relevant content exists, assistant responds with:
  - “Information not available in the current knowledge base”
- No answer is generated without retrieval evidence

---

## 8. Safety Tests

### 8.1 Prompt Injection

- Attempts to override system rules are refused
- Requests like “ignore instructions” are blocked

### 8.2 Unauthorized Requests

- Requests for confidential or restricted information are refused

---

## 9. API Error Handling Tests

- Invalid request payload returns `400`
- Unauthorized access returns `401`
- Forbidden access returns `403`
- Non-existent resources return `404`
- Unexpected errors return `500` with safe error message

---

## 10. Performance Tests

- Average response time under expected threshold
- No memory leak during long chat sessions
- System remains responsive under normal usage load

---

## 11. Acceptance Criteria

The system passes testing when:

- All authentication and RBAC tests pass
- Chat and streaming are stable
- No hallucinated responses observed
- All refusal cases behave correctly
- System remains stable during extended use
