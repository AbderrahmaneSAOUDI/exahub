# Project Overview — Exam Archive

## 1. Core App Purpose & Motivation
**Exam Archive** is a free, community-powered past university exam paper platform designed to centralize and organize academic resources.
- **Intent**: Built as **sadaqa jariya** (ongoing charity) to serve students continuously.
- **Nature of Project**: A pure public utility. There is no monetization, no ads, no premium subscriptions, and no revenue generation.
- **The Core Loop**: Sustained entirely by student contributions. Standard users get a generous but limited number of downloads, which they can permanently unlock to unlimited access by contributing a single verified exam paper.

## 2. Problem Statement
Algerian university students waste valuable exam-prep time searching for past papers across scattered, unreliable sources:
- Dead or expiring Telegram and Messenger links.
- Fragmented and disorganized WhatsApp group threads.
- Seniors who may be unreachable or slow to respond.
- Unreliable USB drives or local transfers.

Exam Archive solves this by providing a unified, searchable, always-available, and structured repository of past papers.

## 3. Target Audience & Expansion Strategy (Alpha Phase)
To ensure a successful roll-out and validate the self-sustaining community loop, the project implements a strict phased expansion.

### Focus: Alpha Phase
- **Target**: Computer Science (CS) students at the **University of Ghardaia**.
- **Rationale**: 
  - The founder is a master's student there, providing direct campus access to students and professors.
  - Zero cold-start problem: Direct access to complete course modules, exam histories, and existing student social networks.
  - **Dean's Approval**: Verbal approval has already been obtained. Official written confirmation (via email) is the final gate before launch.
  - **Institutional Support**: Professors can help source exams directly through the Dean's backing.

### Replicable Expansion Plan
- **Rule**: Never expand to a new phase until the current phase is fully self-sustaining (operating without direct founder intervention).

| Phase | Target Audience | Expansion Trigger |
|---|---|---|
| **Alpha (Current)** | CS Students, University of Ghardaia | Launch & Validate core submission loops |
| **Phase 2** | All departments, University of Ghardaia | Alpha runs itself without founder intervention |
| **Phase 3** | All Algerian universities | Ghardaia model replicated successfully |
| **Phase 4** | BAC (Baccalauréat) students nationally | University model proven nationwide |

---

## 4. Technology Stack (Flutter Web & Mobile)
The app is engineered to be lightweight, cross-platform, and highly optimized for budget devices (targeting 2GB RAM minimum profiles) and low-connectivity environments.

- **Frontend**: **Flutter (Web & Mobile)**
  - Single codebase for both mobile application and web-based viewing/moderation interfaces.
  - Built-in offline support and local caching of metadata.
- **Authentication**: **Firebase Authentication**
  - Simple, free-tier friendly, and highly secure.
- **Database**: **Cloud Firestore**
  - Lightweight storage for document/exam metadata.
  - Aggressive client-side caching implemented so the app operates offline and minimizes read costs.
- **File Storage**: **Google Drive API**
  - Uses the founder's 2TB Google Drive account.
  - Segregated folders: `/Pending` for submissions and `/Approved` for verified public files.
  - Bypasses Firebase Storage limits (1GB free limit is too restrictive for large scale PDF distribution).
- **Security & Delivery**:
  - **Watermarking**: Automated app watermarks applied to PDFs prior to user delivery.
  - **Encrypted Local Storage**: Available for Contributor users to download and view exams offline securely (files are saved invisibly on-device and kept out of public gallery/file managers).
