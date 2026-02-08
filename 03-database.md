# 03 — Database Schema

## 1. Purpose

This document defines the **logical database schema** for the Internal AI Chat Agent.

The database supports:

- User authentication and role management
- Conversation history and session memory
- Website content ingestion and indexing
- Retrieval and answer logging
- Feedback collection
- Auditability and traceability

This schema is designed to be **simple, normalized, and scalable**, while remaining implementation-agnostic.

---

## 2. Database Overview

The system uses:

- **PostgreSQL** for relational data
- **pgvector** extension for semantic embeddings

All tables described below are logical representations. Physical schema (indexes, constraints) will be defined during implementation.

---

## 3. Core Tables

---

### 3.1 `users`

Stores authenticated system users.

| Column | Type | Description |
|------|-----|-------------|
| id | UUID | Primary key |
| email | TEXT | Unique user email |
| name | TEXT | Display name |
| role | TEXT | `agent` or `admin` |
| is_active | BOOLEAN | Account status |
| created_at | TIMESTAMP | Account creation time |
| last_login_at | TIMESTAMP | Last login timestamp |

---

### 3.2 `roles`

Defines supported user roles.

| Column | Type | Description |
|------|-----|-------------|
| id | TEXT | Role identifier (`agent`, `admin`) |
| description | TEXT | Role description |

---

## 4. Conversation & Memory Tables

---

### 4.1 `conversations`

Represents a chat session tied to a user.

| Column | Type | Description |
|------|-----|-------------|
| id | UUID | Conversation ID |
| user_id | UUID | Owner of the conversation |
| created_at | TIMESTAMP | Start time |
| updated_at | TIMESTAMP | Last activity |
| is_active | BOOLEAN | Active/inactive |

---

### 4.2 `messages`

Stores individual chat messages.

| Column | Type | Description |
|------|-----|-------------|
| id | UUID | Message ID |
| conversation_id | UUID | Parent conversation |
| sender | TEXT | `user` or `assistant` |
| content | TEXT | Message text |
| created_at | TIMESTAMP | Message timestamp |

---

### 4.3 `conversation_summaries`

Optional rolling summary for long conversations.

| Column | Type | Description |
|------|-----|-------------|
| conversation_id | UUID | Conversation reference |
| summary | TEXT | Condensed conversation context |
| updated_at | TIMESTAMP | Last update time |

---

## 5. Website Content Ingestion Tables

---

### 5.1 `web_pages`

Stores metadata for ingested website pages.

| Column | Type | Description |
|------|-----|-------------|
| id | UUID | Page ID |
| url | TEXT | Page URL |
| title | TEXT | Page title |
| status | TEXT | `active`, `excluded`, `error` |
| last_crawled_at | TIMESTAMP | Last ingestion time |
| content_hash | TEXT | Change detection hash |
| created_at | TIMESTAMP | Record creation time |

---

### 5.2 `content_chunks`

Stores cleaned and chunked website text.

| Column | Type | Description |
|------|-----|-------------|
| id | UUID | Chunk ID |
| page_id | UUID | Source web page |
| chunk_index | INTEGER | Order in page |
| content | TEXT | Chunk text |
| token_count | INTEGER | Approximate token size |
| created_at | TIMESTAMP | Creation time |

---

### 5.3 `embeddings`

Stores vector embeddings for retrieval.

| Column | Type | Description |
|------|-----|-------------|
| id | UUID | Embedding ID |
| chunk_id | UUID | Linked content chunk |
| embedding | VECTOR | Vector embedding |
| created_at | TIMESTAMP | Creation time |

---

## 6. Feedback & Logging Tables

---

### 6.1 `feedback`

Stores user feedback for assistant responses.

| Column | Type | Description |
|------|-----|-------------|
| id | UUID | Feedback ID |
| message_id | UUID | Assistant message |
| user_id | UUID | User who submitted feedback |
| rating | TEXT | `up` or `down` |
| comment | TEXT | Optional comment |
| created_at | TIMESTAMP | Feedback time |

---

### 6.2 `audit_logs`

Tracks system-level actions for traceability.

| Column | Type | Description |
|------|-----|-------------|
| id | UUID | Log ID |
| user_id | UUID | Actor |
| action | TEXT | Action type |
| target | TEXT | Affected entity |
| created_at | TIMESTAMP | Action timestamp |

---

## 7. Relationships Summary

- `users` → `conversations` (one-to-many)
- `conversations` → `messages` (one-to-many)
- `web_pages` → `content_chunks` (one-to-many)
- `content_chunks` → `embeddings` (one-to-one)
- `messages` → `feedback` (one-to-many)
- `users` → `audit_logs` (one-to-many)

---

## 8. Retention & Governance

- Conversation data retained per system policy
- Embeddings rebuilt when source content changes
- Feedback and audit logs retained for internal review

---

## 9. Design Rationale

This schema:

- Separates ingestion, retrieval, and conversation concerns
- Supports strict grounding via indexed website content
- Enables RBAC through user-role association
- Allows clean analytics and auditing without exposing sensitive data
