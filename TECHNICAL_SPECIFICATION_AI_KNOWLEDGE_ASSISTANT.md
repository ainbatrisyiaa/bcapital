# Technical Specification Document: AI Knowledge Assistant

**Version:** 1.0  
**Date:** [Insert Date]  
**Prepared By:** [Your Name]  
**Status:** Draft Proposal

---

## 1. Executive Summary

This project aims to develop a dedicated **AI Chat Agent** for internal use at **[Company Name]**.

The system will act as an intelligent assistant for real estate agents and staff to quickly retrieve accurate information about projects, property listings, and company documentation through a chat interface.

**Primary focus:**

- Information accuracy
- Source citation (traceable answers)
- Reducing manual time spent searching for information

---

## 2. Objectives & Scope

### 2.1 Main Objectives

**Centralized Knowledge**  
All project data (from website & PDFs) can be accessed through a single entry point (chat interface).

**Instant Retrieval**  
Reduce the time spent searching for specific information (price, unit size, sales package) from minutes to seconds.

**Data-Driven Responses**  
Ensure every AI answer is supported by official company data, not fabricated content (no hallucination).

---

### 2.2 Phase 1 Scope (MVP)

- Web-based chat interface
- Intelligent search system (RAG) using company website & selected PDFs
- Persistent chat history
- Source citation links for every answer

---

## 3. Technical Architecture

### 3.1 Tech Stack

| Component | Technology | Description |
|----------|------------|-------------|
| Frontend | React.js (Vite) + Tailwind CSS | Fast, responsive UI |
| Backend | Python (FastAPI) | High performance, AI-ready |
| LLM Engine | OpenAI API (GPT-4o) | Language intelligence |
| Orchestrator | LangChain | Manage data flow between DB and LLM |
| Vector DB | ChromaDB / Pinecone | Semantic search storage |
| App DB | PostgreSQL | Store users, sessions, chat history |

---

### 3.2 Data Flow (RAG Pipeline)

#### Ingestion

- Automatic scraper reads the company website
- Text is extracted, cleaned, and chunked
- Text converted into embeddings and stored with metadata (source URL)

#### Retrieval

- User asks question → Backend converts question into embedding
- Vector DB retrieves top 3–5 most relevant chunks

#### Generation

- Relevant info + question sent to OpenAI
- System prompt:  
  > "Answer only based on this information and include the source."

#### Response

- Answer + source links returned to React frontend

---

## 4. Functional Requirements

### 4.1 Chat Module (Frontend)

- Chat interface similar to ChatGPT
- Streaming response (real-time typing effect)
- Citation rendering with clickable source links
- Markdown support (bullets, bold text, tables)

---

### 4.2 Chat History Management

- Sidebar navigation for previous chat sessions
- Create/Delete chat sessions
- Persistent history (stored in PostgreSQL)

---

### 4.3 Backend & Admin

- Knowledge refresh script (e.g., weekly) to keep data updated

---

## 5. Security & Data Privacy

### 5.1 AI Data Policy

Using OpenAI API (Paid/Enterprise Tier) ensures:

- **Zero Data Retention** — data not used to train public models
- **Encryption** during transfer (TLS 1.2+) and at rest

---

### 5.2 User Access

- MVP: Restricted access via office IP/VPN or simple login
- Future: Integration with Google Workspace / Microsoft 365 SSO

---

## 6. Implementation Roadmap

### Phase 1 — Proof of Concept (Week 1–2)

- Setup Python backend
- Scrape data from 1 project URL
- Test Q&A via terminal (no UI)

---

### Phase 2 — Backend Development (Week 3–4)

- Setup Vector Database (ChromaDB)
- Build ingestion pipeline (Website → Vector DB)
- Setup FastAPI endpoints

---

### Phase 3 — Frontend Development (Week 5–6)

- Setup React & Tailwind
- Integrate API with chat UI
- Implement chat history feature

---

### Phase 4 — Testing & Deployment (Week 7–8)

- User Acceptance Test (UAT) with selected agents
- Deploy to internal server or cloud (AWS / DigitalOcean)

---

## 7. Success Criteria

- 80–90% accuracy in answering project-related questions
- Response time under 3 seconds
- Reduced time spent by agents searching for project info
- Reduced repetitive enquiries to staff

---

## 8. Risks & Mitigation

| Risk | Mitigation |
|------|------------|
| Inaccurate project data | Validate KB with business team |
| Irrelevant AI response | Enforce RAG + citation |
| Outdated knowledge | Weekly refresh script |
| System downtime | Reliable cloud hosting |

---

## 9. Maintenance Plan

- Project team updates project files in GitHub
- Scheduled knowledge refresh
- Monthly log review for improvements

---

## 10. Estimated Running Cost (High Level)

| Item | Estimated Monthly Cost |
|------|------------------------|
| OpenAI API Usage | RM150 – RM500 |
| Cloud Server | RM80 – RM150 |
| Others | RM0 |

---

## 11. Future Expansion

- WhatsApp chatbot integration
- Internal staff AI assistant
- CRM / lead system integration
