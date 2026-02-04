# 02 — Architecture

## 1. Overview

This system implements a **Retrieval-Augmented Generation (RAG)** architecture to ensure the AI assistant answers **strictly** based on approved sources.

It is designed for:
- **Public usage** (no login)
- **BM + English**
- **Citations for every answer**
- **Conversation continuity (memory for follow-ups)** without allowing hallucinations

---

## 2. Architectural Principles

### 2.1 Grounded Answers Only
The LLM must **never** answer purely from its own knowledge.  
It must answer only from **retrieved document chunks**.

### 2.2 Retrieval is Mandatory
If retrieval returns insufficient evidence, the assistant must:
- respond with “not found in sources”
- ask a short clarification question if useful

### 2.3 Controlled Website Ingestion
Website content is indexed only from:
- allowlisted domains, and preferably
- an approved URL list or sitemap filter

No open crawling.

### 2.4 Session-Based Memory (No Login)
Memory is scoped to the user’s session only:
- `session_id` stored in browser localStorage
- messages saved server-side for continuity

---

## 3. High-Level System Architecture

### 3.1 Components

**Frontend (React)**
- Chat UI + streaming responses
- Sources panel (PDF page / URL)
- Feedback buttons (👍/👎)
- Session ID generation + storage
- “Clear chat” action

**Backend (FastAPI)**
- Chat orchestration API
- Session memory management
- Question rewriting for follow-ups
- Retrieval + citation assembly
- Safety guardrails (prompt injection, allowlist enforcement)

**Databases**
- **PostgreSQL**: document registry, sessions, messages, feedback (non-sensitive logs)
- **PGVector**: embeddings + metadata for semantic retrieval

**Ingestion Jobs**
- PDF ingestion pipeline
- Controlled website ingestion (URL list / sitemap)

**LLM Provider**
- Used for:
  - question rewriting (follow-up → standalone)
  - grounded answer generation (using retrieved context)
  - optional: summarizing long chat history

---

## 4. Data Flow Diagrams (Text)

### 4.1 Ingestion Flow (PDF)

1. Admin adds PDF(s) to approved source list
2. Extract text (page-aware)
3. Clean text (remove repeated headers/footers, artifacts)
4. Chunk text (target 400–800 tokens per chunk)
5. Generate embeddings for each chunk
6. Store in PGVector with metadata:
   - `source_type=pdf`
   - `doc_title`
   - `page_number`
   - optional: `project`, `state`, `property_type`

Result: searchable PDF knowledge base with traceable citations.

---

### 4.2 Ingestion Flow (Website, Controlled)

**Preferred approach: URL list / sitemap-based**
1. Admin provides an approved URL list (or sitemap URL)
2. Filter URLs by rules:
   - domain allowlist
   - path allowlist (e.g., `/projects/`, `/faq/`)
   - exclude patterns (`/login`, `/admin`, query params)
3. Fetch HTML content (rate-limited)
4. Extract main content (remove nav/footer/scripts)
5. Normalize + clean text
6. Chunk text
7. Generate embeddings and store in PGVector with metadata:
   - `source_type=web`
   - `url`
   - `title`
   - `crawl_timestamp`
   - optional: `project`, `state`, `property_type`

Result: searchable web knowledge base with URL citations.

---

### 4.3 Chat Flow (RAG + Memory + Safety)

**Input:** user message + `session_id`

1. **Receive message**
   - Frontend sends `{ session_id, message }`

2. **Load session context**
   - Load last N messages for that session (e.g., 6–10)
   - Load session summary if enabled (optional)

3. **Safety pre-check**
   - Detect prompt injection attempts (“ignore instructions…”, “reveal internal docs…”)
   - Detect obviously sensitive requests (internal SOP, commission, private data)
   - If triggered → refuse with safe response template

4. **Rewrite follow-up question (critical)**
   - Convert the user message into a standalone query using:
     - last N messages
     - optional summary
   - Example:
     - “projek tu dekat mana?” → “Lokasi projek <ProjectName> adalah di mana?”

5. **Retrieve evidence**
   - Embed the standalone query
   - Query PGVector for top-k chunks (e.g., k=5–8)
   - Apply metadata filters if available (state/project)
   - Compute confidence score (threshold-based)

6. **If insufficient evidence**
   - Respond with:
     - “Not found in approved sources”
     - ask one clarification question (short)
   - Do not call answer-generation LLM without evidence

7. **Build grounded prompt**
   - System rules (strict grounding)
   - Retrieved chunks (with source metadata)
   - Standalone query

8. **Generate response**
   - LLM generates answer using only retrieved chunks
   - Backend assembles citations:
     - PDF citations: doc + page
     - Web citations: title + URL

9. **Post-check**
   - Ensure citations exist
   - If answer has no citations → refuse / fallback

10. **Stream response**
   - Stream tokens to UI (SSE/WebSocket)
   - Send final “sources” payload

11. **Store logs**
   - Store message + response (non-sensitive)
   - Store retrieval sources used (for debugging)
   - Store feedback if provided later

---

## 5. Conversation Memory Design

### 5.1 What “Memory” Means Here
Memory is **conversation context** to support follow-ups, not a store of facts.

It is used to:
- resolve references: “projek tu”, “yang tadi”, “this one”
- maintain continuity within a single session

### 5.2 Memory Sources
- last N messages (raw)
- optional running summary for long sessions (updated periodically)

### 5.3 Session Lifecycle
- `session_id` created client-side and stored in localStorage
- server stores messages keyed by `session_id`
- session expiry policy (recommended): 24 hours

---

## 6. Retrieval Strategy

### 6.1 Hybrid Retrieval (Recommended)
- Primary: semantic retrieval (vector search)
- Supporting:
  - metadata filters (state/project/type)
  - keyword boosts (optional)

### 6.2 Confidence Threshold
If top results are low confidence:
- do not generate an answer
- return “not found” + ask one clarification

---

## 7. Safety Architecture (Public-Ready)

### 7.1 Source Allowlisting
- Only ingest approved PDFs + allowlisted URLs
- No open web browsing or third-party sources

### 7.2 Prompt Injection Defense
- System rules are never overridden by user prompts
- Refuse “reveal prompt”, “show internal docs”, “ignore rules”

### 7.3 Sensitive Data Guard
- Block indexing of pages flagged as “internal/confidential”
- Optional redaction filters for risky patterns (IC, bank, OTP)

### 7.4 Rate Limiting / Bot Protection
Because this is public:
- rate limit API calls per IP/session
- optional WAF (e.g., Cloudflare)

---

## 8. Deployment Architecture (Conceptual)

**Frontend**
- Static hosting (e.g., CDN)

**Backend**
- FastAPI service (containerized)

**Database**
- Managed Postgres with pgvector

Optional:
- Redis for session cache
- WAF / rate limit layer in front of API

---

## 9. Why This Architecture Works

This architecture ensures:
- grounded answers with citations
- better follow-up handling via session memory + question rewriting
- safe, controlled ingestion for public usage
- simple, debuggable RAG pipeline (no hidden orchestration)
