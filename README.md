# AI Public Chat Agent — Real Estate Knowledge Assistant

This project implements a ChatGPT-like AI assistant for the company website that answers public questions using **only approved PDFs and allowlisted website pages**.

The assistant is bilingual (Bahasa Melayu & English), provides **source citations** for every answer, and supports **follow-up questions** using session-based conversation memory — while remaining strictly grounded in documents (no hallucination).

---

## 🚀 Key Features

- Chat interface similar to ChatGPT
- Answers strictly from approved sources (PDF + website)
- Source citations (PDF page / URL + snippet)
- Supports BM and English automatically
- Session memory for follow-up questions (no login required)
- Safe for public usage (no internal/sensitive data exposure)
- Easy document ingestion and re-indexing

---

## 🧠 Core Concept: RAG (Retrieval-Augmented Generation)

The AI does not rely on pre-trained knowledge about the company.

Instead, it:
1. Retrieves relevant chunks from documents
2. Builds context from those chunks
3. Generates answers strictly from that context
4. Shows sources for transparency

If information is not found, the assistant will say so.

---

## 🏗️ Architecture Overview

**Frontend**
- React (Vite) + Tailwind
- Streaming chat UI
- Markdown rendering + Sources panel

**Backend**
- FastAPI (Python)
- Custom RAG orchestration (no LangChain required)
- Session memory + question rewriting for follow-ups

**Database**
- PostgreSQL (sessions, logs, feedback, document registry)
- PGVector (embeddings + metadata filtering)

---

## 🔄 How It Works (High Level Flow)

1. PDFs and approved web pages are ingested
2. Text is cleaned, chunked, embedded, and stored in PGVector
3. User sends a message with a `session_id`
4. Backend rewrites follow-up questions into standalone queries
5. Relevant chunks are retrieved
6. LLM answers using only retrieved content
7. Answer and sources are streamed to the UI

---

## 🔐 Safety by Design

- Only approved public documents are ingested
- Website crawling uses strict allowlist or approved URL list
- Prompt injection defenses
- Refusal when information is not found in sources
- No storage of sensitive user data
- “Clear chat” option for privacy

---

## 📁 Documentation Structure

| Tier | Description |
|------|-------------|
| Tier 1 | Getting started and overview |
| Tier 2 | Core system design and specifications |
| Tier 3 | Deployment, security, and operations |

Start with: `00-getting-started.md`

---

## 🧭 Development Philosophy

**Documentation → Architecture → API Spec → Implementation → Testing → Deployment**

No coding should start before Tier 2 documents are completed.

---

## 🌱 Future Expansion

- WhatsApp chatbot integration
- Internal staff assistant with login + internal knowledge base
- CRM / lead capture integration
