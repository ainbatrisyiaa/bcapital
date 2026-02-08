# AI Internal Chat Agent — Real Estate Knowledge Assistant

This project implements an **internal ChatGPT-like AI assistant** for a real estate company.

The assistant is accessible **only to registered Agents and Admins** via login and answers questions **strictly based on approved and ingested company website content**.

The system is designed to be secure, reliable, and fully grounded in company-approved information, with **no hallucination** and **no reliance on external knowledge**.

---

## 🚀 Key Features

- Chat interface similar to ChatGPT
- Login required (Agent & Admin roles)
- Answers strictly from ingested website content
- Supports Bahasa Melayu and English automatically
- Session-based conversation memory for follow-up questions
- Streaming responses for better user experience
- Role-based access control (RBAC)
- Safe for internal company usage
- Easy content ingestion and re-indexing

---

## 🧠 Core Concept: Grounded AI with Controlled Knowledge

The AI does **not** rely on its pre-trained knowledge about the company.

Instead, it:
1. Retrieves relevant information from ingested website content
2. Builds context strictly from retrieved content
3. Generates answers only from that context
4. Refuses to answer when information is not available

This approach ensures accurate, consistent, and policy-compliant responses.

---

## 🏗️ Architecture Overview

### Frontend
- Next.js (App Router)
- Tailwind CSS
- Secure authentication (Agent / Admin)
- Streaming chat interface
- Markdown-rendered responses

### Backend
- FastAPI (Python)
- Secure API endpoints with token validation
- Custom retrieval and answer orchestration
- Session memory and follow-up question rewriting

### Data Layer
- PostgreSQL (users, roles, sessions, logs, feedback)
- Vector database (pgvector) for indexed website content

---

## 🔄 How It Works (High-Level Flow)

1. Approved website content is ingested and indexed
2. Content is cleaned, chunked, embedded, and stored
3. A logged-in user sends a message through the chat interface
4. The backend rewrites follow-up questions into standalone queries
5. Relevant content is retrieved based on the user role
6. The AI generates an answer strictly from retrieved content
7. The response is streamed back to the user interface

If no relevant content is found, the assistant clearly states that the information is not available.

---

## 🔐 Security and Access Control

- Login is mandatory for all users
- Role-based access control (Agent vs Admin)
- Only approved website content is ingested
- Prompt injection defenses enforced at the backend
- Minimal storage of user data
- Session memory can be cleared by the user

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

**Documentation → Architecture → API Specification → Implementation → Testing → Deployment**

No implementation should begin before Tier 2 documentation is finalized.

---

## 🌱 Future Expansion

- Analytics dashboard for Admins
  - Most frequently asked questions
  - Question trends over time
  - Most active users (by role)
  - Unanswered or low-confidence questions
  - Feedback analysis (👍 / 👎)
- Content gap detection (identify missing or outdated website content)
- Performance monitoring (response time, retrieval success rate)
- Advanced role-based usage insights
