# 08 — Implementation Phases (Phase-by-Phase Checklist)

Dokumen ini ialah **manual teknikal** untuk membina sistem AI Public Chat Agent secara berfasa.  
Anda boleh tandakan `[x]` bila selesai setiap tugasan.

**Stack**: React (Vite) + FastAPI (Python) + PostgreSQL + PGVector  
**Konsep**: RAG (strict sources) + Session Memory (follow-up) + Controlled Crawling (allowlist/URL list)  
**Nota**: MVP adalah **public (no login)**, tetapi admin endpoints akan disediakan untuk ingestion (protected later).

---

## Phase 0 — Project Setup & Documentation (Week 1–2)

### Objective
Sediakan blueprint, repo, dan environment supaya coding berjalan lancar.

### Checklist
- [ ] Create repo + folder structure:
  - [ ] `/docs`
  - [ ] `/backend`
  - [ ] `/frontend`
- [ ] Setup git + `.gitignore` (Python + Node)
- [ ] Finalize docs (Tier 2):
  - [ ] `01-requirements.md`
  - [ ] `02-architecture.md`
  - [ ] `03-database-schema.md`
  - [ ] `04-api-specifications.md`
  - [ ] `05-components.md`
  - [ ] `06-permissions.md`
  - [ ] `08-implementation-phases.md`
- [ ] Define governance lists:
  - [ ] Approved PDF list (folder /data/pdfs)
  - [ ] Allowlisted domain(s)
  - [ ] Approved URL list OR sitemap strategy
- [ ] Decide session expiry policy (recommended: 24 hours)

### Deliverables
- [ ] Repo ready
- [ ] Documentation approved
- [ ] Source list ready (PDF + URL)

---

## Phase 1 — Database & Core RAG (PDF Only) (Week 3–4)

### Objective
AI boleh “membaca” PDF dan retrieve chunks dengan betul + citation.

### Checklist

#### 1.1 Database Setup
- [ ] Install PostgreSQL 15+ (local/dev)
- [ ] Create database (e.g., `realestate_ai`)
- [ ] Enable extensions:
  - [ ] `CREATE EXTENSION vector;`
  - [ ] `CREATE EXTENSION "uuid-ossp";`
- [ ] Run schema SQL (from `03-database-schema.md`):
  - [ ] `documents`
  - [ ] `chunks`
  - [ ] `chat_sessions`
  - [ ] `chat_messages`
  - [ ] `chat_summaries` (optional)
  - [ ] `feedback`

#### 1.2 PDF Ingestion Script
- [ ] Create script `backend/scripts/ingest_pdf.py`
- [ ] Load PDFs from `/data/pdfs`
- [ ] Extract text per page (page-aware)
- [ ] Clean text:
  - [ ] remove repeated header/footer
  - [ ] normalize whitespace
- [ ] Chunking:
  - [ ] chunk size 400–800 tokens (or 800–1200 chars)
  - [ ] overlap 10–20%
- [ ] Generate embeddings (OpenAI embeddings)
- [ ] Insert into DB:
  - [ ] add row in `documents`
  - [ ] add rows in `chunks` with metadata:
    - [ ] source_type=pdf
    - [ ] title
    - [ ] page_number
    - [ ] metadata jsonb (project/state/type if available)

#### 1.3 Retrieval Test (CLI)
- [ ] Create script `backend/scripts/test_retrieval.py`
- [ ] Input: question text
- [ ] Output:
  - [ ] top-k chunks
  - [ ] scores
  - [ ] citations (doc title + page)

### Exit Criteria
- [ ] 최소 20 soalan test boleh retrieve chunk yang betul
- [ ] Citation boleh trace (PDF name + page)

---

## Phase 2 — Chat API + Session Memory (Week 5–6)

### Objective
Jadikan RAG engine satu chat service yang boleh follow-up.

### Checklist

#### 2.1 FastAPI Setup
- [ ] Create FastAPI app (`backend/app/main.py`)
- [ ] Enable CORS for frontend origin (e.g., `http://localhost:5173`)
- [ ] Add env config (`.env`):
  - [ ] `OPENAI_API_KEY`
  - [ ] `DATABASE_URL`

#### 2.2 Session Memory
- [ ] Implement session create-on-demand:
  - [ ] if session_id not found in `chat_sessions`, create it
- [ ] Store messages into `chat_messages`
- [ ] Load last N messages (e.g., 6–10) for context
- [ ] Implement `/session/clear`

#### 2.3 Follow-up Handling (Question Rewriting)
- [ ] Implement question rewriting:
  - [ ] input: last N messages + current question
  - [ ] output: standalone question
- [ ] Use standalone question for retrieval

#### 2.4 `/chat/stream` SSE Endpoint
- [ ] Implement SSE streaming:
  - [ ] `token` events
  - [ ] `final` event (answer + sources)
  - [ ] `error` event
- [ ] Strict grounding enforcement:
  - [ ] if retrieval confidence low → return NOT_FOUND_IN_SOURCES (BM/EN)
- [ ] Store assistant answer + sources JSON in `chat_messages`

### Exit Criteria
- [ ] Follow-up questions work (10+ scenarios)
- [ ] Every normal answer contains citations OR refusal
- [ ] Streaming works end-to-end

---

## Phase 3 — Frontend Chat UI (Week 7–8)

### Objective
Bina UI ChatGPT-like, streaming, citations panel.

### Checklist

#### 3.1 Frontend Setup
- [ ] Create React Vite app in `/frontend`
- [ ] Install + configure Tailwind CSS
- [ ] Create component structure:
  - [ ] `ChatContainer`
  - [ ] `MessageBubble`
  - [ ] `ChatInput`
  - [ ] `SourcesPanel`
  - [ ] `FeedbackButtons`
  - [ ] `SessionManager` utility

#### 3.2 SSE Integration
- [ ] Start/continue SSE stream from `/chat/stream`
- [ ] Append tokens to active assistant bubble
- [ ] On final event:
  - [ ] render sources panel
  - [ ] capture `message_id` for feedback

#### 3.3 Session Handling
- [ ] Generate UUID session_id (if none)
- [ ] Save in localStorage
- [ ] Implement “Clear chat” button (call `/session/clear`)

#### 3.4 Feedback
- [ ] Implement 👍/👎 (call `/feedback`)

### Exit Criteria
- [ ] UI works on desktop + mobile
- [ ] Citations render correctly
- [ ] Clear chat works

---

## Phase 4 — Website Ingestion (Controlled Crawling) (Week 9–10)

### Objective
Index approved website pages safely.

### Checklist
- [ ] Decide ingestion method:
  - [ ] Approved URL list (preferred)
  - [ ] Sitemap parsing (if available)
- [ ] Implement script `backend/scripts/ingest_web.py`
- [ ] Enforce allowlist:
  - [ ] domain allowlist
  - [ ] path allowlist
  - [ ] exclude patterns (login/admin/query params)
- [ ] Fetch pages (rate limited)
- [ ] Extract main content:
  - [ ] remove nav/footer/scripts/styles
- [ ] Clean + chunk text
- [ ] Change detection:
  - [ ] compute content_hash
  - [ ] skip unchanged pages
- [ ] Store into `documents` + `chunks`
- [ ] Add metadata extraction (if possible):
  - [ ] state/project/type tags

### Exit Criteria
- [ ] 30+ approved pages indexed
- [ ] AI answers from web + pdf with citations

---

## Phase 5 — Safety & Public Hardening (Week 11–12)

### Objective
Pastikan sistem selamat untuk public (anti abuse + guardrails).

### Checklist
- [ ] Implement SafetyGuard checks:
  - [ ] prompt injection patterns
  - [ ] sensitive request patterns (internal SOP/HR/commission)
- [ ] Enforce refusal templates (BM/EN)
- [ ] Validate output:
  - [ ] if answer has no citations → refuse/fallback
- [ ] Add rate limiting:
  - [ ] per IP
  - [ ] per session
- [ ] Add basic logging & error handling

### Exit Criteria
- [ ] Injection attempts refused consistently
- [ ] No answers without citations
- [ ] Rate limit triggers under abuse testing

---

## Phase 6 — Evaluation & Tuning (Week 13–14)

### Objective
Improve accuracy and reliability with measurable testing.

### Checklist
- [ ] Create evaluation set (50–100 Q):
  - [ ] BM + English
  - [ ] follow-up sequences
  - [ ] not-found scenarios
- [ ] Measure:
  - [ ] groundedness (citations correct)
  - [ ] refusal correctness
  - [ ] follow-up success rate
  - [ ] latency
- [ ] Tune:
  - [ ] chunk size + overlap
  - [ ] top-k retrieval
  - [ ] metadata filters

### Exit Criteria
- [ ] >= 80% correct with citations
- [ ] follow-up success >= 80%

---

## Phase 7 — Deployment & Website Integration (Week 15–16)

### Objective
Deploy and embed chatbot into the company website.

### Checklist
- [ ] Deploy Postgres + PGVector (managed DB)
- [ ] Deploy backend FastAPI (container or service)
- [ ] Deploy frontend React (static hosting)
- [ ] Setup environment variables on server
- [ ] Setup reverse proxy / HTTPS (if needed)
- [ ] Embed widget / route integration in website
- [ ] Monitor logs + fix issues during stabilization

### Exit Criteria
- [ ] Live chatbot on production website
- [ ] Stable usage and error rate acceptable
- [ ] Monitoring and rollback plan available
