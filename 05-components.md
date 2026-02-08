# 05 — Components

## 1. Purpose

This document defines the **frontend UI components** used in the Next.js application for the Internal AI Chat Agent.

Components are organized to support authentication, chat interaction, streaming responses, and admin management.

---

## 2. Core Chat Components

### ChatLayout
- Root layout for chat pages
- Enforces authentication guard
- Provides header, content area, and footer

### ChatHeader
- Displays application title
- Shows logged-in user name and role
- Provides logout action

### ChatMessage
- Renders a single message
- Supports `user` and `assistant` message types
- Uses markdown formatting for assistant responses

### ChatMessageList
- Displays ordered list of messages
- Auto-scrolls to latest message
- Handles empty state

### ChatInput
- Text input for user messages
- Submit on Enter
- Disabled while streaming response

### ChatStreamHandler
- Handles Server-Sent Events (SSE)
- Appends streamed tokens to the active assistant message
- Detects stream completion and errors

### ClearChatButton
- Clears current conversation context
- Calls `/chat/clear` API

---

## 3. Conversation Components

### ConversationList
- Displays list of user conversations
- Shows last updated timestamp
- Allows switching between conversations

### ConversationItem
- Represents a single conversation entry
- Highlights active conversation

---

## 4. Feedback Components

### FeedbackButtons
- 👍 and 👎 buttons
- Visible on assistant messages
- Sends feedback to `/feedback` API

---

## 5. Authentication Components

### LoginForm
- Email and password inputs
- Submit handler for `/auth/login`
- Displays authentication errors

### AuthGuard
- Redirects unauthenticated users to `/login`
- Protects chat and admin routes

---

## 6. Admin Components

### AdminLayout
- Layout wrapper for admin pages
- Enforces Admin role access

### ContentPageList
- Displays ingested website pages
- Shows status and last ingested time

### ReindexButton
- Triggers `/content/reindex` API
- Displays progress or status

### ExcludePageAction
- Excludes a page via `/content/exclude`
- Requires confirmation before action

---

## 7. Shared UI Components

### Button
- Standard button component
- Variants: primary, secondary, danger

### Input
- Text input field
- Used across forms

### Modal
- Confirmation and alert dialogs

### LoadingIndicator
- Shows loading or streaming state

---

## 8. Component Principles

- Components are stateless where possible
- Business logic handled in hooks or services
- Authentication and RBAC enforced outside UI
- UI reflects backend state, not authority
