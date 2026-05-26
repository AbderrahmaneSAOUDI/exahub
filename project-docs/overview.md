# Project Overview — ExaHub (Exam Archive)

> **Document Version:** 1.0 — Greenfield Build
> **Source of Truth:** `exahub_description.md` v1.0
> **Last Updated:** May 26, 2026

---

## 1. Core Purpose

**ExaHub** is a free, community-powered mobile and web application that centralizes past university exam papers in one organized, searchable, always-available platform. Students browse, filter, and download exam papers from previous years and semesters. The platform is sustained by student contributions — users who upload approved exams unlock additional features, creating a self-reinforcing contribution loop.

**Intent:** Built as **sadaqa jariya** — an ongoing charitable act intended to benefit students continuously.

**Nature:** A pure public utility. There is no monetization, no ads, no premium subscriptions, no investors, and no revenue generation. Student benefit and project sustainability are the only success metrics.

---

## 2. Problem Statement

Algerian university students waste significant exam-preparation time searching for past papers across scattered, unreliable sources:

| Source | Problem |
|---|---|
| Telegram links | Expire or go dead within weeks |
| Telegram group threads | Fragmented, unsearchable, buried in noise |
| Senior students | May be unreachable, slow, or graduated |
| USB transfers | Unreliable, require physical proximity |

**The single pain point:** Students always ask friends and search in groups for past exams. ExaHub replaces that hunt with a reliable, structured, always-available archive.

---

## 3. Target Audience & Expansion Strategy

### Expansion Phases

| Phase | Target | Trigger to Expand |
|---|---|---|
| **Alpha** | CS students, University of Ghardaia | Launch |
| **Beta** | All departments, University of Ghardaia | Alpha is self-sustaining without founder |
| **Phase 3** | All Algerian universities | Ghardaia model fully replicated |
| **Phase 4** | BAC students nationally | University model nationally proven |

### Why This Order

- **Ghardaia CS** is the training ground. The founder is a master's student there — zero cold-start problem. Direct access to users, professors, complete module list, and existing social networks.
- **1.8 million** university students represent the primary total addressable market.
- **BAC** (Baccalauréat) is the long-term opportunity — publicly published by the Ministry of Education, zero copyright exposure.

### Iron Rule

> Never expand to a new phase until the current phase runs itself without founder intervention.

---

## 4. Technology Stack

| Layer | Technology | Rationale |
|---|---|---|
| **Frontend** | Flutter (Web + Mobile) | Cross-platform single codebase, offline-capable, budget-device friendly |
| **Authentication** | Firebase Authentication | Free-tier friendly, Google + Email/Password at MVP |
| **Database** | Cloud Firestore | Lightweight metadata store, aggressive client-side caching |
| **File Storage** | Google Drive API (2TB account) | Bypasses Firebase Storage 1GB/10GB limits entirely |
| **Watermarking** | App name stamped on every PDF during the upload process | One-time task, results in persistent organic advertising |
| **Local Storage** | Encrypted in-app cache | Files exist only as encrypted blobs inside the app's internal cache — never as standalone files on the device |
| **Push Notifications** | Firebase Cloud Messaging | Upload approval and limit-reached alerts |

### Why Google Drive Over Firebase Storage

Firebase Storage free tier: **1GB storage, 10GB/month downloads**. At 200+ PDF exams and growing, this ceiling is hit within weeks. Google Drive provides **2TB** at zero cost. Firebase is used **only** for auth and Firestore metadata — never for file storage.

---

## 5. Architecture Flow

```
User opens app
    → Firebase Auth verifies session
    → Firestore fetches exam metadata (no file transfer yet)
    → User browses and filters metadata (cached locally)
    → User requests download
    → App fetches file via Google Drive API using stored Drive link
    → File delivered:
        - All Users → encrypted and cached inside the app internal storage
        - Standard User → increments downloadCount (blocked after 10)
        - Contributor/Mod/Prof → unlimited downloads
```

### Key Data Flow Boundaries

1. **Client ↔ Firestore**: Metadata only (exam records, user state). Aggressively cached on-device.
2. **Client ↔ Google Drive**: PDF file delivery. Must go through a secure boundary (server-side proxy or signed URLs) — browser code must never hold Drive service account credentials.
3. **Moderator ↔ Drive**: File move operations from `/Pending` to `/Approved` or `/Rejected` folders. Requires server-side or admin-scoped access.

---

## 6. Performance Constraints & Targets

| Constraint | Target |
|---|---|
| Minimum device RAM | 2GB |
| Minimum connectivity | 3G |
| Firestore metadata | Cached locally, stale-while-revalidate |
| File compression | Applied at upload time |
| Drive API quota | 1,000,000 units/min (safe at ~350 concurrent users × 5 downloads) |
| Backoff strategy | Exponential backoff on failed API requests |

---

## 7. Access Model Summary

| Feature | Standard User | Contributor User |
|---|---|---|
| **Entry** | Email registration | 1 approved upload |
| **Download limit** | 10 files total | Unlimited |
| **Download type** | Encrypted in-app cache (10-file cap) | Encrypted in-app cache (unlimited) |
| **Cost** | Free | Free |
| **Duration** | Until limit hit | Permanent |

The 10-file limit creates a natural, non-aggressive incentive. Upload once → benefit forever.

---

## 8. Content Model

### Seed Content

- **200 exams** pre-collected by the founder before launch.
- Uploaded via admin access before public availability.
- Provides genuine value from day one without depending on user contributions.

### Crowdsourced Uploads

- Any registered user can submit via 9-field mandatory form.
- Accept PDF and image files (JPG/PNG).
- All fields must be completed before submit is enabled.
- Files go to deep path format: `/Pending/{Country}/{University}/{Speciality}/{ExamType}/{file_name}` → Firestore record created with `approvalStatus: false`.

---

## 9. Moderation Model

- **Two volunteer student moderators** recruited from Ghardaia CS network before launch.
- Each mod assigned to a specific academic year cohort.
- Panel: list of pending submissions → file preview → Approve or Reject (two buttons).
- **Approval** triggers: file move from `/Pending/...` to `/Approved/...` (same deep path), Firestore update, contributor unlock, push notification.
- **Rejection** triggers: file move from `/Pending/...` to `/Rejected/...` (same deep path), Firestore record deletion, rejection notification.

### Post-Founder Continuity

- Three named backup administrators documented before launch.
- Full system documentation enables handover without founder involvement.

---

## 10. Anti-Piracy Strategy

| Layer | Mechanism | Effect |
|---|---|---|
| **Watermarking** | App name stamped once during upload processing | Leaked files become organic advertising |
| **Encrypted in-app cache** | Files exist only as encrypted blobs inside the app's internal cache — never as extractable PDFs on the device | Sharing requires screen recording or re-photographing, both low-quality |

### Rejected Approaches

- Screenshot prevention → Decision cancelled; screenshots are fully allowed
- DRM → over-engineering for this scale
- Download prevention → contradicts in-app offline viewing feature

---

## 11. Success Metrics (First 4 Weeks Post-Launch)

| Metric | Signal |
|---|---|
| Registration count | Genuine demand exists |
| Downloads per user | Archive is actively used |
| Upload submissions | Contribution model works |
| Approval rate | Content quality is acceptable |
| **Contributor conversion rate** | **Core loop health** |
| Mod response time | Moderation system is sustainable |

### The One Number That Matters

> **Contributor conversion rate** — the percentage of Standard users who hit the 10-file limit and upload an exam.

> - Healthy → platform is self-sustaining
> - Near zero → fundamental product problem

---

## 12. Self-Sustaining Design

| Function | Operator | Founder Required? |
|---|---|---|
| New exam uploads | Students (triggered by mod prompt) | No |
| Post-exam upload prompts | Designated mod via Telegram | No |
| File approval/rejection | Volunteer moderators | No |
| Content organization | Structured form → Firestore | No |
| File delivery | Google Drive API | No |
| User registration | Self-serve | No |

### Occasional Maintenance

- Flutter version updates (update when needed - feature/security driven)
- Google API changes (monitor annually)
- New moderator onboarding when mods graduate
- Adding new universities to dropdown on expansion

---

## 13. Distribution Strategy

1. **Launch**: Founder shares in Ghardaia CS Telegram groups.
2. **Ongoing**: Moderator sends Telegram message after every exam session with upload link.
3. **Viral**: Watermarked leaked files spread the app name organically.

- **Telegram** is the primary discovery mechanism for Algerian students.

---

## 14. Internal Classification Tags

| Tag | Scope |
|---|---|
| `#auth` | Firebase Auth, user states, role logic, session management |
| `#data` | Firestore schema, exam/user documents, cache layer |
| `#api` | Google Drive API, file delivery, upload pipeline |
| `#ui` | Screens, navigation, filters, empty states, modals |
| `#admin` | Moderation panel, approval/rejection flow |
| `#media` | PDF handling, watermarking, encrypted storage |
| `#infra` | Environment config, secrets, build tooling |
| `#logic` | Download guard, role transitions, business rules |
| `#testing` | Unit, widget, and integration tests |
| `#web` | Web-specific concerns (CORS, browser storage, no filesystem) |
