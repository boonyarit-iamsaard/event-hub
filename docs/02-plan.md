# Phase 1 — Foundation & Event Creation

## Goal

Create/manage events, dynamic registration form, confirmation email with QR.

## Tech stack

- **Frontend:** Next.js (App Router, TS), React Hook Form + Zod, Tailwind.
- **Backend:** ASP.NET Core (.NET 8) Web API (controller-based).
- **DB:** PostgreSQL (**JSONB** for custom fields) — Azure Flexible Server (basic) or Neon (free/serverless); Railway managed Postgres as lightweight fallback.
- **Email:** SendGrid (free tier).
- **QR:** QRCoder (QR image), JWT (System.IdentityModel.Tokens.Jwt) for short-lived scan tokens.
- **Hosting:** Vercel (web), Azure App Service (API).
- **Auth (admin):** Cookie/session (ASP.NET Identity) or simple JWT for admin.

## Deliverables

- Admin: create/edit **Event** (title, dates, venue, banner, **registration_schema**) via controller endpoints.
- Public: `/events/[slug]` renders form from schema; client + server validation.
- Registration submission → create/merge **Attendee**, **Registration** (answers in JSONB).
- Email confirmation with **pass link** and embedded QR (view page can mint short-lived tokens).
- DB migrations (EF Core) for `events`, `attendees`, `registrations`.

## Acceptance

- Event can be created and registration published.
- New registrant gets email with working pass link and QR within 60s.
- Duplicate email per event is prevented (idempotent).

---

## Phase 2 — Check-in (QR) & Admin Ops

### Phase 2 Goal

On-site validation with a web scanner; simple admin list & exports.

### Phase 2 Tech stack

- **Scanner:** Next.js admin console tuned for HID barcode/QR scanners (auto-focus input, optional ZXing WASM fallback for browsers that support cameras).
- **Backend:** Check-in controller actions; idempotent insert; rate-limit basic abuse.
- **Exports:** CsvHelper for CSV (server-side streaming).

### Phase 2 Deliverables

- Staff `/scan` console optimized for HID scanner input → POST `/api/checkins` with short-lived token.
- Check-in writes unique `(event_id, attendee_id)` row; returns `{ firstTime: boolean }`.
- Admin list: registrants, search/filter, check-in status, re-send pass.
- Export CSV: registrations + check-ins.
- Support entry of short registration codes and USB/Bluetooth barcode scanner input as fallback for QR scanning.

### Phase 2 Acceptance

- p95 scan→ack < 2s on normal wifi.
- Duplicate scans don’t create extra rows.
- CSV export opens in Excel/Sheets with correct headers.

---

## Phase 3 — Survey & Certificate Automation

### Phase 3 Goal

Custom per-event survey; submission triggers PDF certificate + email.

### Phase 3 Tech stack

- **Survey UI:** Next.js page rendered from **survey_schema** (RHF + Zod).
- **Certificates:** QuestPDF (A4 template with logo); store in **object storage** (Azure Blob Storage or S3-compatible bucket).
- **Email:** SendGrid with signed download link (expiry).

### Phase 3 Deliverables

- Admin defines **survey_schema** per event.
- Unique survey link per attendee (tokenized); link from pass page and shareable QR.
- On submit: save **Survey** (answers in JSONB), generate PDF, upload, email.
- Admin: re-send certificate, download certificate, resend failures.

### Phase 3 Acceptance

- Survey submit → certificate email received within 5 minutes.
- PDFs render correct name, event, date; link guarded (time-limited or one-time).
- Admin can re-send to a specific attendee.

---

## Phase 4 — Minimal Dashboard & KPIs

### Phase 4 Goal

Give organizers live visibility (no OLAP).

### Phase 4 Tech stack

- **Charts:** Recharts (frontend).
- **Backend aggregates:** direct SQL (Postgres) with `date_trunc` bucketing.

### Phase 4 Deliverables

- KPIs: registrations, check-ins, survey completion, certificates sent.
- Charts: **hourly** registrations vs check-ins; **funnel** (checked-in → surveyed → certificated).
- Audience split (internal/external) if captured.
- Recent check-ins list; certificate failures list.

### Phase 4 Acceptance

- Summary and charts load < 300ms server time.
- Numbers match exports when spot-checked.

---

## Phase 5 — Production Hardening & DevEx

### Phase 5 Goal

Reliability, security, and developer workflow.

### Phase 5 Tech stack

- **CI/CD:** GitHub Actions → Vercel (web) + Azure App Service (API).
- **Secrets:** GitHub Encrypted Secrets + Azure Key Vault (optional).
- **Observability:** Serilog + App Insights (API logs/metrics).
- **Testing:**

  - Backend: xUnit + Testcontainers (Postgres) exercising controller layers.
  - Frontend: Playwright (critical flows).

### Phase 5 Deliverables

- CI: build, test, deploy on main.
- Basic rate limiting on check-ins, input size limits, request logging.
- Email retry (exponential backoff), dead-letter for failed sends.
- Backups: daily Postgres snapshot; storage lifecycle policy (cert PDFs).

### Phase 5 Acceptance

- Green pipeline from PR → prod.
- Logged audit trail for admin actions (create event, export, resend cert).
- Recovery drills: restore from backup in staging.

---

## Cross-cutting Application Structure

- Controller-based ASP.NET Core Web API exposing feature-focused endpoints (`EventsController`, `RegistrationsController`, `CheckInsController`, etc.) backed by annotated DTOs and service layer abstractions.

## Cross-cutting Data Model (EF Core)

- `events(id, slug, name, starts_at, ends_at, venue, registration_schema_jsonb, survey_schema_jsonb, created_by)`
- `attendees(id, full_name, email, affiliation, is_internal, created_at)`
- `registrations(id, event_id, attendee_id, status, answers_jsonb, created_at)` **UNIQUE(event_id, attendee_id)**
- `checkins(id, event_id, attendee_id, checked_in_at, gate)` **UNIQUE(event_id, attendee_id)**
- `surveys(id, event_id, attendee_id, answers_jsonb, submitted_at)`
- `certificates(id, event_id, attendee_id, pdf_path, issued_at, email_status)`

_JSONB gives form flexibility without leaving Postgres._

---

## Environments & Config

- **Local:** Dockerized Postgres + storage emulator (Azurite or MinIO).
- **Staging:** Same as prod, feature flags for email sandbox.
- **Prod:** Vercel + Azure App Service (F1) + managed Postgres (Azure Flexible Server or Neon); Railway Postgres kept as a later option if consolidating services. SendGrid free tier.

---

## Risks & Mitigations

- **Email deliverability:** set SPF/DKIM; add manual resend & bounce report.
- **PDF throughput:** queue large batches; cap concurrency; pre-warm templates.
- **Scanner hardware issues:** ensure manual registration code entry and spare scanner hardware are available.
- **Hardware availability:** recommend at least one laptop/tablet (Chromebook-class or better) per check-in gate plus a wired USB or Bluetooth 2D (QR-capable) scanner (~฿1,200–฿8,000 in TH market).
- **Schema drift:** validate schemas server-side; version `*_schema_jsonb`.

---

## Stretch (post-MVP, drop-in)

- **OneDrive/SharePoint export** (M365 Graph).
- **Session/booth scans** (same check-in pipeline; add `mode` and `location_id`).
- **Edge cache for check-ins** (Redis) if gates get spiky.
- **Azure AD admin SSO** (if institutional adoption happens).
