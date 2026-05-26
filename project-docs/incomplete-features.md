# Post-MVP / Incomplete Features

> **Document Version:** 2.0 — Greenfield Build
> **Source of Truth:** `exahub_description.md` v1.0
> **Last Updated:** May 2026

---

## Purpose

This document catalogs every feature that is **intentionally excluded** from the MVP (v1.0) build. Each item includes the planned scope, the specific reason for deferral, the prerequisite phase or trigger for re-evaluation, and any architectural considerations the MVP build should respect to avoid making future implementation harder.

> **Rule:** No item on this list becomes a work item until the core archive loop is stable and the current phase is self-sustaining.

---

## 1. Gamification Layer

**Planned Phase:** Phase 4 (post Algerian university expansion)
**Prerequisite:** Community is large enough for competitive dynamics to be meaningful.

### Planned Features

| Feature | Description |
|---|---|
| **Yearly Leaderboard** | Ranks top contributors by approved upload count. Resets every January 1st at 00:00. Labeled: *"Best Contributors of [Year]"*. |
| **Scoped Competition** | Leaderboard scoped by university — national comparison is irrelevant; local comparison drives behavior. |
| **Contributor Badge** | Visible "Contributor" tag on user profile view. |
| **Upload Count Display** | Approved upload count displayed publicly on profile. |
| **Module Champion Tag** | Student with the most approved uploads for a specific module gets highlighted next to that module in the browse view. |

### Reason for Deferral

Gamification adds development complexity (leaderboard queries, ranking logic, UI for badges and tags, reset cron jobs) without changing the core value proposition at alpha scale. The 10-file limit and contributor unlock are sufficient behavioral incentives for the initial user base.

### MVP Architectural Consideration

- The `users` collection already tracks `uploadCount`. No schema changes needed to enable leaderboards later.
- Future: may need an `uploads` subcollection or a `leaderboard` collection with pre-aggregated data for efficient ranking queries.
- The `exams` collection already links to `contributorId`, enabling Module Champion calculation without schema changes.

---

## 2. In-App Camera Scanner

**Planned Phase:** Phase 5
**Prerequisite:** Core upload flow proven and contribution loop validated.

### Planned Features

| Feature | Description |
|---|---|
| **Camera Capture** | In-app photo capture for physical paper copies of exams. |
| **Auto Processing** | Automated crop, deskew, and contrast enhancement filters. |
| **PDF Conversion** | Automatic conversion from captured images to PDF format before upload. |
| **Multi-Page Scan** | Capture and stitch multiple pages into a single PDF document. |

### Reason for Deferral

- Adds **3+ weeks** of development time for camera integration, image processing pipelines, and platform-specific permissions.
- Introduces heavy library dependencies (camera, image processing, PDF generation).
- **Current alternative**: Users compile and convert files externally using Adobe Scan, CamScanner, or system print-to-PDF, then upload the resulting PDF.

### MVP Architectural Consideration

- The upload pipeline accepts PDF and image files (`jpg`, `jpeg`, `png`).
- Future camera integration can still perform optional enhancement before upload.
- File size limits (≤10MB) should be enforced consistently for both PDFs and images.

---

## 3. Baccalauréat (BAC) Integration

**Planned Phase:** Phase 6
**Prerequisite:** University model proven nationwide (Phase 3 fully operational).

### Planned Features

| Feature | Description |
|---|---|
| **BAC Past Papers** | National Baccalauréat past papers from all years. |
| **Public Domain Content** | Officially published by Ministry of Education — zero copyright exposure. |
| **Separate Schema** | Different subject/module structure from university schema (streams: Sciences, Lettres, Math, Technique). |
| **Scale Readiness** | Serving potentially hundreds of thousands of concurrent high school students. |

### Reason for Deferral

- Introduces a fundamentally **different content model** (national standardized exams vs. university-specific exams).
- Requires a separate metadata schema (subjects, streams, national exam types vs. university modules, departments, exam types).
- Scale requirements are dramatically higher (~800K BAC students per year vs. ~350 university users at alpha).
- Must not introduce architectural complexity that distracts from proving the university model.

### MVP Architectural Consideration

- The `exams` collection schema should not be designed with BAC in mind. Keep it university-focused.
- When BAC is added, it will likely need its own collection (`bac_exams`) with a different schema, rather than overloading the `exams` collection.
- The authentication and user model can remain shared — a BAC user is still a Standard or Contributor user.
- The Google Drive folder structure should anticipate eventual expansion: consider `/ExaHub/University/Approved` and `/ExaHub/BAC/Approved` when the time comes.

---

## 4. Search Bar (Full-Text Search)

**Planned Phase:** Post-MVP (when exam count exceeds filter efficiency)
**Prerequisite:** Exam catalog grows beyond ~500 items where dropdown filters alone become cumbersome.

### Planned Features

| Feature | Description |
|---|---|
| **Text Search** | Search by module name, exam type, or keywords across all metadata fields. |
| **Autocomplete** | Suggestions as the user types, based on existing module names. |
| **Combined Filters + Search** | Search within filtered results (e.g., search "algo" within "2025-2026" → "First Semester"). |

### Reason for Deferral

- At **~200 seed exams** scoped to Ghardaia CS, dropdown filters (University, Department, Module, Year, Semester, Type) are highly efficient.
- Full-text search requires either Firestore full-text workarounds (client-side filtering, Algolia integration, or Cloud Functions) — all add cost and complexity.
- The structured filter system provides fast, deterministic browsing without index management overhead.

### MVP Architectural Consideration

- Module names are stored as free text. If search is added later, consider normalizing module names into a canonical list or adding a `moduleNameNormalized` field for matching.
- Firestore does not support native full-text search. Future options: Algolia, Typesense, or client-side search over cached metadata.

---

## 5. Comments, Peer Reviews & Ratings

**Planned Phase:** No specific timeline — evaluated post-scale
**Prerequisite:** Platform has a large enough community to moderate comment activity.

### Planned Features

| Feature | Description |
|---|---|
| **Quality Ratings** | Star or thumbs-up system for exam paper quality/readability. |
| **Report Mechanism** | Flag inappropriate content or incorrect metadata. |

### Reason for Deferral

- Introduces **significant moderation overhead** — monitoring comments for spam, abuse, or sensitive data leaks.
- Adds database read/write complexity (nested subcollections under each exam document).
- The platform is an **archive**, not a social discussion board. Comments don't serve the core use case.
- Report mechanism could be implemented standalone without full comments system.

### MVP Architectural Consideration

- No subcollections needed on `exams` documents at MVP. If comments are added later, they go in an `exams/{examId}/comments` subcollection.
- A lightweight "Report" button could be implemented independently as a simple Firestore write to a `reports` collection.

---

## 6. Social Sign-In Scope

**MVP Decision:** Keep exactly two providers enabled at launch.

- Google Sign-In
- Email/Password

All other providers remain disabled unless explicitly added later.

---

## 7. Multi-Language Support (i18n)

**Planned Phase:** Post-MVP
**Prerequisite:** Platform expands beyond Ghardaia (Phase 2+).

### Planned Languages

| Language | Priority | Audience |
|---|---|---|
| French | Primary MVP | Ghardaia CS students (French-medium instruction) |
| Arabic | Phase 2 | Broader Algerian university audience |
| English | Phase 3 | International accessibility |

### Reason for Deferral

- Single-language UI is sufficient for Ghardaia CS alpha.
- i18n infrastructure (`.arb` files, `intl` package, RTL layout support) adds meaningful development overhead.
- Arabic support requires RTL layout consideration — should be planned but not built into MVP.

### MVP Architectural Consideration

- **All user-facing strings should be externalized** from the start (no hardcoded strings in widget trees). This makes future i18n adoption significantly easier.
- Use Flutter's built-in `Localizations` pattern even if only one locale is supported initially.
- Module names and exam metadata remain in their original language (French) — they are not translated.

---

## 8. Analytics Dashboard

**Planned Phase:** Post-MVP, when scale justifies investment
**Prerequisite:** Phase 2+ with multiple departments/universities.

### Planned Features

| Feature | Description |
|---|---|
| **Admin Dashboard** | Registration trends, download volume, upload rates, conversion rates. |
| **Per-University Stats** | Engagement metrics scoped to each university. |
| **Mod Performance** | Average review time, approval/rejection ratios. |

### Reason for Deferral

- System metrics can be checked directly via Firebase Console, Firestore telemetry, and Google Drive access logs.
- In-app dashboards add design and development work with no user-facing benefit at alpha scale.
- The **one metric that matters** (contributor conversion rate) can be calculated manually from Firestore data.

### MVP Architectural Consideration

- No additional data collection needed at MVP. All metrics can be derived from existing `users` and `exams` collections.
- If Firebase Analytics is enabled (optional), it provides basic engagement metrics automatically.

---

## 9. Profile Editing

**Planned Phase:** Post-MVP
**Prerequisite:** There is something meaningful to edit (display name, avatar, bio).

### Reason for Deferral

- At MVP, user profiles contain only: email, contributor status, download count, upload count, and creation date.
- There is nothing to edit that changes functionality.
- Display names, avatars, and bio fields are part of the gamification and social features — both deferred.

---

## 10. Social Sharing Features

**Planned Phase:** Not currently planned
**Rationale:** Watermarking handles organic sharing. Explicit share buttons are unnecessary.

### Reason for Deferral

- Watermarked PDFs naturally spread the app name when shared externally.
- In-app share buttons add UI complexity without meaningful behavioral change.
- Students already share content via Telegram — adding share buttons doesn't change that behavior.

---

## Summary: MVP Exclusion Table

| # | Feature | Reason | Dev Time Saved | Future Phase |
|---|---|---|---|---|
| 1 | Gamification / Leaderboard | Not needed at alpha scale | 2-3 weeks | Phase 4 |
| 2 | Camera Scanner | External tools sufficient | 3+ weeks | Phase 5 |
| 3 | BAC Integration | Different content model | 4+ weeks | Phase 6 |
| 4 | Search Bar | Filters sufficient at 200 exams | 1-2 weeks | Post-500 exams |
| 5 | Comments & Ratings | Moderation overhead, not archive's purpose | 2 weeks | Post-scale |
| 6 | Social Sign-In | Email/password sufficient | 1 week | Post-MVP |
| 7 | Multi-Language | Single language sufficient for alpha | 2 weeks | Phase 2+ |
| 8 | Analytics Dashboard | Firebase Console sufficient | 2-3 weeks | Phase 2+ |
| 9 | Profile Editing | Nothing to edit at MVP | <1 week | Post-gamification |
| 10 | Social Sharing | Watermarking handles this | <1 week | Not planned |
