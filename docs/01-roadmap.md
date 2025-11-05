# Delivery Roadmap — _Event Hub (1.0)_

## Phase 1 — Core Setup (2–3 weeks)

**Goal:** Establish working foundation for event creation and registration.

**Key Deliverables:**

- Admin authentication (basic)
- Create/manage events
- Public registration page (custom fields)
- Email confirmation + QR generation

## Phase 2 — Check-In System (1–2 weeks)

**Goal:** Enable on-site attendee tracking.

**Key Deliverables:**

- Check-in console optimized for HID barcode/QR scanners
- Check-in validation + dashboard update
- Basic admin table for attendance with manual code entry fallback

## Phase 3 — Survey & Certificate Automation (2–3 weeks)

**Goal:** Close the event feedback loop.

**Key Deliverables:**

- Per-event survey customization
- Tokenized survey links
- Automated PDF certificate generation + email
- Re-send and download support

## Phase 4 — Dashboard & Exports (1–2 weeks)

**Goal:** Provide organizers real-time visibility and reporting.

**Key Deliverables:**

- Event dashboard: KPIs + charts
- CSV exports (registration, survey, certificates)
- Event summary view

## Phase 5 — Polish & Rollout (1 week)

**Goal:** Final quality, performance, and UX adjustments.

**Key Deliverables:**

- UI refinements and accessibility
- Retry logic for emails
- Cloud deployment (App Service / Vercel / managed Postgres)
- Basic documentation

---

## Estimated Timeline

> **Total: 6–10 weeks** (solo developer, part-time pace)
> MVP usable by end of **Phase 3** (around week 6).

---

### Milestone Readiness Criteria

| Milestone            | Readiness Criteria                                                                  |
| -------------------- | ----------------------------------------------------------------------------------- |
| **MVP Ready**        | Event creation, registration, check-in, and certificate flow functional end-to-end. |
| **Usability Ready**  | Admin UI intuitive for non-technical staff; can run events independently.           |
| **Production Ready** | Stable deployment, backups, and SSL-secured endpoints.                              |
