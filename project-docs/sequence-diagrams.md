# Sequence Diagrams — ExaHub Flows

This file contains the core behavioral logic for ExaHub, from user uploads to moderator approvals and secure downloads.

---

## 1. Unified Upload & Review Pipeline

Covers Student uploads (Pending), Professor/Mod/Admin auto-approvals, and Moderator review flows.

```mermaid
sequenceDiagram
    autonumber
    actor U as User (Any Role)
    participant APP as Flutter App
    participant SRV as Server Boundary (Functions)
    participant FS as Firestore
    participant GD as Google Drive
    actor M as Moderator

    U->>APP: Fills 9-field form + Selects PDF/Image
    Note over APP: Client-side validation:<br/>PDF/JPG/PNG, ≤10MB
    APP->>SRV: Upload request (File + Metadata)

    SRV->>GD: Watermark file once during upload processing

    alt isProfessor OR isModerator OR isAdmin
        SRV->>GD: Upload to "/Approved/{Country}/{University}/{Speciality}/{ExamType}/..." path
        SRV->>FS: Create exam record (approvalStatus: true)
        APP-->>U: "Exam Uploaded & Auto-Approved!"
    else Standard/Contributor User
        SRV->>GD: Upload to "/Pending/{Country}/{University}/{Speciality}/{ExamType}/..." path
        SRV->>FS: Create exam record (approvalStatus: false)
        APP-->>U: "Your exam is under review"

        Note over M: Moderator opens review panel
        M->>APP: Previews File + checks metadata

        alt APPROVED
            M->>SRV: Approve action
            SRV->>GD: Move File from "/Pending/{...}" to "/Approved/{...}" (same deep path)
            SRV->>FS: Update exam (approvalStatus: true)
            SRV->>FS: Update user (isContributor: true, increment uploadCount)
            APP-->>U: Notification: "Exam Approved! Unlimited Access Unlocked!"
        else REJECTED
            M->>SRV: Reject action
            SRV->>GD: Move File from "/Pending/{...}" to "/Rejected/{...}" (same deep path)
            SRV->>FS: Delete Firestore exam record
            APP-->>U: Notification: "Submission rejected. Check guidelines."
        end
    end
```

---

## 2. Secure Download & Statistics Flow

All files are delivered as encrypted in-app cache blobs. Download counts are tracked for all users.

```mermaid
sequenceDiagram
    autonumber
    actor U as Registered User
    participant APP as Flutter App
    participant SRV as Server Boundary (Functions)
    participant FS as Firestore
    participant GD as Google Drive

    U->>APP: Taps "Download"

    APP->>FS: Fetch User Profile (displayName, roles, downloadCount)

    alt Standard User AND downloadCount >= 10
        APP-->>U: Show Closable Limit Modal ("Upload Now" / "Upload Later")
    else Access Allowed (Contr/Mod/Prof/Admin OR Standard < 10)
        APP->>SRV: Secure Download Request (Exam ID)
        SRV->>FS: Verify Access Permissions
        SRV->>GD: Fetch File Buffer
        GD-->>SRV: raw blob
        SRV->>SRV: Encrypt blob with user-specific keys (optional)
        SRV-->>APP: encrypted stream

        APP->>APP: Save to Local Encrypted Cache (Internal Storage)
        APP->>FS: Increment user.downloadCount (Statistic)

        APP-->>U: Open in-app PDF Viewer
    end
```

---

## 3. Account Initialization (Auth + Profile)

```mermaid
sequenceDiagram
    autonumber
    actor U as New Student
    participant AUTH as Firebase Auth
    participant SRV as Server Trigger
    participant FS as Firestore

    U->>AUTH: Register (Email/Pass or Google)
    AUTH-->>U: Account Created (UID)

    SRV->>FS: Create user doc (displayName, isModerator: false, isAdmin: false, downloadCount: 0)
    Note over FS: profile: { id: UID, email: email, isContributor: false, ... }

    U->>AUTH: Login
    AUTH-->>U: Token issued
    U->>FS: Sync Profile
```
