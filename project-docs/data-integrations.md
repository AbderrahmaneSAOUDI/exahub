# Data Integrations & Storage Architecture

> **Document Version:** 2.1 — Greenfield Build
> **Last Updated:** May 26, 2026

---

## 1. Firestore Database Schema

Cloud Firestore stores metadata only. Binary files are stored on Google Drive.

### 1.1 `exams` Collection

| Field | Type | Constraints | Description |
|---|---|---|---|
| `id` | String | Auto-generated | Firestore document ID |
| `moduleName` | String | Required | Module name |
| `year` | String | Required (`YYYY-YYYY`) | Academic year |
| `examType` | String | Required | Exam Cours, Exam Rattrapage, Exam TD, Exam TP, Fiche TD, Fiche TP |
| `semester` | String | Required | First / Second |
| `university` | String | Required | University name |
| `department` | String | Required | Department name |
| `speciality` | String | Required | Speciality/track name |
| `approvalStatus` | Boolean | Required, default `false` | Pending/Approved |
| `contributorId` | String | Required | Uploader UID |
| `driveFileLink` | String | Required | Approved Drive file ID/link |
| `fileType` | String | Required | `pdf`, `jpg`, `jpeg`, `png` |

### 1.2 `users` Collection

| Field | Type | Constraints | Description |
|---|---|---|---|
| `id` | String | Required | Firebase UID |
| `email` | String | Required | Account email |
| `displayName` | String | Required | User name for profile/score system |
| `isContributor` | Boolean | Default `false` | Unlocks unlimited downloads |
| `isModerator` | Boolean | Default `false` | Managed by admin; can approve/reject |
| `isProfessor` | Boolean | Default `false` | Auto-approved upload role |
| `isAdmin` | Boolean | Default `false` | Full app permissions + role management |
| `downloadCount` | Integer | Default `0` | Tracks successful downloads for all users |
| `uploadCount` | Integer | Default `0` | Tracks approved uploads |
| `createdAt` | Timestamp | Required | Registration timestamp |

### State Transition Rules

```text
Register -> Standard (downloadCount: 0, uploadCount: 0)
    -> Successful download: downloadCount++ (all roles)
    -> Standard reaches 10: popup shown on every download click
    -> First approved upload: isContributor = true, uploadCount++
       (downloadCount still increases for analytics)
```

---

## 2. Google Drive Folder Architecture

### 2.1 Folder Structure

```text
/ExaHub
  /Pending/{Country}/{University}/{Speciality}/{ExamType}/{file_name}
  /Approved/{Country}/{University}/{Speciality}/{ExamType}/{file_name}
  /Rejected/{Country}/{University}/{Speciality}/{ExamType}/{file_name}
```

Example path:

```text
/ExaHub/Pending/Algeria/University_of_Ghardaia/Computer_Science/Exam_Cours/algo_2025_s1.pdf
```

All three statuses (`Pending`, `Approved`, `Rejected`) use the same deep subfolder hierarchy.

### 2.2 Service Account Integration

| Operation | Actor | Scope |
|---|---|---|
| Upload pending file | App via server boundary | `drive.file` |
| Move pending -> approved | Moderator/Admin server flow | `drive` |
| Move pending -> rejected | Moderator/Admin server flow | `drive` |
| Serve approved files | Server boundary | `drive.readonly` |

Critical rule: service account credentials are server-only and never shipped in web bundles.

---

## 3. Upload Constraints

| Rule | Enforcement |
|---|---|
| File format | Accept PDF and images (`jpg`, `jpeg`, `png`) |
| File size | `<= 10MB` |
| Required metadata | All fields mandatory before submit |
| Watermarking | Apply once during upload processing |
| Compression | Optional optimization during upload processing |

---

## 4. Metadata Caching Strategy

- Use aggressive local cache of Firestore metadata.
- Render cache first, then refresh in background (stale-while-revalidate).
- Never block UI on first network request if cache exists.
- Keep cache freshness marker and visible subtle sync indicator.

---

## 5. Data Integrity Guarantees

| Operation | Atomic Requirements |
|---|---|
| Approval | Move Drive file + update exam status + update user contributor/upload counters + notify |
| Rejection | Move Drive file to Rejected path + remove pending exam record + notify |
| Download success | File delivered + increment `downloadCount` |

Consistency rules:
1. `downloadCount` increments after successful delivery for every role.
2. `isContributor` flips only after successful approval flow.
3. Role checks (`isModerator`, `isAdmin`) are enforced server-side.
4. Upload counters remain visible and cumulative in profile.

---

## 6. Sequence Diagrams

All sequence diagrams are maintained in:

- `project-docs/sequence-diagrams.md`

This includes:
- Upload and review (including rejection)
- Auto-approved uploads (Professor/Moderator/Admin)
- Download and popup guard behavior
- Auth/profile initialization
