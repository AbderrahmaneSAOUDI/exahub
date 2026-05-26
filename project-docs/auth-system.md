# Authentication & Access Control System

> **Document Version:** 2.1 — Greenfield Build
> **Last Updated:** May 26, 2026

---

## 1. Authentication Provider

### Firebase Authentication — Google + Email/Password

| Property | Value |
|---|---|
| Provider | Firebase Authentication |
| Method | Google Sign-In and Email/Password |
| Required profile fields | `displayName`, `email` |
| Phone auth | Excluded from MVP |
| Anonymous auth | Not used |
| Other social providers | Disabled |

### Registration Flow

```text
User opens app -> Auth screen
    -> Chooses Google or Email/Password
    -> Firebase creates account (UID)
    -> Server trigger creates users document:
       {
         id,
         email,
         displayName,
         isContributor: false,
         isModerator: false,
         isProfessor: false,
         isAdmin: false,
         downloadCount: 0,
         uploadCount: 0,
         createdAt
       }
```

---

## 2. User States & Privileges

### Standard User
- Browse approved exams
- Upload submissions (go to pending review)
- Download cap: 10 files total
- Download behavior: encrypted in-app file storage
- If limit reached: keep download button active, show popup each click

### Contributor User
- Unlocked after first approved upload
- Unlimited downloads
- Counters continue increasing (`downloadCount`, `uploadCount`) for statistics

### Professor User
- Assigned by admin
- Uploads are auto-approved
- Unlimited downloads
- Counters continue increasing and visible in profile

### Moderator User
- Assigned/promoted by admin
- Can approve/reject submissions
- Uploads are auto-approved
- Unlimited downloads
- Counters continue increasing and visible in profile

### Admin User
- Full access to app operations
- Promotes users to moderators
- Manages role assignment (`isModerator`, `isProfessor`, `isAdmin`)

---

## 3. Access Control Matrix

| Feature | Anonymous | Standard | Contributor | Professor | Moderator | Admin |
|---|---|---|---|---|---|---|
| Browse approved exams | No | Yes | Yes | Yes | Yes | Yes |
| Upload exam (pending) | No | Yes | Yes | No | No | No |
| Upload exam (auto-approved) | No | No | No | Yes | Yes | Yes |
| Download encrypted files | No | Yes (max 10) | Yes | Yes | Yes | Yes |
| Moderate approvals/rejections | No | No | No | No | Yes | Yes |
| Promote moderators | No | No | No | No | No | Yes |

---

## 4. 10-Download Guard Logic

```text
User taps Download
    -> Read authoritative user state
    -> If Standard and downloadCount >= 10
          show modal: [Upload Now] [Upload Later]
          (modal is closable)
    -> Else allow secure download
    -> On successful delivery: increment downloadCount for ALL roles
```

### UX Rules
- Modal is closable.
- Buttons are exactly: **Upload Now** and **Upload Later**.
- Download button never disappears; it always triggers logic.

### Counter Rules
- `downloadCount` increments after successful delivery for all users.
- `uploadCount` increments after successful approved uploads.
- Counters stay tracked even after contributor unlock; contributor users just do not see hard cap behavior.

---

## 5. Role Transition: Standard -> Contributor

```text
Moderator/Admin approves first pending upload
    -> Move file Pending -> Approved (same deep path structure)
    -> Update exam approvalStatus = true
    -> Update user isContributor = true
    -> Increment uploadCount
    -> Send approval notification
```

Notes:
- Contributor status remains permanent in MVP.
- `downloadCount` remains tracked in background for later analytics/score systems.

---

## 6. File Security Model

### 6.1 Watermarking
- Watermarking is performed once at upload processing time.
- Files are not repeatedly watermarked at each download.
- Output is the stored file used for future delivery.

### 6.2 Encrypted In-App Storage (All Users)
- All delivered files are stored as encrypted in-app blobs.
- Applies to Standard, Contributor, Professor, Moderator, and Admin users.
- Web uses browser-private encrypted storage wrapper.
- Mobile uses internal app storage with encryption.

### 6.3 Screenshots
- Screenshots are allowed.
- No screenshot prevention is implemented.

---

## 7. Firebase Auth Configuration Checklist

- [ ] Enable Google provider
- [ ] Enable Email/Password provider
- [ ] Disable Phone and Anonymous
- [ ] Disable unused social providers
- [ ] Ensure signup captures `displayName`
- [ ] Enforce server-side checks for moderator/admin operations
