# Product Requirements Document

**Project:** Event Hub
**Target Release:** 1.0

---

## 1. Overview

### Purpose

Provide an integrated system to manage events efficiently — from registration to post-event certification — suitable for small to medium-sized academic or professional gatherings. The system should minimize manual work, centralize data, and improve attendee experience.

### Objectives

- Centralize event setup, registration, and tracking.
- Automate post-event evaluations and certificate delivery.
- Allow per-event customization of forms and surveys.
- Operate as an independent web application, cloud-ready and extendable.

---

## 2. Core Features

### 2.1 Event Management

- Admins can create and manage multiple events.
- Each event includes configurable details: title, description, dates, venue, and banner.
- Each event defines its own **registration form** and **survey form**.
- System auto-generates:

  - Public registration link
  - Survey link
  - QR or unique attendee pass code
  - Admin dashboard

---

### 2.2 Registration

- Public registration page (responsive, accessible).
- Customizable fields (text, select, checkbox, email, etc.).
- Automatic validation for required inputs.
- Attendee receives confirmation email with QR pass and short registration code.
- Email-based deduplication to prevent duplicate registrations.

---

### 2.3 Check-In

- Barcode/QR-based check-in using dedicated scanners (HID emulation) or manual code entry.
- Real-time updates to the event dashboard.
- One valid check-in per attendee per event.
- Offline fallback via attendee registration code.

---

### 2.4 Survey & Certificate

- Each event has a **custom post-event survey** defined by the admin.
  - Survey links are unique per attendee (tokenized, embed code in email).
- Upon submission:

  - Record survey responses
  - Automatically generate and email a PDF certificate

- Admins can re-send or download certificates on demand.

---

### 2.5 Event Dashboard

- Key metrics:

  - Registrations (total)
  - Check-ins (unique)
  - Survey completion rate
  - Certificates issued

- Charts:

  - Registration and check-in timeline
  - Audience breakdown
  - Funnel view: registered → checked-in → surveyed → certificated

- Lists:

  - Recent check-ins
  - Certificate delivery status

- Exports:

  - CSV of all attendees with form responses and statuses

---

## 3. Roles & Permissions

| Role                  | Permissions                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------- |
| **Admin (Organizer)** | Create/edit events, configure forms, monitor dashboards, export data, manage certificates |
| **Attendee**          | Register, check-in, complete survey, receive certificate                                  |

---

## 4. Non-Functional Requirements

### Scalability

- Designed for up to 10 concurrent events and ~10,000 attendees per year.

### Performance

- API responses < 200 ms typical load.
- Certificate delivery within 5 minutes post-survey submission.

### Security

- Tokenized, expiring QR codes.
- Minimal personal data retention.
- All write operations are idempotent and traceable.

### Reliability

- Automatic retries for email and certificate delivery.
- Durable, consistent data storage.

### Usability

- Mobile-first, accessible interface.
- Simplified admin workflows: **create → share → monitor**.

---

## 5. Future Enhancements (Post-MVP)

- Multi-session tracking and booth analytics.
- Badge printing and physical device integrations.
- Participation gamification (passport, lucky draw).
- Integration with organizational identity systems (e.g., Microsoft 365, Azure AD).
- Enhanced analytics and feedback reporting.

---

## 6. Success Metrics

| Metric                    | Target                                     |
| ------------------------- | ------------------------------------------ |
| Registration completion   | ≥ 90%                                      |
| Check-in accuracy         | ≥ 99% unique QR/code validation            |
| Survey response rate      | ≥ 60% of check-ins                         |
| Certificate email success | ≥ 98%                                      |
| Organizer satisfaction    | “Simpler than Google Forms + certificates” |

---

## 7. Constraints

- Utilize free-tier cloud hosting and services when possible.
- Web-first system without SMS; optional support for commodity 2D barcode scanners.
- Generic academic branding and simple visual design.

---

## 8. Assumptions

- Stable internet connectivity during events.
- Attendees can access their registration QR or code on a personal device or printed copy.
- One administrative account manages multiple events.
- Certificates follow a standard institutional format and layout.
