Plan for bjkai_notes (fully open source, Docmost-class features, AI enabled, no SSO/email)

Goals and constraints (from request)
- Match Docmost feature surface: realtime collaborative editor, docs/wiki UX, search, import/export, comments, diagrams, tables, media, permissions, etc.
- Remove all enterprise-licensed features and any SAML/OIDC/LDAP SSO.
- Avoid external email delivery setup (no SMTP/Postmark); keep minimal auth flows.
- Keep AI features, but make provider/model pluggable (any API provider).
- Ship as fully open source (single license; no “official” or dual-licensed editions).

Key decision: build approach
Option A: Fork Docmost OSS core
- Pros: fastest to parity; reuse editor + collaboration + existing UI/UX; lower risk.
- Cons: must audit and remove EE references and any mixed licensing concerns.
Option B: Rebuild from scratch using same stack
- Pros: clean IP separation; full control.
- Cons: much longer; more bugs; parity takes time.
Recommendation: Fork Docmost OSS core, then refactor for single-license + open AI features.

Scope alignment and cleanup
- Remove Enterprise Edition packages and UI entry points.
- Remove SAML/OIDC/LDAP and any EE-specific API routes.
- Remove/disable email delivery drivers; replace with log-only or in-app token flows.
- Ensure all “EE” references and flags are removed from UI/UX copy and settings.
- Keep diagrams, comments, page history, sharing, and editor extensions.

AI design (provider-agnostic)
Core capabilities to implement
1) RAG/semantic search over workspace content
2) Writing assistant (summarize, rewrite, expand, extract)
3) Optional: Q&A agent for “Ask AI” across notes
Provider abstraction
- Define a single AI provider interface:
  - embeddings.create(texts[])
  - completions.create(prompt, tools?, context?)
  - chat.create(messages, tools?, context?)
- Implement adapters for:
  - OpenAI-compatible APIs (OpenAI, Together, Azure OpenAI, etc.)
  - Local models via Ollama (optional)
RAG stack
- pgvector + chunking + background indexing
- ingestion hooks: on page save, enqueue chunking and embedding
- retrieval: top-k vector search with permission filters

How many “AI agents” do you need?
Minimal, production-worthy set: 2 agents
1) Content Assistant Agent
   - rewrite/summarize/expand/format
   - deterministic prompts + guardrails
2) Search/Q&A Agent (RAG)
   - answers only from indexed content
   - citations to page titles/IDs
Optional later: 3rd “Automation Agent”
- handles workflows (tagging, auto-summary on publish, etc.)
Recommendation: start with 2, expand if the UX demands it.

Auth and email without external providers
- Use password auth with email as identifier but no outbound email delivery.
- Add in-app “magic link” alternatives only if you can deliver links; otherwise:
  - Provide admin-generated password reset tokens.
  - Provide “copy reset token” in admin UI, or CLI-only reset.
- Audit all flows that assume SMTP; swap to log/mock driver.

Monorepo and tooling (practical choices)
IDE: VS Code
- Strong TypeScript, React, NestJS support
- Good debugging for Node + Vite
AI coding tool: GitHub Copilot (or equivalent)
- Use for boilerplate; keep AI changes reviewed
Monorepo:
- pnpm + Nx (matches docs and scalable task graph)
Frontend: React + Vite + Mantine UI
Backend: NestJS + Fastify + Kysely
Realtime: Hocuspocus + Yjs
Jobs: BullMQ + Redis
DB: Postgres + pgvector

Implementation phases (recommended)
Phase 0: Legal + repo hygiene
- Confirm license compatibility (AGPLv3 only)
- Remove ee/ directories and any proprietary code pointers
- Update README, docs, and package metadata to single-license

Phase 1: Core functionality baseline
- Get client/server running
- Verify editor, collaboration, spaces/pages, comments, attachments
- Confirm Docker Compose and local dev flow

Phase 2: Remove SSO + email delivery
- Delete SAML/OIDC/LDAP strategies + UI entries
- Replace mail driver with log/mock
- Provide admin reset token flow

Phase 3: AI foundation
- Add AI provider interface + adapters
- Add embeddings + chunking + pgvector schema
- Add background jobs for indexing
- Add RAG API + basic UI

Phase 4: Polish and docs
- Update docs to reflect open-source single license
- Document AI provider configuration
- Add “no-email” auth guidance

Risks and mitigations
- License contamination: run a quick audit of imported EE code or assets.
- Feature drift: ensure search stack (Algolia/Typesense) references are cleaned up.
- AI costs: add rate limits, caching, and provider quotas.
- Security: permission-aware retrieval and prompt injection mitigations.

What I need from you to proceed
- Preferred license (AGPLv3 ok?)
- Which AI providers to support first (OpenAI-compatible? Ollama?)
- Whether “no email” means no email at all, or “no external SMTP”
- Target deployment (self-hosted only? cloud?)

If you want, I can now:
1) Draft a concrete repo task list + milestones.
2) Build a minimal fork plan with file-by-file removals.
3) Produce a first-pass AI provider interface design.
