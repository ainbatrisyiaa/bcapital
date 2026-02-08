# 01 — Requirements

## 1. Project Overview

This project builds an **internal AI chat assistant** for a real estate company.

The assistant behaves like ChatGPT but answers **strictly based on approved and crawled website content**, with enforced role-based access control (RBAC) for internal users.

The system is designed to be:

- Private (login required)
- Role-based (Agent and Admin)
- Bilingual (Bahasa Melayu and English)
- Fully grounded in crawled website content (no hallucination)
- Safe for internal company usage
- Able to handle follow-up questions using session memory
- Easy to maintain and extend by admins

---

## 2. Primary Objective

Enable registered users (Agents and Admins) to quickly obtain accurate and consistent information related to:

- Property projects
- Price ranges
- Locations
- Eligibility requirements
- Incentives and promotions
- Property types
- Inquiry and booking steps
- Company-approved real estate information

All responses must be generated **only from crawled and approved website content**.

If information is not found in the crawled content, the assistant must clearly state so.

---

## 2A. User Requirements

| ID | User Requirement |
|----|------------------|
| UR-01 | Users must log in to access the AI assistant |
| UR-02 | Users can ask questions via a chat interface |
| UR-03 | Users receive responses in BM or English automatically |
| UR-04 | Users can ask follow-up questions within the same session |
| UR-05 | Users are informed when information is not found in the crawled content |
| UR-06 | Users can clear chat session memory |
| UR-07 | Users can submit feedback (👍 / 👎) |
| UR-08 | Admin users can manage website crawling settings |
| UR-09 | Agents can only access information permitted to their role |

---

## 2B. Acceptance Criteria

| ID | Acceptance Criteria |
|----|---------------------|
| AC-01 | Login is mandatory before accessing the chat |
| AC-02 | RBAC is enforced at the backend level |
| AC-03 | The assistant answers only using crawled website content |
| AC-04 | If no relevant content is found, the assistant responds with a “not found in content” message |
| AC-05 | Follow-up questions correctly reference prior session context |
| AC-06 | Average response time is under 8 seconds |
| AC-07 | Unauthorized or confidential questions are refused |
| AC-08 | The UI works smoothly on desktop and mobile browsers |
| AC-09 | Crawled content updates are reflected after re-indexing |
| AC-10 | At least 80% of evaluation questions are answered correctly |

---

## 3. Out of Scope (MVP)

The following are not included in this system:

- Public or guest access
- PDF ingestion or document upload
- Citation display
- CRM or lead capture
- Personalized recommendations outside crawled content
- Automatic crawling of non-approved websites

---

## 4. Target Users

| User Type | Example Questions |
|-----------|-------------------|
| Agent | "Range harga projek A?", "Jenis unit?", "Steps booking?" |
| Admin | "Trigger website re-crawl", "Update allowed URLs", "Check content coverage" |

---

## 5. Functional Requirements

### 5.1 Authentication and Roles

- Login required
- Supported roles:
  - Agent
  - Admin
- Role determines:
  - Accessible content
  - Administrative permissions

---

### 5.2 Chat Interface

- Web-based chat interface similar to ChatGPT
- Streaming responses
- Markdown formatting support
- Clear chat option

---

### 5.3 Knowledge Source (Website Crawling Only)

The assistant may only use:

- Crawled content from approved website domains or URL lists

Rules:
- Crawling is restricted to allowlisted domains
- Admin controls which URLs are included
- Crawled data is stored and indexed for retrieval

---

### 5.4 Language Support

- Automatic detection of BM or English
- Responses in the same language as the query

---

### 5.5 Strict Content Grounding

- The assistant must not respond without retrieved website context
- If context is insufficient, it must state information is not available
- Attempts to override system instructions must be refused

---

### 5.6 Session Memory

- Conversation memory maintained per session
- Follow-up questions rewritten into standalone queries before retrieval

---

### 5.7 Feedback

- Users can rate responses using 👍 / 👎
- Feedback is stored for evaluation and improvement

---

### 5.8 Admin Crawling Management

- Configure allowed domains or URL lists
- Trigger manual re-crawling
- View crawl status and last updated timestamp

---

## 6. Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Accuracy | Zero hallucination |
| Latency | < 8 seconds |
| Security | Role-based access enforced |
| Scalability | Easy website expansion |
| Maintainability | Simple re-crawl workflow |
| Privacy | Minimal user data storage |

---

## 7. Safety and Governance Rules

The assistant must refuse requests when:

- Information is confidential or unauthorized
- Personal data is requested
- System rules are being bypassed
- No crawled content is available

Crawling rules:

- Only allowlisted domains or approved URLs
- No external or third-party sites
- Backend enforces role-based filtering during retrieval

---

## 8. Definition of Done

The system is considered complete when:

- At least 80% of evaluation questions are answered correctly
- No hallucinated responses are observed
- Fallback responses are consistent
- Follow-up questions work reliably
- RBAC restrictions are verified
- System performance is stable for internal usage
