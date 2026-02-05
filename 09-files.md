# 09 — File Changes (File References)

This document tracks **which files are created/modified** in each phase.  
It helps you:

- stay organized while coding
- keep documentation aligned with implementation
- know where each feature lives

Folder structure target:

```text
repo-root/
├─ docs/
├─ backend/
│  ├─ app/
│  ├─ scripts/
│  ├─ tests/
│  └─ migrations/            (optional)
├─ frontend/
│  ├─ src/
│  └─ public/
└─ data/
   ├─ pdfs/
   └─ urls/
```
---

## Phase 0 — Setup & Documentation

### Create / Update
- `README.md`
- `docs/01-requirements.md`
- `docs/02-architecture.md`
- `docs/03-database-schema.md`
- `docs/04-api-specifications.md`
- `docs/05-components.md`
- `docs/06-permissions.md`
- `docs/08-implementation-phases.md`
- `docs/09-file-changes.md`

### Create Folder
- `backend/`
- `frontend/`
- `data/pdfs/`
- `data/urls/`
- `.env.example`
- `.gitignore`

---

## Phase 1 — Database & Core RAG (PDF Only)

### Backend
- `backend/requirements.txt`
- `backend/app/main.py`
- `backend/app/config.py`
- `backend/app/db.py`
- `backend/app/repositories/documents_repo.py`
- `backend/app/repositories/chunks_repo.py`

### Scripts
- `backend/scripts/ingest_pdf.py`
- `backend/scripts/test_retrieval.py`

### Database SQL (optional)
- `backend/sql/schema.sql`

### Data
- `data/pdfs/*.pdf`

---

## Phase 2 — Chat API & Session Memory

### Routes
- `backend/app/routes/chat.py`
- `backend/app/routes/session.py`
- `backend/app/routes/feedback.py`
- `backend/app/routes/health.py`

### Services
- `backend/app/services/session_service.py`
- `backend/app/services/question_rewriter.py`
- `backend/app/services/retrieval_service.py`
- `backend/app/services/prompt_builder.py`
- `backend/app/services/answer_generator.py`
- `backend/app/services/safety_guard.py`

### Repositories
- `backend/app/repositories/chat_repo.py`
- `backend/app/repositories/feedback_repo.py`

---

## Phase 3 — Frontend Chat UI

### Core Files
- `frontend/package.json`
- `frontend/src/main.jsx`
- `frontend/src/App.jsx`

### Components
- `frontend/src/components/ChatContainer.jsx`
- `frontend/src/components/MessageBubble.jsx`
- `frontend/src/components/ChatInput.jsx`
- `frontend/src/components/SourcesPanel.jsx`
- `frontend/src/components/FeedbackButtons.jsx`

### Utilities
- `frontend/src/lib/session.js`
- `frontend/src/lib/sse.js`
- `frontend/src/lib/api.js`

---

## Phase 4 — Website Ingestion

### Data
- `data/urls/approved_urls.txt`
- `data/urls/allowlist.json`

### Scripts & Services
- `backend/scripts/ingest_web.py`
- `backend/app/services/web_crawler.py`
- `backend/app/services/html_extractor.py`
- `backend/app/services/metadata_extractor.py`

---

## Phase 5 — Safety & Hardening

### Middleware
- `backend/app/middleware/rate_limit.py`

### Update
- `backend/app/services/safety_guard.py`
- `backend/app/config.py`

---

## Phase 6 — Testing & Evaluation

### Tests
- `backend/tests/test_retrieval.py`
- `backend/tests/test_followup.py`
- `backend/tests/test_not_found.py`
- `backend/tests/test_injection.py`

### Evaluation Data
- `data/eval/questions_bm.json`
- `data/eval/questions_en.json`

---

## Phase 7 — Deployment (Optional Files)

- `backend/Dockerfile`
- `frontend/Dockerfile`
- `docker-compose.yml`
- `deploy/nginx.conf`
