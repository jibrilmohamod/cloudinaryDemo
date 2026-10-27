# Clearpath v0.1 — Senior Engineering Baseline

Status: **Proposed, not implemented**
Repository: `jibrilmohamod/cloudinaryDemo`
Working branch: `feature/clearpath-v01`
Target: Netlify + Supabase
Reference: `docs/PRODUCT_PLAN.md` (long-term direction); this document supersedes its MVP boundaries.

## Decision record

**Decision:** Build a private, single-user-first, mobile-responsive daily execution tool: **Capture → Clarify → Pick 3 → Focus → Complete**. Keep Notion in service until Clearpath works reliably and migration is audited.

**Why not a full Notion clone:** Actual Notion records have rich relations, email provenance, meeting links, evidence and deadline confidence. Copying all this into v0.1 creates several products and exposes data to migration risk before the basic capture-and-execute experience is useful.

**Why not AI-first:** Every essential action must work without an LLM. AI can later use the same authenticated business functions. An MCP endpoint requires separate authentication and ChatGPT account compatibility tests.

### Core audience and constraints

- Primary user on Windows browser and Android/iOS browsers; app installed as a PWA when supported.
- Areas: Work, School, Personal and Admin.
- Cross-border work means unreliable networks: show sync state and protect edits from silent loss. Offline capture queue is the first follow-up if needed, not fake offline support.
- Some client information may be confidential: no actual employer data or credentials in the public repo; cloud import only with permission.
- Dates and time: store absolute instants in UTC; store all-day deadlines as date-only and the user's IANA timezone (default Africa/Nairobi). Scheduled time is not a deadline.

## MVP feature boundary

| Ship in v0.1 | Defer deliberately |
|---|---|
| Email/password or supported secure login; account isolation | Teams, workspace sharing, roles |
| Global quick capture, inbox and search | Notion-style custom database builder |
| Task editing and completion; status/priority/area/project | Kanban drag-and-drop, complex recurrence |
| Due date vs planned date vs waiting reason | Google Calendar sync and AI scheduling |
| Exactly up to 3 **user-chosen** Today priorities | Automatic AI reordering |
| Projects with next action and active tasks | Full portfolio health automation |
| Task-linked focus timer and accurate session logging | Gamification, streaks, mood tracking |
| Responsive site; keyboard navigation | Native mobile apps |
| Export/restore JSON and cross-device sync | Full Notion migration, attachments, rich docs |
| Automated tests and production deploy | In-app AI chat and MCP integration |

*Notes*: Task details/description can hold short notes; dedicated knowledge base comes later.

### Representative acceptance journeys

1. **10-second capture:** user opens the site, presses Ctrl/Cmd+K or the mobile + control, types a title and saves without filling other fields.
2. **Truthful plan:** moving a task to tomorrow does **not** mutate its confirmed deadline. 'Waiting' remains distinct from 'Done'.
3. **Today:** the user selects at most 3 priorities; attempting a fourth gives a clear replace-or-cancel choice, not a hidden reorder.
4. **Focus:** a timer starts against a task, can pause/resume, and records actual elapsed active time. Closing the browser must not invent elapsed focus.
5. **Multi-device:** creating a task on PC, reloading on phone, and editing there updates PC after refresh. A failed write shows error and retained input.
6. **Isolation:** signed-out or other-user requests cannot read, update or delete the first user's data, even if a row ID is known.
7. **Recovery:** JSON export includes IDs, links and timestamp metadata; restore into a clean test account with documented collision handling.

## System architecture — modular monolith

```text
Windows / Android / iPhone browsers
        |
    Nuxt 4 PWA
        |
        | HTTPS API (/api/*)
        v
Nuxt Nitro server handlers (deployed on Netlify)
  |-> Authenticated task / project / today / focus services
  |-> Validation and business invariants (e.g. 3 pins)
  |-> Optional future AI/MCP adapters (NOT MVP)
        |
        | Supabase SDK with caller's validated auth context
        v
Supabase Auth + Postgres (RLS)
        |
   Database migrations, indexes, owner isolation

Later: OpenAI API -> same domain services
Later: OAuth-protected MCP -> same domain services
```

Architecture rules:
- One deployable app and one database. No microservices, message bus, queues, vector DB, or separate MCP deployment in v0.1.
- API endpoints act as a stable boundary for future mobile/agent clients. Server validates input and permissions; database independently enforces user ownership with RLS.
- Never expose a Supabase service-role/secret key or OpenAI credentials in frontend output. Prefer user-scoped Supabase JWT operations and RLS.
- Shared typed request/response schemas, versioned database migrations, and predictable error responses.
- No passive AI ingestion; external emails are not connected.

## Minimal SQL domain schema (conceptual; migration must be reviewed)

- `projects`: id UUID PK, user_id UUID FK auth.users, name, area, status, next_action, created_at, updated_at, deleted_at.
- `tasks`: id UUID PK, user_id, project_id nullable, title, description, area, status, priority, deadline_on DATE nullable, deadline_confidence, planned_on DATE nullable, waiting_on, source_type nullable, source_id nullable, source_url nullable, estimate_minutes nullable, completed_at, revision INT, created_at, updated_at, deleted_at.
- `daily_priorities`: user_id, for_date DATE, slot SMALLINT CHECK 1..3, task_id FK, unique(user_id,for_date,slot), unique(user_id,for_date,task_id). Set/update in a transaction; tasks and pins must have same owner.
- `focus_sessions`: id, user_id, task_id nullable, started_at, finished_at nullable, elapsed_active_seconds, state, created_at.
- `profiles`: user_id PK, timezone IANA, preferred_focus_minutes, created_at.

Do not add a generic polymorphic 'all content' table or 25+ empty tables in v0.1.

Implement status and priority enums/check constraints matching actual Notion terms so a future import preserves meaning: status=Inbox, Not started, Up next, In progress, Waiting, Blocked, Done, Cancelled; priority=Critical, High, Normal, Low.

### Key invariants

- RLS on all exposed tables; grants + separate SELECT/INSERT/UPDATE/DELETE policies (USING and WITH CHECK).
- Database checks ensure linked project/task belongs to same user, not merely a valid global ID.
- Index (user_id, status, planned_on), (user_id, deadline_on), (user_id, project_id), and focus task FK where needed; evaluate actual query patterns.
- Soft deletion with deleted_at so accidental deletes can be recovered.
- Optimistic update uses revision; stale edits must show conflict, not silently overwrite another device.
- Never invent deadlines; carry `deadline_confidence` and original source references in imported data.

## API contract (first slice)

- `POST /api/tasks` — accepts title only; creates Inbox task.
- `GET /api/tasks?status=&project=&search=` — paginated list.
- `PATCH /api/tasks/:id` — validated partial update with revision check.
- `GET /api/today?date=YYYY-MM-DD` — chosen three, due/overdue, scheduled, waiting.
- `PUT /api/today/priorities` — replace ordered list of <=3 task IDs atomically.
- `GET/POST/PATCH /api/projects` — basic project CRUD (with archive).
- `POST /api/focus-sessions` and `PATCH /api/focus-sessions/:id` — focus lifecycle.
- `GET /api/export` + validated `POST /api/import` — versioned JSON backup / restore.

All endpoints require authenticated user except login and healthcheck; mutation schemas are strict; no arbitrary row/user IDs trusted.

## UI structure

Desktop: persistent compact sidebar; command search/capture; Today primary screen. Mobile: short navigation (Today, Inbox, Projects, Focus, More). Each screen has one obvious primary action.

Today:
1. Pick/confirm three priorities; nothing silently auto-selected.
2. Start one task now, optionally with a five-minute start.
3. See due, scheduled, waiting, overdue separately.
4. End-of-day short review (Done / Move / Waiting) without streak pressure.

Provide accessible labels, tab order, escape-to-close, reduced motion, light/dark theme, loading/saving/errors/empty states.

## Engineering workflow & gates

**Gate 0: Audit and publish pipeline**
- Verify GitHub repo default branch and current Netlify project Git connection. Current Netlify site has no verified published deployment in connector results; do not claim live.
- Keep `master` unchanged while building on `feature/clearpath-v01`.
- Configure Netlify preview deployments and runtime env configuration (no secrets in git).
- Add lint/typecheck/tests/build to CI.

**Gate 1: Database/auth**
- Supabase project (explicit account and project), migrations, RLS policies, index checks and two-user isolation tests. Use a non-production test DB.
- Validate no anonymous leakage and that user A cannot guess user B's task IDs.

**Gate 2: One vertical slice**
- Login -> create one task -> database write -> read -> edit -> complete -> reload on another device.
- Real error/loading states before adding features.

**Gate 3: Full MVP UX**
- Today pins (transactional constraint), Inbox, Projects, Focus, fast mobile capture, keyboard flows.
- JSON import/export round trip.

**Gate 4: Verification and launch**
- E2E tests (Playwright) on desktop and mobile viewport; accessibility and keyboard smoke; API authentication/ownership tests; failed-network checks; backup restore; secret scan and correct live link.
- Code review/PR; merge only when production deployment is verified.

### Failure and rollback policy

- Merge through PR; preserve current static app and old Cloudinary demo archive branch.
- Database migration tested for backward compatibility; backup before destructive operations.
- Failed deploy never considered a successful launch; do not change Notion or turn off automations.
- Roll back frontend through Git if necessary, avoid irreversible DB migrations until proven.

## What constitutes 'done'?

A real user can sign in, create and organise tasks, choose exactly three priorities, log a truthful focus session, and continue on a second device. The app has documented backup/restore, automated RLS tests and a verified production URL. It can replace the daily task execution part of Notion. It cannot replace all Notion databases yet.

## First developer PR

1. Generate Nuxt 4 + TypeScript shell from supported current tooling.
2. Configure Supabase client/server auth integration with test environment.
3. Add migration for `projects`, `tasks`, `daily_priorities`, `focus_sessions`, RLS and indexes.
4. Implement `POST /api/tasks`, `GET /api/tasks`, login + Inbox.
5. Add a true browser-to-database E2E test and cross-user denial test.
6. Deploy a working Netlify preview and record URL, logs and test results.

No assistant, MCP, Gmail, migration or multi-user team features in first PR.

## Open decisions (default until evidence changes)

- **Framework:** Nuxt 4 with Vue 3 + TypeScript, leveraging previous project familiarity, not a gratuitous framework switch.
- **Backend:** Nuxt/Nitro endpoints deployed with Netlify; if unsupported behavior is found, fix with official adapter rather than designing microservices.
- **Persistence/Auth:** Supabase Postgres + Auth, owner_id RLS.
- **Public code:** acceptable; personal and client data never checked into repo.
- **Offline:** truthful online-first in v0.1, with explicit save failure; offline inbox capture can follow if field usage tests justify it.
- **Notifications:** not in v0.1; browser notification permissions never required for task capture.
