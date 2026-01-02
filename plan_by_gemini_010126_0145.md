# Project Plan: bjkai_notes
**Date:** January 1, 2026
**Based on:** Docmost Architecture & User Requirements
**Evaluated by:** Gemini CLI

## 1. Executive Summary & Evaluation of NotebookLM Advice

The advice provided by NotebookLM is **architecturally sound** but **operationally optimistic**.
*   **Verdict:** **Agree on "Fork & Modify".** Rebuilding a real-time collaborative editor (Tiptap + Y.js + Hocuspocus) from scratch is a massive undertaking (estimated 6-12 months for one developer). Forking Docmost gives you a mature foundation immediately.
*   **Correction on "Agents":** NotebookLM suggests "2 agents." In modern LLM application engineering, you don't need distinct "agents" so much as **specialized pipelines**. You will need at least **three** distinct AI workflows, not just two (detailed in Section 3).
*   **Correction on "Stubbing Email":** This is critical. You cannot just "log" it; you must ensure the *frontend* doesn't expect an OTP (One-Time Password) flow if you strip the email service, or you must capture that OTP from the server logs during development.

## 2. Strategic Approach: The "Clean Fork"

### 2.1 License & Repository Setup
*   **Source:** Fork the `docmost/docmost` repository.
*   **License:** The core is AGPL-3.0. You **must** keep this license if you distribute the app. It ensures your project remains open source.
*   **Sanitization:**
    1.  **Delete** `packages/ee` (Enterprise Edition).
    2.  **Delete** `apps/client/src/ee` and `apps/server/src/ee`.
    3.  **Search & Destroy:** Grep for imports starting with `@docmost/ee` and remove/refactor them. This is the hardest part of the fork—untangling the "hooks" that check for license keys.

### 2.2 Auth & Email (The "No-SaaS" Approach)
Your goal is "no external email setups."
*   **Authentication:** Stick to standard `Email/Password` auth provided by the core.
*   **The "Log Driver" for Email:**
    *   In `apps/server`, look for the `MailService`.
    *   Create a `DevMailAdapter` that simply does `console.log(emailContent)`.
    *   **Why this matters:** When you create a user or reset a password, the system generates a token. You need to see this token to log in. By dumping it to the Docker logs, you can copy-paste it without needing SendGrid/SMTP.

## 3. AI Architecture: "Bring Your Own Key" (BYOK)

You requested "any/desired API provider." This requires a **Gateway Pattern**. Do not hardcode OpenAI.

### 3.1 The Three AI Pipelines (Agents)
You don't need "autonomous agents" wandering your codebase. You need three specific functional pipelines:

1.  **The "Editor Co-pilot" (Fast & Smart)**
    *   **Trigger:** User highlights text in the editor or types `/ai`.
    *   **Task:** "Fix grammar," "Shorten," "Change Tone."
    *   **Tech:** Tiptap AI extension.
    *   **Model:** Needs a low-latency model (e.g., `gpt-4o-mini`, `claude-3-haiku`, or local `llama3`).

2.  **The "Librarian" (RAG - Retrieval Augmented Generation)**
    *   **Trigger:** Background process (Cron job or Event-triggered).
    *   **Task:** Whenever a document is saved, chunk the text, generate vector embeddings, and save them to `pgvector` (PostgreSQL extension).
    *   **Why:** This enables the "Chat with my notes" feature.
    *   **Model:** Embedding Model (e.g., `text-embedding-3-small` or local `nomic-embed-text`).

3.  **The "Oracle" (Q&A Chat)**
    *   **Trigger:** User opens the "Ask AI" sidebar.
    *   **Task:** Takes user query -> Searches Vector DB (The Librarian) -> Sends context + query to LLM -> Streams answer.
    *   **Model:** High-reasoning model (e.g., `gpt-4o`, `claude-3.5-sonnet`).

### 3.2 The "Universal Gateway"
Instead of hardcoding APIs, use the Vercel AI SDK (`ai` npm package) in `apps/server`. It abstracts the provider.
*   **Config:** Allow users to set `AI_PROVIDER` (openai, anthropic, ollama) and `AI_API_KEY` in the `.env` file.

## 4. Development Environment & Tooling

### 4.1 Recommended IDE
*   **Top Pick:** **Cursor** (Fork of VS Code).
    *   **Why:** Since you are building an AI-heavy app, Cursor's built-in "Composer" feature allows you to highlight code and say "Refactor this to use the OpenAI SDK instead of the hardcoded logic," and it will do it across multiple files. It is superior to standard VS Code + Copilot for heavy refactoring tasks (like stripping the `ee` folder).
*   **Alternative:** **VS Code** with "GitHub Copilot" extension.

### 4.2 The "Agent" Team Size
You asked "how many different ai agents do i need."
*   **Runtime (In the app):** 3 (Editor, Librarian, Oracle).
*   **Development (To build it):** 1 (You + Cursor/Copilot). You do not need a complex multi-agent coding framework (like CrewAI) to build this. Standard tools are sufficient and less error-prone.

## 5. Step-by-Step Implementation Plan

### Phase 1: The Surgery (Day 1-2)
1.  `git clone` the Docmost repo.
2.  Run `npm install` (or `pnpm install`) to verify the base state.
3.  **Delete** the `ee` folders.
4.  Run the build command (`pnpm build`). It **will fail**.
5.  Use Cursor/VS Code to fix every compilation error caused by missing `ee` modules. Replace them with "No-op" (empty) functions or standard open-source equivalents.

### Phase 2: The Bypass (Day 3)
1.  Locate `mail.service.ts` in the server.
2.  Implement the `console.log` mail driver.
3.  Boot the app using `docker-compose up`.
4.  Register a user. Check the logs for the verification link. Click it. Confirm you can log in without an email server.

### Phase 3: The Brain Transplant (Day 4-7)
1.  Install Vercel AI SDK: `pnpm add ai @ai-sdk/openai @ai-sdk/anthropic` in `apps/server`.
2.  **Vector DB:** Enable `pgvector` in the `docker-compose.yml` Postgres service.
3.  **The Librarian:** Write a subscriber to the `DocumentUpdate` event. When a doc updates -> generate embedding -> save to Postgres.
4.  **The Oracle:** Create a new API route `/api/ai/chat` that queries Postgres and sends the prompt to the LLM.

### Phase 4: Polish & UI (Day 8+)
1.  Hook up the frontend "Ask AI" button to your new API route.
2.  Test with different providers (OpenAI, Anthropic, Ollama for local).

## 6. Summary of Component Approach

| Component | Docmost Base | **bjkai_notes** (Your App) |
| :--- | :--- | :--- |
| **License** | AGPL + Enterprise | **AGPL 3.0 (Pure)** |
| **Auth** | Complex (SAML/SSO/Email) | **Simple (Email/Pass + Log Driver)** |
| **Database** | Postgres | **Postgres + pgvector** (for AI memory) |
| **AI Logic** | Hardcoded / Enterprise | **Vercel AI SDK** (Provider Agnostic) |
| **Email** | SMTP / Postmark | **Console Logger** (Zero Config) |
| **Hosting** | Docker | **Docker** (Self-contained) |

This plan gives you a professional-grade application without the "Enterprise" bloat, fully controlled by you.
