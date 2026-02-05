# 05 — Components Overview

This document describes the **main components** of the AI Chat Agent system and the role of each component.

The system is divided into:

- Frontend Components (React)
- Backend Components (FastAPI)
- RAG Engine Components
- Ingestion Components
- Safety & Control Components

---

## 1. Frontend Components (React)

These components handle the user interface and interaction.

### 1.1 `ChatContainer`

**Purpose**: Main wrapper for the chat system.

Responsibilities:
- Holds chat messages
- Manages SSE connection
- Controls session lifecycle
- Renders chat window layout

---

### 1.2 `MessageBubble`

**Purpose**: Displays each message.

Responsibilities:
- Different style for user vs assistant
- Render markdown
- Display streaming text
- Attach sources panel for assistant replies

---

### 1.3 `ChatInput`

**Purpose**: User input box.

Responsibilities:
- Capture user message
- Trigger API call
- Disable input while streaming
- Handle Enter key submission

---

### 1.4 `SourcesPanel`

**Purpose**: Show citations for assistant answers.

Responsibilities:
- List PDF or Web sources
- Show page number or URL
- Display snippet preview

---

### 1.5 `FeedbackButtons`

**Purpose**: Collect user feedback.

Responsibilities:
- Send 👍 / 👎 to backend
- Attach to specific `message_id`

---

### 1.6 `SessionManager` (utility)

**Purpose**: Manage session ID.

Responsibilities:
- Generate UUID
- Store in localStorage
- Clear session when requested

---

## 2. Backend Components (FastAPI)

These components expose API endpoints and manage chat flow.

### 2.1 Routes Layer

| File | Responsibility |
|-----|------------------|
| `chat.py` | `/chat/stream` endpoint |
| `session.py` | Clear session memory |
| `feedback.py` | Store feedback |
| `health.py` | Health check |

---

### 2.2 Services Layer

Core AI logic lives here.

| File | Responsibility |
|-----|------------------|
| `retrieval_service.py` | Query vector DB for relevant chunks |
| `question_rewriter.py` | Rewrite follow-up into standalone question |
| `prompt_builder.py` | Build strict RAG prompt |
| `answer_generator.py` | Call LLM and stream response |
| `session_service.py` | Manage chat memory |
| `safety_guard.py` | Detect injection & sensitive queries |

---

### 2.3 Repositories Layer

Database interaction layer.

| File | Responsibility |
|-----|------------------|
| `documents_repo.py` | Access documents table |
| `chunks_repo.py` | Vector similarity search |
| `chat_repo.py` | Store chat messages |
| `feedback_repo.py` | Store feedback |

---

## 3. RAG Engine Components

This is the "brain" of the system.

### Flow

1. Receive user message
2. Rewrite question if follow-up
3. Retrieve top-k chunks from vector DB
4. Build strict prompt with sources
5. Generate answer from LLM
6. Return answer + citations

---

## 4. Ingestion Components

Responsible for populating the knowledge base.

### 4.1 `ingest_pdf.py`

Responsibilities:
- Read PDFs from `/data/pdfs`
- Clean text
- Chunk content
- Generate embeddings
- Store into DB

---

### 4.2 `ingest_web.py`

Responsibilities:
- Read approved URL list
- Crawl pages safely (allowlist)
- Extract main content
- Detect content changes
- Chunk and embed

---

### 4.3 Supporting Services

| File | Responsibility |
|-----|------------------|
| `web_crawler.py` | Safe page fetching |
| `html_extractor.py` | Remove header/footer/scripts |
| `metadata_extractor.py` | Extract project/state/type tags |

---

## 5. Safety & Control Components

Ensures system is safe for public use.

### 5.1 `safety_guard.py`

Detects:
- Prompt injection attempts
- Requests for sensitive/internal info
- Attempts to override instructions

---

### 5.2 `rate_limit.py`

Prevents abuse:
- Limit requests per IP/session

---

## 6. How Components Work Together

`User → React UI → FastAPI Route → Services → Retrieval → LLM → Stream back → UI`

Each layer has a single responsibility, making the system:

- Easy to debug
- Easy to maintain
- Easy to scale
