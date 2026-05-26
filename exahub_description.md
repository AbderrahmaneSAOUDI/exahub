# EXAHUB — EXAM ARCHIVE — FULL PROJECT DOCUMENTATION

> **Version:** 1.0  
> **Status:** MVP Definition Complete — Ready to Build  
> **Last Updated:** May 2026

---

## TABLE OF CONTENTS

1. [Project Overview](#1-project-overview)
2. [Motivation & Intent](#2-motivation--intent)
3. [Problem Being Solved](#3-problem-being-solved)
4. [Target Users & Expansion Path](#4-target-users--expansion-path)
5. [Institutional Foundation](#5-institutional-foundation)
6. [Technical Architecture](#6-technical-architecture)
7. [Database Schema](#7-database-schema)
8. [Access Model](#8-access-model)
9. [Content Model](#9-content-model)
10. [Upload Flow](#10-upload-flow)
11. [Moderation System](#11-moderation-system)
12. [Anti-Piracy Strategy](#12-anti-piracy-strategy)
13. [Distribution Strategy](#13-distribution-strategy)
14. [Gamification (Future)](#14-gamification-future)
15. [MVP Scope](#15-mvp-scope)
16. [What Is Explicitly Excluded from MVP](#16-what-is-explicitly-excluded-from-mvp)
17. [Success Metrics](#17-success-metrics)
18. [Self-Sustaining Design](#18-self-sustaining-design)
19. [Future Roadmap](#19-future-roadmap)
20. [Pre-Build Checklist](#20-pre-build-checklist)

---

## 1. PROJECT OVERVIEW

**ExaHub** is a free, community-powered mobile application that centralizes past university exam papers in one organized, searchable platform. Students can browse, filter, and download exam papers from previous years and semesters. The platform is sustained by student contributions — users who upload approved exams unlock additional features, creating a self-reinforcing contribution loop.

The app is built with the intent of being a public utility — free forever, self-sustaining after setup, and designed to outlast its founder.

---

## 2. MOTIVATION & INTENT

This project is built as **sadaqa jariya** — an ongoing charitable act intended to benefit students continuously. The secondary goals are:

- Learn Flutter development through a real, complete project
- Build a portfolio project that covers all aspects of mobile development
- Create something genuinely useful that people use and benefit from

**This is not a business.** There is no monetization in scope. Revenue, investors, and growth metrics are not the measure of success. Student benefit and project sustainability are.

---

## 3. PROBLEM BEING SOLVED

Algerian university students waste significant time before exams searching for past papers. The content exists — it is scattered across:

- Dead Telegram links
- Fragmented WhatsApp group threads
- Senior students who may or may not respond
- Unreliable USB transfers between students

The result is that students spend time they should be studying on hunting for exam papers instead. The app eliminates this friction entirely by centralizing all past papers in one organized, searchable, always-available platform.

**The single pain point:** Students always ask their friends for past exams and search for them in Telegram and Messenger groups. The app replaces that search with a reliable, organized archive.

---

## 4. TARGET USERS & EXPANSION PATH

### Expansion Phases

| Phase | Target | Trigger to Expand |
|---|---|---|
| **Alpha** | CS students, University of Ghardaia | Launch |
| **Phase 2** | All departments, University of Ghardaia | Alpha is self-sustaining without founder |
| **Phase 3** | All Algerian universities | Ghardaia model fully replicated |
| **Phase 4** | BAC students nationally | University model nationally proven |

### Why This Order

**Ghardaia CS** is the training ground. The founder is a master's student there — zero cold-start problem, direct access to users, professors, and the complete module list. Dean approval already obtained.

**All Algerian universities** is the destination. 1.8 million university students represent the primary total addressable market.

**BAC** is the long-term opportunity. The Baccalauréat is a national standardized exam published publicly by the Ministry of Education — zero copyright exposure, uniform content serving every high school student in Algeria. This is addressed only after the university model is nationally proven.

### Rule: Never expand before the current phase runs itself

Ghardaia CS must operate without founder involvement before a single new university is added. The self-sustaining Ghardaia model becomes the replicable template.

---

## 5. INSTITUTIONAL FOUNDATION

- **Dean of University of Ghardaia** has verbally approved the project and offered to help source exams from professors
- **Written confirmation required** before launch — one email to the dean summarizing the conversation and requesting a reply confirmation
- **App disclaimer** included regardless of written confirmation: *"Content uploaded by students for educational purposes"*
- **Founder's position** as a master's student on campus provides direct access to professors, students, the complete module list, and the existing student social networks

---

## 6. TECHNICAL ARCHITECTURE

### Stack Overview

| Layer | Technology | Reason |
|---|---|---|
| Frontend | Flutter | Cross-platform, single codebase, offline-capable |
| Authentication | Firebase Authentication | Simple, free, reliable |
| Database | Firebase Firestore | Lightweight metadata store, free at current scale |
| File Storage (Pending) | Google Drive — "Pending" folder | Founder has 2TB, zero cost |
| File Storage (Approved) | Google Drive — "Approved" folder | Same account, organized by approval state |
| File Delivery | Google Drive API | Serves approved files directly to users |
| Local Storage | Encrypted on-device storage | Invisible in device file manager |

### Why Google Drive Over Firebase Storage

Firebase Storage free tier caps at 1GB storage and 10GB/month downloads. At 200+ exams and growing, this ceiling is reached quickly. Google Drive provides 2TB of existing storage at zero cost. Firebase is used only for authentication and Firestore metadata — never for file storage.

### Architecture Flow

```
User opens app
    → Firebase Auth verifies session
    → Firestore fetches exam metadata (no file transfer yet)
    → User browses and filters metadata
    → User requests download
    → App fetches file via Google Drive API using stored Drive link
    → File delivered: stored as encrypted local blob for all users
```

### Performance Considerations

- **Aggressive metadata caching** on device — Firestore data cached locally so app does not call the database on every screen load
- **Exponential backoff** on failed API requests — handles exam-season traffic spikes gracefully
- **File compression** applied at upload — keeps Drive storage and download sizes manageable on 3G connections and budget phones (2GB RAM target)
- **Google Drive API quota** — current free tier allows 1,000,000 quota units per minute per project. At 350 users downloading 5 files each simultaneously = ~350,000 quota units. Safe at current scale.

---

## 7. DATABASE SCHEMA

### Exam Document (Firestore Collection: `exams`)

| Field | Type | Values / Notes |
|---|---|---|
| `id` | Auto-generated | Unique exam identifier |
| `moduleName` | String | Free text — e.g., "Algorithmique Avancée" |
| `year` | String | Format: "2025-2026" |
| `examType` | String (Dropdown) | See Exam Types below |
| `semester` | String (Radio) | "First" or "Second" |
| `university` | String (Dropdown) | Expands as platform grows |
| `department` | String | Free text for now — structured format in future update |
| `approvalStatus` | Boolean | `false` = pending, `true` = approved |
| `contributorId` | String (Reference) | User ID of uploader |
| `driveFileLink` | String | Direct Google Drive link to approved file |

### Exam Type Dropdown Values

| Value | Description |
|---|---|
| Exam Cours | Lecture / course exam |
| Exam Rattrapage | Makeup / resit exam |
| Exam TD | Directed tutorial exam |
| Exam TP | Practical work exam |
| Fiche TD | TD worksheet / handout |
| Fiche TP | TP worksheet / handout |

### User Document (Firestore Collection: `users`)

| Field | Type | Notes |
|---|---|---|
| `id` | Auto-generated | Firebase Auth UID |
| `email` | String | Registration email |
| `displayName` | String | Used for display/scoring |
| `isContributor` | Boolean | `false` by default; `true` when first upload approved |
| `isModerator` | Boolean | `false` by default |
| `isProfessor` | Boolean | `false` by default |
| `isAdmin` | Boolean | `false` by default |
| `downloadCount` | Integer | Tracks downloads for all users |
| `uploadCount` | Integer | Total approved uploads by this user |
| `createdAt` | Timestamp | Registration date |

---

## 8. ACCESS MODEL

Multiple permanent user states. Simple logic, no subscriptions, no payments.

### Standard User
- **How to get it:** Register an account
- **What you can do:** Browse all approved exams, download up to **10 files** total
- **Download type:** Encrypted local storage caching
- **Upgrade path:** Submit one exam that gets approved → permanently becomes Contributor

### Contributor User
- **How to get it:** Have one uploaded exam approved by a moderator
- **What you can do:** Browse all approved exams, download unlimited files
- **Download type:** Encrypted local storage caching
- **Status:** Permanent — never revoked

### Professor / Moderator / Admin
- **Privileges:** Assigned by admin. Uploads auto-approve for professors/mods. Unlimited downloads. Counters remain active. Includes specific management privileges based on role tier.

### The Logic Behind the Model

- 10 free downloads is generous enough to be useful for one exam session — a student can revise for an upcoming exam without being forced to contribute
- The 10-file ceiling creates a natural, non-aggressive incentive to upload — students who find the app valuable will unlock more rather than abandon it
- Contributor status is permanent — upload once, benefit forever — so the exchange feels fair and proportionate
- No solutions, no AI, no private content — exam papers only, always

### Download Limit Behavior

When a Standard user attempts to download their 11th file, the app displays:

> *"You've reached your 10-file limit. Upload one approved exam to unlock unlimited downloads."*

With a direct link to the upload screen.

---

## 9. CONTENT MODEL

### Seed Content
- **200 exams** already collected by the founder — enough to give the app genuine value from day one without depending on user contributions at launch
- These are uploaded by the founder directly with admin access before public launch

### Crowdsourced Uploads
- Any registered user can submit an exam via the in-app upload form
- All 9 metadata fields are mandatory — upload button remains disabled until all fields are completed
- Accept PDF and image files (JPG/PNG)
- Submitted file goes directly to the Google Drive "Pending" folder
- Firestore record created with `approvalStatus: false`

### Content Quality Control
The structured upload form is the first filter. By requiring all fields — university, department, module, year, semester, exam type — the form eliminates approximately 80% of spam and low-quality submissions before they reach a human moderator.

---

## 10. UPLOAD FLOW

```
Student opens Upload screen
    → Completes all 9 mandatory fields (dropdowns, radio, text)
    → Selects PDF file from device
    → Taps Submit
    → File uploads to Google Drive "Pending" folder
    → Firestore record created (approvalStatus: false)
    → Student sees: "Your exam is under review. You will be notified when approved."
    → Moderator receives pending item in moderation panel
    → Moderator reviews → Approves or Rejects
    → If Approved:
        → File moved to Google Drive "Approved" folder
        → Firestore approvalStatus flipped to true
        → driveFileLink updated to approved folder link
        → User's isContributor flipped to true
        → Push notification sent: "Your exam was approved. You now have unlimited downloads."
    → If Rejected:
        → File deleted from Google Drive "Pending" folder
        → Firestore record deleted
        → Push notification sent: "Your submission was not approved. Please check the upload guidelines."
```

---

## 11. MODERATION SYSTEM

### Who Moderates
Two volunteer student moderators recruited from the existing CS network at Ghardaia before launch. Each mod is assigned to a specific academic year cohort so the workload is distributed and manageable.

### Mod Responsibilities
- Review pending uploads in the moderation panel
- Approve or reject based on: correct metadata, readable PDF, genuine exam paper, no duplicates
- Send one WhatsApp message to the CS group after every exam session: *"Exam just finished? Upload it here → [deep link]"*

### Moderation Panel Features (MVP)
- List of all pending submissions with full metadata visible
- File preview before decision
- Approve button — triggers full approval flow
- Reject button — triggers deletion and notification
- No complex tools — two buttons per submission

### Why Moderation Cannot Be Skipped
Open uploads without moderation turn the archive into a chaotic dump within weeks. Bad content — blurry photos, wrong modules, duplicate files — destroys student trust faster than no content. The structured form reduces noise. Human review ensures quality.

### Post-Founder Continuity
When the founder graduates or becomes unavailable, moderator access is transferred to trusted successors. Full system documentation ensures handover is possible without the founder's involvement. Three named backup administrators are documented before launch.

---

## 12. ANTI-PIRACY STRATEGY

### Watermarking — Primary Defense
Every approved file is watermarked with the app name at upload/processing time. Leaked PDFs become free advertising. A student who shares an exam outside the app inadvertently promotes the platform. Piracy becomes a distribution channel rather than a threat.

### Encrypted Local Downloads — Secondary Defense
All downloads (regardless of user role) are stored in encrypted local storage, invisible in the device file manager. Students cannot easily locate and forward the raw file through standard file sharing. This adds friction without destroying the user experience.

### Contributor Account Integrity
Shared accounts lose upload history and therefore lose contributor status verification. Sharing login credentials has a meaningful cost — the recipient gets standard access, not contributor access, because the upload record is tied to the original account.

### What Was Rejected and Why
- **Screenshot prevention** — trivially bypassed by pointing another phone at the screen; waste of development time
- **DRM** — over-engineering for this scale and audience
- **Download prevention** — contradicts the offline download feature entirely

---

## 13. DISTRIBUTION STRATEGY

### Launch Distribution
1. Founder shares app in Ghardaia CS WhatsApp and Telegram groups directly
2. Friends download and test — organic word of mouth within a small, known community
3. Dean endorsement communicated to students — institutional legitimacy accelerates trust

### Ongoing Distribution
- Moderator sends WhatsApp message after every exam session with upload link
- Watermarked leaked files spread app name organically
- Contributor conversion creates invested users who naturally recommend the app to peers

### Why WhatsApp-First
WhatsApp and Telegram groups are the primary discovery mechanism for Algerian students. Fighting this is futile. The distribution strategy uses existing student group infrastructure rather than competing with it.

---

## 14. GAMIFICATION (FUTURE)

**Gamification is not part of the MVP.** It is a planned future feature.

### Planned Features (Post-MVP)

**Yearly Leaderboard**
- Displays top contributors by approved upload count
- Resets every January 1 at 00:00
- Labeled: "Best Contributors of 2026"
- Scoped by university when the platform scales — national comparison is irrelevant; local comparison drives behavior

**Contributor Visibility**
- Contributor tag visible on user profile
- Upload count displayed publicly on profile
- Module Champion tag — student with most uploads for a specific module gets named next to that module in the browse view

**Why Deferred**
Gamification adds development complexity without changing the core value proposition at MVP scale. The 10-file standard limit and contributor unlock are sufficient behavioral incentives for the alpha phase. Gamification is added when the community is large enough for competition to be meaningful.

---

## 15. MVP SCOPE

### What Is Built in Version 1.0

**Authentication**
- Email registration and login
- Two user states: Standard and Contributor
- Contributor boolean tracked on user record

**Home / Browse Screen**
- Full list of approved exams
- Filter by: University, Department, Module name, Year, Semester, Exam type
- Total exam count displayed — social proof
- No search bar — filters sufficient at 200-exam scale

**Exam Detail Screen**
- All metadata fields displayed
- Download button — disabled for Standard users after 10 files with upgrade prompt
- No comments, ratings, or sharing UI

**Upload Screen**
- 9-field mandatory form matching database schema
- PDF-only file selection
- Submit triggers Drive upload and Firestore record creation
- Confirmation message shown after submission

**Moderation Panel**
- Accessible to mod accounts only
- Pending submissions list with metadata and file preview
- Approve and Reject buttons
- Approval triggers contributor status unlock and push notification

**Download System**
- Online streaming for Standard users (within 10-file limit)
- Encrypted local storage for Contributors
- Watermark applied to all files before delivery

**Push Notifications**
- Upload approved — notify uploader
- Upload limit reached — prompt to upload

---

## 16. WHAT IS EXPLICITLY EXCLUDED FROM MVP

| Feature | Reason Excluded |
|---|---|
| Camera scan | Adds 3+ weeks of dev time — future update |
| Gamification / leaderboard | Not needed to validate core model — future update |
| Verified solutions | Not in scope — exam papers only |
| AI features | Not in scope |
| Comments and ratings | Unnecessary complexity at MVP scale |
| Search bar | Filters cover the need at 200-exam scale |
| Google / social sign-in | Unnecessary auth complexity |
| Multi-language support | Single language first — future update |
| Analytics dashboard | Not needed until scale justifies it |
| Payment system | Not in scope — free forever |
| BAC content | University model must be proven first |
| Profile editing | Nothing to edit at MVP stage |
| Social sharing features | Watermark handles organic sharing |

---

## 17. SUCCESS METRICS

### First 4 Weeks After Launch

| Metric | What It Tells You |
|---|---|
| Registration count | Is there genuine demand? |
| Downloads per user | Are students actually using the archive? |
| Upload submissions | Is the contribution model working? |
| Approval rate | Is content quality acceptable? |
| **Contributor conversion rate** | **How many Standard users hit 10 files and upload?** |
| Mod response time | Is the moderation system sustainable? |

### The One Number That Matters Most

**Contributor conversion rate** — the percentage of Standard users who hit the 10-file limit and upload an exam to unlock more.

- If this number is healthy → the core loop works; the platform is self-sustaining
- If this number is near zero → students hit the limit and abandon; the product has a fundamental problem

---

## 18. SELF-SUSTAINING DESIGN

The platform is designed to operate without the founder after initial setup.

| Function | Who Does It | Founder Required |
|---|---|---|
| New exam uploads | Students, triggered by mod prompt | No |
| Post-exam upload prompts | Designated mod sends WhatsApp message | No |
| File approval / rejection | Volunteer moderators | No |
| Content organization | Structured form auto-populates Firestore | No |
| File delivery | Google Drive API | No |
| New user registration | Self-serve | No |

### What Still Needs Occasional Attention
- Flutter version updates — approximately every 6 months
- Google API changes — monitor annually
- New moderator onboarding when existing mods graduate
- Adding new universities to the dropdown as the platform expands

**Goal: Low maintenance, not zero maintenance.**

### Continuity Plan
Before launch, the following must exist:
1. Three named administrators with full system access documented
2. Written documentation covering: Drive folder structure, Firebase project details, Firestore schema, credential locations, and moderation workflow
3. Written confirmation from the dean on file

---

## 19. FUTURE ROADMAP

### Phase 2 — Ghardaia University (All Departments)
- Add all departments to the university dropdown
- Recruit one mod per new department
- Replicate the CS moderation workflow exactly

### Phase 3 — All Algerian Universities
- Add universities to the dropdown progressively
- Each new university gets its own mod team
- Leaderboards scoped by university

### Phase 4 — Gamification Layer
- Yearly leaderboard with January reset
- Module Champion tags
- Contributor profile visibility
- Upload count on public profiles

### Phase 5 — Camera Scan
- In-app camera capture for physical exam papers
- Auto-convert to PDF before upload
- Reduces upload friction for students with paper copies only

### Phase 6 — BAC Integration
- National Baccalauréat past papers (publicly published by Ministry of Education)
- Zero copyright exposure — government content
- Separate module/subject structure from university schema
- Potentially the largest user segment on the platform

---

## 20. PRE-BUILD CHECKLIST

Complete these before writing a single line of Flutter code:

- [ ] Email the dean for written confirmation of project approval
- [ ] Recruit 2 volunteer moderators from the CS student network
- [ ] Compile complete module list for Ghardaia CS — all modules, all years
- [ ] Run 7-day Google Form experiment: post a simple upload form in the CS WhatsApp group with no explanation; count organic submissions after 7 days; understand whether passive contribution is viable or requires active prompting
- [ ] Set up Google Drive folder structure: `/Pending` and `/Approved`
- [ ] Create Firebase project — Authentication + Firestore
- [ ] Enable Google Drive API and obtain credentials
- [ ] Compress and organize 200 seed exams with correct metadata before admin upload

---

*This document reflects the complete project definition as of the end of the negotiation and stress-test phase. All decisions recorded here are final for Version 1.0. Future updates are appended as new phases.*
