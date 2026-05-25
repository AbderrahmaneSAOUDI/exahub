# Post-MVP / Incomplete Features

To maintain momentum and strict execution velocity during the initial build, all secondary, highly complex, and post-MVP features have been deferred. These features are cataloged here so they do not distract from executing the core version 1.0 (MVP) release.

---

## 1. Gamification Layer
Gamification is deferred until the community is large enough for competitive dynamics to be meaningful.

### Planned Features:
* **Yearly Leaderboard**:
  * Displays a ranking of top contributors by total approved upload count.
  * Resets automatically every January 1st at 00:00.
  * Labeled: *"Best Contributors of [Year]"*.
  * **Scoped Scope**: Restricted to local departments/universities. National comparison is less engaging than local department competition.
* **Contributor Visibility & Social Proof**:
  * Visible "Contributor" badge/tag on the user profile view.
  * Approved upload count displayed publicly.
  * **Module Champion Tag**: The user with the highest approved uploads for a specific module gets highlighted directly next to that module in the main browser list.

---

## 2. In-App Camera Scanner
Adding direct image-capture and scanning flows increases development time, asset management, and library dependencies.

### Planned Features:
* In-app scanner to take pictures of physical paper copies of exams.
* Automated crop, deskew, and contrast enhancement filters.
* Automated local conversion to PDF format prior to initiation of the Google Drive upload pipeline.
* **Current MVP Alternative**: Users must compile, convert, and upload their files using external tools (like Adobe Scan, CamScanner, or system prints) and upload the resulting PDF.

---

## 3. Baccalauréat (BAC) Integration
The Baccalauréat represents the largest potential user segment in Algeria, but introducing it during the University launch introduces unnecessary architectural complexity.

### Planned Features:
* Separate high school schema.
* Standardized national BAC past papers (official Ministry of Education releases).
* Zero copyright exposure (government content is public domain).
* Scaled-up content serving system for hundreds of thousands of concurrent high school students.

---

## 4. Advanced Browsing & Social Features

### In-App Search Bar
* **Reason for Deferral**: With ~200 seed exams in Ghardaia CS at launch, structural dropdown filters (Year, Semester, Module, Exam Type) are highly efficient and sufficient for fast browsing. Search text parsing adds index management costs without significant UX benefit.

### Comments, Peer Reviews, & Ratings
* **Reason for Deferral**: Introduces moderation overhead (monitoring comments for spam, verbal abuse, or leaking of sensitive data) and database read/write complexity. The platform remains an archive, not a social discussion board.

### Google / Social Auth Sign-In
* **Reason for Deferral**: Minimizes initial setup friction and reduces dependency on external auth package integrations that require extensive OAuth consent screens and app verification. Email/Password sign-up is simple, standard, and highly effective for MVP.

### Multi-Language Support
* **Reason for Deferral**: Focuses first on Ghardaia CS students, where single-language localized UI suffices. Translation workflows are deferred to a post-launch phase.

### Analytics Dashboard
* **Reason for Deferral**: System-wide performance and engagement metrics are checked directly via Firestore telemetry and Google Drive access metrics. In-app dashboards add excessive design work.
