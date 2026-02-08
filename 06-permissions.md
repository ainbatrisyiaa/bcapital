# 06 — Permissions (RBAC)

## 1. Purpose

This document defines the Role-Based Access Control (RBAC) rules for the Internal AI Chat Agent.

RBAC ensures:
- Only authenticated users can access the system
- Agents and Admins have different capabilities
- All permission checks are enforced at the backend (frontend restrictions are UX only)

---

## 2. Roles

### 2.1 Agent
Primary user role for registered real estate agents.

**Allowed**
- Access chat interface
- Create and use conversations
- View own conversation history
- Clear own conversation context
- Submit feedback (👍 / 👎)

**Not Allowed**
- Access admin routes/pages
- Manage website ingestion settings
- Trigger re-indexing
- Exclude pages from knowledge base
- View other users’ conversations or logs

---

### 2.2 Admin
Administrative role for system and content management.

**Allowed**
- All Agent permissions
- Access admin dashboard/pages
- View ingested pages list
- Trigger content re-indexing
- Exclude pages from the knowledge base
- View audit logs (high-level system actions)

**Not Allowed**
- None within system scope (subject to company policy)

---

## 3. Permission Matrix

| Feature / Action | Agent | Admin |
|------------------|:-----:|:-----:|
| Login | ✅ | ✅ |
| Access chat | ✅ | ✅ |
| Stream AI response | ✅ | ✅ |
| Create conversation | ✅ | ✅ |
| View own conversation history | ✅ | ✅ |
| Clear own conversation | ✅ | ✅ |
| Submit feedback | ✅ | ✅ |
| Access admin pages | ❌ | ✅ |
| List ingested pages | ❌ | ✅ |
| Trigger re-index | ❌ | ✅ |
| Exclude page URL | ❌ | ✅ |
| View audit logs | ❌ | ✅ |

---

## 4. Enforcement Rules

### 4.1 Backend is Source of Truth
- Backend validates token on every request
- Backend determines the user role
- Backend rejects unauthorized actions

### 4.2 Frontend Restrictions are UX Only
- Frontend may hide admin menus for Agents
- Frontend must not be relied on for security

### 4.3 Endpoint-Level Protection
Admin-only endpoints:
- `GET /content/pages`
- `POST /content/reindex`
- `POST /content/exclude`

If an Agent calls these endpoints, backend returns:
- `403 Forbidden`

---

## 5. Data Access Rules

### 5.1 Conversations
- Users can only access conversations where `conversations.user_id == current_user.id`
- Admin role does not automatically grant access to other users’ conversations unless explicitly added in a future scope

### 5.2 Logs and Audit
- Audit logs are recorded for admin and system actions
- Audit log viewing is Admin-only

---

## 6. Security Notes

- Token must be validated server-side
- Role must be derived from backend user record
- No trust in frontend-provided role claims
- All permission denials should return consistent error responses
