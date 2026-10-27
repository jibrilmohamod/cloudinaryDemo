# Clearpath 2.0 — Product and Implementation Plan

**Status:** Approved direction for planning; no production migration or replacement has occurred.
**Product:** Private, ADHD-friendly personal operating system for work, university and personal life.
**Source repository:** `jibrilmohamod/cloudinaryDemo`
**Target deployment:** Netlify (verify Git integration and live deployment before changing production).
**Initial audience:** One primary user. Design schema for additional accounts and workspaces, but do not build collaboration first.

## 1. Product outcome

Replace the specific things the owner uses Notion for, while cutting the overhead of deciding what to do next.

The loop is: **Capture → Clarify → Choose → Focus → Follow up → Review**.

The application must be capable of representing tasks, projects, documents and evidence, meetings, stakeholder records, email-derived actions, learning and personal routines. It is not a generic public Notion clone and must not add a form-builder, wiki marketplace or gamified social system before delivering reliable daily value.

### Success criteria

- Create a task from any screen with one shortcut/button and a text field. All extra fields are optional.
- Launch to a useful **Today** view with at most three user-controlled priorities and a realistic schedule.
- Quickly answer what is **due**, **scheduled**, **waiting**, **blocked**, and **unprocessed**; never confuse these statuses.
- Sync between desktop and phone and retain an exportable copy of everything.
- Open any project and see its linked tasks, meetings, documents, evidence requests, communications and stakeholders.
- Every AI-derived task or deadline retains a traceable source and review status.
- No external communication is sent, deleted, archived or marked read without explicit permission.
- Verify migrated Notion record counts, relationships, attachment references and representative samples before switching away from Notion.
- The live app must pass authenticated read/write and isolation tests, mobile tests, keyboard tests and backup restore tests.

## 2. Information architecture and navigation

### Main navigation (keep short)

1. **Today** — at most 3 chosen priorities, due items, upcoming meetings, next small action, focus panel, and overdue triage with a non-punitive tone.
2. **Inbox** — brain dump for unprocessed tasks, notes, links and captured items; classify later.
3. **My Work** — projects, tasks (list/board/calendar), meetings, evidence, documents and stakeholders grouped by context.
4. **Knowledge** — searchable notes and pages; attachments and versions; backlinks to their source projects.
5. **Learning** — university assignments, study sessions, skills and practice logs.
6. **Focus** — adjustable focus intervals, interruption capture, resumable sessions and lightweight review.
7. **Assistant** — natural-language command/search over workspace records; clearly displayed proposed actions.
8. **Settings / Integrations** — account, exports, Notion migration, Gmail/Calendar, AI costs, security, MCP.

Mobile uses bottom navigation for Today, Inbox, Focus, Projects and Search; advanced modules live behind context navigation. No eight-column desktop tables on mobile.

### Primary UX principles

- **Fast capture:** Ctrl/Cmd+K command palette and a fixed mobile quick-capture control.
- **Progressive disclosure:** show title first; reveal dates, project, priority, effort and metadata only when asked.
- **Next-action clarity:** an item can have a one-line next action and an optional “start for 5 minutes” option.
- **Scheduling is distinct from deadlines:** a planned work session must never turn into a claimed external due date.
- **Low-energy mode:** optional filters for 5/15/30 minutes, effort and context; no diagnosis or passive monitoring.
- **No punitive streaks:** missed days and overdue items produce a recovery flow, not shame or confetti.
- **Visual discipline:** responsive, dark/light modes, restrained colour, good contrast, accessible focus rings, reduced-motion support.
- **User agency:** AI can suggest priorities but never silently override user choices.

## 3. Domain data model

**Multi-workspace base:** `profiles`, `workspaces`, `workspace_memberships`, `integrations`, `audit_events`, `activity_events`, `external_id_map`.

**Daily execution:** `projects`, `tasks`, `task_dependencies`, `focus_sessions`, `routines`, `recurrences`.

**Knowledge and provenance:** `pages`, `documents`, `document_versions`, `attachments`, `entity_links`, `source_records`.

**Work operations:** `meetings`, `meeting_participants`, `decisions`, `commitments`, `stakeholders`, `evidence_requests`, `email_threads`, `calendar_events`.

**Learning:** `courses`, `assignments`, `skills`, `practice_sessions`.

**AI & automation:** `ai_suggestions`, `approval_requests`, `sync_jobs`, `sync_cursors`, `automation_rules`, `automation_runs`.

All top-level records are owned by a workspace/user. Use immutable IDs, created/updated timestamps, soft delete and an import source mapping. Do not substitute arbitrary unvalidated JSON for core relations.

### Essential task fields

`id, workspace_id, title, description, project_id, area, status, priority, task_type, due_at, deadline_confidence, planned_for, estimated_minutes, effort, waiting_on, recurrence, source_record_id, source_url, created_by, created_at, updated_at, completed_at`.

Task statuses: **Inbox, Not started, Up next, In progress, Waiting, Blocked, Done, Cancelled**.

Priority: **Critical, High, Normal, Low**.

Area: **Work, School, Personal, Admin**.

`deadline_confidence`: **Confirmed, Implied, No deadline**. An inferred date is not promoted to confirmed without review.

### Project record

Area, type, client, countries, stage, phase, health, priority, next action, milestone, final deadline, relations. A project dashboard should derive counts from actual linked records rather than manually duplicated totals.

### Evidence integrity

An evidence request must independently track **requested / partially received / received / under review / verified / unavailable**, plus verification sufficiency. A received document is not automatically verified. Preserve source links and chronology.

## 4. Technical architecture

- **Web framework:** Nuxt 4 + Vue 3 + TypeScript, app routes and reusable typed components.
- **UI:** Nuxt UI/Tailwind, accessible component tokens and responsive layouts. Avoid prematurely designing a bespoke design system.
- **Hosting:** existing GitHub repository deployed to Netlify through Git integration, with preview deployments, environment settings and build checks.
- **Data:** Supabase Postgres with migrations, indexes and full-text search; use Storage for user-authorised files.
- **Authentication:** Supabase Auth. RLS on every exposed user/workspace table. No secret/service-role key in the browser.
- **Application logic:** Postgres RPC for data-shaped operations; server-side Nuxt/Netlify or Supabase Edge Functions for integrations and AI calls; pin dependencies.
- **Client state:** composables + focused store; optimistic updates with error recovery. Add an IndexedDB offline queue after authenticated sync is stable.
- **Testing:** unit tests (Vitest), browser E2E (Playwright), integration tests against test DB, mobile and accessibility smoke tests.
- **Search:** indexed Postgres full-text at first; semantic/embedding search only if real retrieval tests show it improves results.
- **Costs:** separate usage tracking for Netlify, Supabase storage/compute and metered OpenAI API calls. Budget alerts and manual AI enable/disable.

Repository target structure:

```
app/
  pages/ components/ composables/ layouts/ stores/
server/
  api/ services/
shared/
  schemas/ types/
supabase/
  migrations/ tests/
mcp/
  src/ tests/
scripts/
  notion-import/ backfill/
tests/
  e2e/ fixtures/
docs/
  PRODUCT_PLAN.md
```

Make a clean build branch, keep the current static preview recoverable and cut over only after acceptance testing. Remove the outdated GitHub Pages workflow once the Netlify production path is verified.

## 5. AI assistant and ChatGPT MCP integration

The **in-app assistant** runs through the OpenAI API with server-side credentials. The **ChatGPT integration** connects to a separately authenticated remote MCP endpoint. Both call the same domain service functions and Supabase data. They are not the same thing, and the OpenAI API does not inherit ChatGPT memory/history.

### First tools to expose

Read: `get_today`, `search_workspace`, `list_tasks`, `get_project`, `list_due_followups`, `get_meeting`, `find_document`.

Write (consent and validation): `capture_task`, `update_task`, `complete_task`, `create_note`, `prepare_week_plan`, `record_evidence_update`.

Every write: resolves the authenticated user's workspace; uses least-privilege scopes; validates input; returns changed IDs; logs origin and before/after where appropriate. Destructive changes, bulk operations, external messages, disputed deadlines and project health overrides need explicit approval. Expose tool descriptions that distinguish factual source retrieval from AI inference.

Use HTTPS Streamable HTTP MCP; implement OAuth 2.1 with PKCE, metadata, token verification (issuer/audience/scopes), refresh/revocation and protected-resource discovery. Use a compatible existing identity provider rather than homebrew auth. Choose a stable dedicated runtime if Netlify's serverless streaming limitations appear in integration tests. Verify using MCP Inspector, then connect in ChatGPT Plugins.

AI should also power: task decomposition on request, summarising sourced notes, inbox classification suggestions, weekly review drafts, semantic context retrieval where useful, and one-click 'help me start' plans. All summaries of third-party documents must preserve provenance.

### Non-negotiable controls

- AI must not invent deadlines, decisions, commitments, project statuses or evidence conclusions.
- Email is **draft-only by default**. Sending, deleting, archiving or marking read requires the user's approval.
- Incoming emails and documents are untrusted data, not instructions for the agent.
- Log proposed vs executed actions separately; show reasons and links to evidence.
- Keep sensitive employer/client documents in a permitted workspace with explicit authorisation; do not automatically ingest confidential Dansom files into a personal cloud.
- No open public MCP write endpoint and no broad service-role access.

## 6. Notion migration map

| Notion database | Clearpath |
|---|---|
| Master Tasks | tasks + source links |
| Projects | projects |
| Meetings | meetings + decisions + commitments |
| Email Intake | email_threads + ingestion and review metadata |
| Documents & Evidence | documents + versions + attachments |
| Evidence Requests & Commitments | evidence_requests + commitments |
| Stakeholders | stakeholders and contact relations |
| Skills/Study databases | skills, courses, practice_sessions, assignments |

Import order: projects and references → stakeholders → documents → meetings → evidence/commitments → tasks → email references → skills. Preserve original Notion page IDs in `external_id_map` for deduplication and reconciliation.

Migration must support resumable dry runs, per-record errors, relationship-repair passes and a final dashboard of source vs imported counts. Validate representative project chains end-to-end, including document URL, task status, deadline confidence, meeting decision and related stakeholders.

Keep Notion read-only for a verification period and **do not turn off existing email/meeting automations until new ingestion is proven**. If both run in parallel, deduplicate on source thread ID/message key rather than creating two records.

For local preview users, provide an explicit JSON import path that retains existing browser-local tasks, notes and projects.

## 7. Delivery phases and acceptance gates

### Phase 0 — Infrastructure and secure foundation
Refactor the single HTML page into Nuxt, create development/preview/prod configuration, link GitHub→Netlify, configure Supabase and auth, implement RLS/migrations and CI.

**Pass when:** deployment is accessible, login works, one user cannot read another's test records, secrets are not present in the built bundle, and builds are reproducible.

### Phase 1 — Daily operating system
Build fast capture, Inbox, Today/top-three, Tasks (search/filter/board), Projects, Notes, calendar and due/scheduled separation.

**Pass when:** full task/project/notes CRUD works on desktop and mobile; edits sync across devices; project-task relations persist; exports can be restored.

### Phase 2 — ADHD execution and knowledge
Add Focus, time estimation, low-energy and short-task filters, interruption capture, recurring routines, quick reviews, pages, uploads, search, and accessibility.

**Pass when:** a user can capture → choose → work → log without visiting a full database editor; no unwanted streak pressure; uploads and search respect workspace permissions.

### Phase 3 — Work and school modules
Add Meetings, Decisions, Evidence Requests, Stakeholders, Documents & versions; then courses, assignments, skill sessions and weekly planning.

**Pass when:** a real FAO/KST-style project can track sources, requests, actions and deliverables without Notion; school assignments and study blocks share the Today calendar.

### Phase 4 — Migration
Read Notion exports/API, build importer, dry-run all entities, reconcile cross-record links, resolve collisions, validate selected complete projects.

**Pass when:** source and target counts reconcile, no unexplained missing records, broken internal relations or silent truncation, and owner approves comparison.

### Phase 5 — AI and MCP
Implement assistant, tool permission system, approvals, retrieval, MCP OAuth server, audit logs and evaluation cases.

**Pass when:** source-grounded reads are correct, explicit authorised writes round-trip into the UI, rejected actions are not applied, and ChatGPT successfully uses authenticated tools.

### Phase 6 — Integrations and cutover
Add Gmail and Calendar ingestion, review queues, duplicate detection, scheduled summaries, reminders, background sync and reliability monitoring. Only then select Clearpath as primary workspace and sunset Notion.

**Pass when:** no duplicate task from repeated email sync, every generated draft/action is reviewable, alarms/retries are observable, and rollback/export is exercised.

## 8. First implementation sprint

1. Create `feature/clearpath-2` from `master`; preserve preview.
2. Scaffold Nuxt/TypeScript, navigation, design tokens and authenticated shell.
3. Set up Supabase project, migration files and ownership policies.
4. Implement typed `projects`, `tasks`, `pages` schemas and CRUD.
5. Build Inbox, Today, Projects, Tasks and Notes with responsive layouts.
6. Deploy preview through Netlify; smoke-test on Android and desktop.
7. Verify cross-device persistence, forbidden cross-user access, backup and core task flow.

**Do not start** AI features, Gmail sync or Notion destruction during this sprint.

## 9. Review criteria and risks

- **Scope creep:** full Notion database builder and multi-user collaboration are deferred.
- **Privacy:** work/client data requires permission, per-workspace isolation and audit trails.
- **Loss of records:** preserve backup branches, browser backup import and Notion source until acceptance.
- **AI errors:** high-impact outputs remain suggestions with attributable evidence.
- **Automation drift:** active Notion syncing continues until new Clearpath sync can be validated side by side.
- **Offline conflicts:** implement explicit versions/conflict resolution; do not silently last-write-win on high-value records.
- **Cost:** monitor API/model usage; keep ordinary task CRUD AI-free.

## 10. Final definition of a Notion replacement

Clearpath qualifies as the owner's replacement only when their routine work and studies can be managed without returning to Notion for essential tasks, project records, evidence, documents, meetings, email triage or review; imports are reconciled; the AI tools are secured; mobile and desktop sync; exports work; and the user explicitly accepts the cutover.

Current status: **Plan only.** The original static preview, Netlify deployment state, migrations, authentication and MCP implementation are not changed by this document.
