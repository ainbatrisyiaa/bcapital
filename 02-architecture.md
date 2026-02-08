# 02 — Architecture

## 1. Overview

This system implements a **Retrieval-Augmented Generation (RAG)** architecture to ensure the AI assistant answers **strictly** based on approved and ingested website content.

The system is designed for:

- Internal usage only (login required)
- Role-based access (Agent and Admin)
- Bahasa Melayu and English
- Grounded answers only (no hallucination)
- Conversation continuity via authenticated session memory

---

## 2. Architectural Principles

### 2.1 Grounded Answers Only
The LLM must **never** answer from its own knowledge.

All responses must be generated **only from retrieved website content** that has been ingested and indexed.

---

### 2.2 Retrieval is Mandatory
If retrieval returns insufficient or low-confidence content, the assistant must:

- clearly state that the information is not available
- optionally ask one short clarification question

The system must not generate answers without evidence.

---

### 2.3 Controlled Website Content Ingestion
Website content is ingested only from:

- approved company website domains
- approved URL lists or sitemap filters

There is **no open web browsing**.

---

### 2.4 Authenticated Session Memory
Conversation memory is tied to an **authenticated user session**.

- Session identity is derived from the logged-in user
- Memory is scoped per user
- Conversation history is isolated between users

---

## 3. High-Level System Architecture

### 3.1 Components

**Frontend (Next.js + Tailwind)**
- Secure login and role handling
- Chat interface with streaming responses
- Conversation history per user
- Feedback controls (👍 / 👎)

**Backend (FastAPI)**
- Authenticated chat orchestration API
- Token validation and role enforcement
- Conversation memory management
- Follow-up question rewriting
- Retrieval and answer generation
- Safety guardrails

**Databases**
- **PostgreSQL**
  - users
  - roles
  - conversations
  - messages
  - feedback
  - audit logs
- **Vector Database (pgvector)**
  - embedded website content
  - semantic retrieval metadata

**Ingestion Jobs**
- Controlled website content ingestion
- Manual or scheduled re-indexing

**LLM Provider**
- Used for:
  - rewriting follow-up questions into standalone queries
  - generating grounded answers from retrieved content

---

## 4. Data Flow

### 4.1 Website Content Ingestion Flow

1. Admin defines approved domains or URL lists
2. Ingestion job fetches website pages
3. HTML is cleaned (remove navigation, footer, scripts)
4. Text is normalized and deduplicated
5. Content is chunked (target 400–800 tokens)
6. Embeddings are generated
7. Data is stored in pgvector with metadata:
   - `url`
   - `page_title`
   - `ingested_at`
   - optional tags (project, location, type)

Result: searchable knowledge base sourced from website content.

---

### 4.2 Chat Flow (RAG + Auth + Memory)

**Input:** authenticated user request + message

1. **Receive message**
   - Frontend sends `{ message }` with auth token

2. **Validate user and role**
   - Backend validates token
   - Enforces Agent/Admin access rules

3. **Load conversation memory**
   - Load recent messages for the user

4. **Safety pre-check**
   - Detect prompt injection attempts
   - Detect unauthorized or sensitive requests
   - If triggered → refuse with safe response

5. **Rewrite follow-up question**
   - Convert follow-up into a standalone query using conversation context

6. **Retrieve relevant content**
   - Embed the rewritten query
   - Query vector database for top-k results
   - Apply optional metadata filters
   - Evaluate confidence score

7. **If insufficient evidence**
   - Respond with:
     - “Information not available in the current knowledge base”
     - Optional clarification question
   - Stop processing

8. **Build grounded prompt**
   - System rules (strict grounding)
   - Retrieved content chunks
   - Standalone user query

9. **Generate response**
   - LLM generates answer using retrieved content only

10. **Stream response**
    - Tokens streamed back to the UI
    - Final response marked as complete

11. **Store logs**
    - Store message and response
    - Store feedback if provided

---

## 5. Conversation Memory Design

### 5.1 Scope
Memory exists to support **follow-up understanding**, not long-term knowledge storage.

### 5.2 Memory Sources
- Recent messages (last N turns)

### 5.3 Retention
- Conversation history retained per user
- Retention window configurable by system policy

---

## 6. Retrieval Strategy

### 6.1 Semantic Retrieval
- Vector similarity search as the primary mechanism

### 6.2 Metadata Filtering
- Filter by project, location, or property type when available

### 6.3 Confidence Threshold
- Low-confidence retrieval results trigger fallback responses
- Prevents hallucinated answers

---

## 7. Safety Architecture

### 7.1 Content Governance
- Only approved website content is indexed
- Restricted or confidential pages are excluded

### 7.2 Prompt Injection Defense
- System instructions cannot be overridden
- Requests to bypass rules are refused

### 7.3 Role-Based Enforcement
- Retrieval scope enforced by user role
- Admin-only information inaccessible to Agents

---

## 8. Deployment Architecture (Conceptual)

**Frontend**
- Next.js application with authentication

**Backend**
- FastAPI service (containerized)

**Database**
- Managed PostgreSQL with pgvector
