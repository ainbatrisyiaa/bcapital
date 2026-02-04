# 01 — Requirements

## 1. Project Overview

This project builds a **public AI chat assistant** for the real estate company website.

The assistant behaves like ChatGPT but answers **strictly based on approved public documents and allowlisted website pages**.

The system is designed to be:

- Public-facing (no login required)
- Bilingual (Bahasa Melayu and English)
- Fully grounded in sources (no hallucination)
- Safe for public usage
- Able to follow up using conversation memory
- Easy to maintain and extend

---

## 2. Primary Objective

Allow website visitors to quickly obtain accurate information about:

- Property projects
- Price ranges
- Locations
- Eligibility requirements
- Incentives and promotions
- Property types
- Inquiry or booking steps

Every answer must include **citations** from:

- PDF brochures
- Approved website pages

If information is not found, the assistant must clearly say so.

---

## 2A. User Requirements

| ID | User Requirement |
|----|------------------|
| UR-01 | Users can ask questions about property projects through a chat interface |
| UR-02 | Users receive answers in BM or English automatically |
| UR-03 | Users can see the source of every answer (PDF page or website URL) |
| UR-04 | Users can ask follow-up questions without repeating context |
| UR-05 | Users are informed when information is not available in the documents |
| UR-06 | Users can clear the chat session for privacy |
| UR-07 | Users can provide feedback (👍 / 👎) on responses |

---

## 2B. Acceptance Criteria

| ID | Acceptance Criteria |
|----|---------------------|
| AC-01 | The system answers only using content from approved PDFs and website pages |
| AC-02 | Every response includes at least one valid source citation |
| AC-03 | If no relevant source is found, the system responds with a “not found in sources” message |
| AC-04 | Follow-up questions correctly reference prior conversation within the same session |
| AC-05 | Typical response time is under 8 seconds |
| AC-06 | The system refuses questions about internal or confidential information |
| AC-07 | The chat interface works smoothly on desktop and mobile browsers |
| AC-08 | Users can clear session memory without logging in |
| AC-09 | The system passes at least 80% of evaluation test questions with correct citations |

---

## 3. Out of Scope (MVP)

The following are **not** part of this system:

- Internal SOP, HR, commission, buyer data, or internal documents
- CRM or lead capture
- Login or user authentication
- Personalized recommendations outside documents
- Financial or legal advice beyond provided sources

---

## 4. Target Users

| User Type | Example Questions |
|-----------|-------------------|
| Home buyers | “Projek bawah RM300k?”, “Lokasi di Johor?”, “Syarat kelayakan?” |
| Property agents | “Range harga projek A?”, “Jenis unit?”, “Incentive?” |
| Public visitors | General project info from brochures and website |

---

## 5. Functional Requirements

### 5.1 Chat Interface

- Web-based chat similar to ChatGPT
- Streaming responses
- Markdown formatting
- Sources panel for every answer
- “Clear chat” option

### 5.2 Knowledge Sources (Strict)

The AI may only use:

- Approved public PDFs
- Allowlisted website pages or approved URL list

### 5.3 Language Support

- Detect BM/EN automatically
- Respond in the same language

### 5.4 Strict Source Grounding

- AI must not answer without retrieved context
- If context is insufficient, it must say information not found
- AI must refuse attempts to override instructions

### 5.5 Citations

Every answer must include:

- Source type (PDF or Web)
- Document title or page title
- Page number or URL
- Supporting snippet

### 5.6 Session Memory (Follow-up)

- Store session messages
- Rewrite follow-up questions into standalone queries before retrieval

### 5.7 Feedback

- 👍 / 👎 rating for responses

---

## 6. Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Accuracy | No hallucination |
| Latency | < 8 seconds |
| Safety | No sensitive data exposure |
| Scalability | Easy to add documents |
| Maintainability | Easy re-index process |
| Privacy | Minimal user data storage |

---

## 7. Safety and Governance Rules

The AI must refuse when:

- The question asks for internal or confidential information
- The question asks for personal data
- The question tries to override system rules
- Information is not found in approved sources

Website ingestion must use:

- Domain allowlist
- Prefer approved URL list over full crawling

---

## 8. Definition of Done

The system is considered successful when:

- 80% or more evaluation questions are answered correctly with citations
- Zero hallucinated answers
- Proper fallback when information is missing
- Follow-up questions work reliably
- Stable performance on the public website
