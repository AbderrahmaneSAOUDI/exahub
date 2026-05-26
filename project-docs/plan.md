# ExaHub — Tactical Execution Plan

> **Document Version:** 2.0 — Greenfield Build
> **Source of Truth:** `overview.md`, `data-integrations.md`, `auth-system.md`, `config-and-env.md`, `incomplete-features.md`
> **Last Updated:** May 2026

---

## 0. Triage Method & Theme Map

All five project-docs were cross-analyzed to extract functional themes and trace cause → effect chains. Since this is a greenfield project (no legacy bugs), the plan focuses on **building the foundation correctly the first time**.

### Theme Mapping

| Theme | Source Docs | Cause → Effect Chain |
|---|---|---|
| **Auth & Access Control** | `auth-system.md`, `overview.md` | Standard vs. Contributor state controls every browse, download, and moderation decision. If the access guard is client-only, it's bypassable. Must be authoritative server-side. |
| **Metadata Caching** | `data-integrations.md`, `overview.md` | App targets 2GB RAM / 3G devices. Blank screens on network lag destroy trust. Firestore cache must load-first, sync-second. |
| **Drive Delivery Boundary** | `config-and-env.md` | Google Drive is the source. Watermark ONCE on upload. A server boundary is mandatory for web. |
| **Upload & Moderation** | `data-integrations.md` | Folders structured `/Status/Univ/Spec/Type/file`. Pro/Mod/Admin uploads auto-approve. |
| **Unified Encryption** | `auth-system.md` | ALL files (streamed or saved) are encrypted blobs in internal storage. |
| **Scope Control** | `incomplete-features.md` | 10 features are explicitly deferred. The architecture must not accidentally block them, but must not build toward them either. |

---

## 1. Architecture Foundations

### 1.1 Project Structure (Feature-First Architecture) `#infra #logic`

The Flutter project should follow a **feature-first** folder structure that scales with the MVP screens and keeps domain logic separated from UI.

```
lib/
├── main.dart                          # App entry point, Firebase init
├── app/
│   ├── app.dart                       # MaterialApp/Router shell
│   ├── router.dart                    # Route definitions + guards
│   └── theme.dart                     # App-wide theme configuration
├── core/
│   ├── constants/
│   │   ├── app_constants.dart         # Download limits, app name, etc.
│   │   └── firestore_paths.dart       # Collection/field name constants
│   ├── errors/
│   │   └── failures.dart              # Typed failure classes
│   ├── services/
│   │   ├── drive_service.dart         # Google Drive API abstraction
│   │   ├── watermark_service.dart     # PDF watermarking logic
│   │   └── notification_service.dart  # FCM wrapper
│   ├── storage/
│   │   ├── secure_storage.dart        # Platform-adaptive encrypted storage interface
│   │   ├── secure_storage_mobile.dart # Mobile implementation (file system + AES)
│   │   └── secure_storage_web.dart    # Web implementation (IndexedDB / OPFS)
│   └── utils/
│       └── validators.dart            # Form validators, PDF checks
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   └── auth_repository.dart   # Firebase Auth + Firestore user doc
│   │   ├── domain/
│   │   │   ├── user_model.dart        # User data model
│   │   │   └── auth_state.dart        # Auth state enum/sealed class
│   │   └── presentation/
│   │       ├── auth_provider.dart     # Auth state management
│   │       ├── login_screen.dart
│   │       └── register_screen.dart
│   ├── browse/
│   │   ├── data/
│   │   │   └── exam_repository.dart   # Firestore exam queries + caching
│   │   ├── domain/
│   │   │   ├── exam_model.dart        # Exam data model
│   │   │   └── filter_state.dart      # Active filter selections
│   │   └── presentation/
│   │       ├── browse_provider.dart   # Browse state management
│   │       ├── browse_screen.dart     # Exam list + filters
│   │       └── exam_detail_screen.dart
│   ├── download/
│   │   ├── data/
│   │   │   └── download_repository.dart  # Download logic + quota check
│   │   ├── domain/
│   │   │   └── download_state.dart       # Download progress/result
│   │   └── presentation/
│   │       ├── download_provider.dart
│   │       └── limit_reached_modal.dart
│   ├── upload/
│   │   ├── data/
│   │   │   └── upload_repository.dart # Upload to Drive + Firestore record
│   │   ├── domain/
│   │   │   └── upload_state.dart      # Upload progress/result
│   │   └── presentation/
│   │       ├── upload_provider.dart
│   │       └── upload_screen.dart
│   └── moderation/
│       ├── data/
│       │   └── moderation_repository.dart  # Pending exams + approve/reject
│       ├── domain/
│       │   └── moderation_state.dart
│       └── presentation/
│           ├── moderation_provider.dart
│           └── moderation_screen.dart
└── shared/
    ├── widgets/
    │   ├── exam_card.dart             # Reusable exam display card
    │   ├── filter_bar.dart            # Filter dropdowns row
    │   ├── loading_skeleton.dart      # Skeleton loading placeholder
    │   └── empty_state.dart           # No-data / no-results view
    └── extensions/
        └── context_extensions.dart    # Theme, navigation shortcuts
```

### 1.2 Why Feature-First

- Each feature owns its data, domain, and presentation layers — no cross-feature imports at the data level.
- `core/` holds shared infrastructure that any feature can depend on.
- `shared/` holds reusable UI widgets that don't belong to a specific feature.
- Scales naturally: adding gamification later means adding `features/gamification/` — no restructuring needed.

---

## 2. State Management Design `#logic #auth #data`

### 2.1 Recommended Approach: Riverpod

**Riverpod** is the recommended state management solution for ExaHub for these reasons:

| Factor | Why Riverpod |
|---|---|
| Compile-time safety | No runtime `ProviderNotFoundException` — catches missing providers at build time |
| Testability | Providers are testable in isolation with `ProviderContainer` |
| Scoping | Feature providers stay scoped; no global god-object state |
| Async support | Native `AsyncValue<T>` maps perfectly to loading/error/data states |
| Firestore integration | `StreamProvider` wraps Firestore snapshots cleanly |
| Code generation | `@riverpod` annotation reduces boilerplate |

### 2.2 Provider Architecture

```
                    ┌─────────────────────────────────────────┐
                    │           App Router / Shell             │
                    │  (reads authStateProvider to decide      │
                    │   initial route + route guards)          │
                    └───────────┬─────────────────────────────┘
                                │
                    ┌───────────▼───────────────────┐
                    │      authStateProvider         │
                    │  (StreamProvider<AuthState>)    │
                    │  Emits: anonymous | standard   │
                    │         | contributor | mod     │
                    └───────────┬───────────────────┘
                                │
              ┌─────────────────┼─────────────────────┐
              │                 │                      │
   ┌──────────▼──────┐  ┌──────▼──────────┐  ┌───────▼─────────┐
   │ userProvider     │  │ examListProvider│  │ moderationProvider│
   │ (reads user doc) │  │ (Firestore     │  │ (pending exams)   │
   │                  │  │  with cache)   │  │                   │
   └──────────┬───────┘  └──────┬─────────┘  └───────┬──────────┘
              │                 │                      │
   ┌──────────▼──────┐  ┌──────▼─────────┐  ┌────────▼─────────┐
   │downloadGuard    │  │filterState     │  │approveExam()     │
   │Provider         │  │Provider        │  │rejectExam()      │
   │(checks quota)   │  │(active filters)│  │(server actions)  │
   └─────────────────┘  └────────────────┘  └──────────────────┘
```

### 2.3 Core Providers

| Provider | Type | Source | Consumers |
|---|---|---|---|
| `authStateProvider` | `StreamProvider<AuthState>` | Firebase Auth stream + Firestore user doc | Router, all screens |
| `currentUserProvider` | `Provider<UserModel?>` | Derived from `authStateProvider` | Download guard, upload, profile |
| `examListProvider` | `StreamProvider<List<ExamModel>>` | Firestore `exams` query (approvalStatus == true) | Browse screen |
| `filterStateProvider` | `StateNotifierProvider<FilterState>` | Local UI state | Browse screen filter bar |
| `filteredExamsProvider` | `Provider<List<ExamModel>>` | Derived: applies filters to `examListProvider` | Browse screen list |
| `downloadGuardProvider` | `Provider<DownloadGuard>` | Derived from `currentUserProvider` | Exam detail download button |
| `uploadStateProvider` | `StateNotifierProvider<UploadState>` | Upload form + Drive upload progress | Upload screen |
| `pendingExamsProvider` | `StreamProvider<List<ExamModel>>` | Firestore `exams` (approvalStatus == false) | Moderation screen |

### 2.4 Auth State Model

```dart
sealed class AuthState {
  const AuthState();
}

class AuthAnonymous extends AuthState {
  const AuthAnonymous();
}

class AuthAuthenticated extends AuthState {
  final UserModel user;
  const AuthAuthenticated(this.user);
}

// UserModel carries the role information
class UserModel {
- **isModerator: true/false** (Managed by admin)
- **isAdmin: true/false** (Managing moderators and app settings)
- **isProfessor: true/false** (Can upload auto-approved files)
- **displayName: user_name** (Used for scoring and profile)
- **downloadCount: int** (Statistic tracked for all users)
```

### 2.5 Download Guard Model

```dart
sealed class DownloadPermission {
  const DownloadPermission();
}

class DownloadAllowed extends DownloadPermission {
  final DownloadType type; // streaming | encryptedLocal
  const DownloadAllowed(this.type);
}

class DownloadBlocked extends DownloadPermission {
  final String message;
  final int downloadsUsed;
  const DownloadBlocked(this.message, this.downloadsUsed);
}

class DownloadRequiresAuth extends DownloadPermission {
  const DownloadRequiresAuth();
}
```

---

## 3. Routing & Navigation Guards `#auth #ui #logic`

### 3.1 Router Design (go_router)

```dart
// Route structure
/                    → Redirect based on auth state
/login               → Login screen (anonymous only)
/register            → Register screen (anonymous only)
/browse              → Browse screen (authenticated)
/browse/:examId      → Exam detail screen (authenticated)
/upload              → Upload screen (authenticated)
/moderation          → Moderation panel (moderator only)
```

### 3.2 Route Guard Logic

```
On every navigation:
    │
    ├─ authState == anonymous?
    │   ├─ Target is /login or /register → ALLOW
    │   └─ Any other target → REDIRECT to /login
    │
    ├─ authState == authenticated?
    │   ├─ Target is /login or /register → REDIRECT to /browse
    │   ├─ Target is /moderation → CHECK isModerator
    │   │   ├─ true → ALLOW
    │   │   └─ false → REDIRECT to /browse
    │   └─ Any other authenticated route → ALLOW
    │
    └─ authState == loading?
        └─ Show splash/loading screen
```

### 3.3 Mid-Session Role Transition

When a user's `isContributor` field changes (via Firestore listener):

1. The `authStateProvider` re-emits with updated `UserModel`.
2. The download guard re-evaluates — limit disappears.
3. Download buttons on visible screens update reactively.
4. **No app restart or re-login required.**

---

## 4. Web-Specific Storage Integration `#web #media #infra`

### 4.1 The Platform Problem

| Capability | Mobile (Android/iOS) | Web (Browser) |
|---|---|---|
| Private cache storage | ✅ App internal cache sandbox | ❌ No direct sandboxed filesystem |
| Encrypted storage | ✅ AES encrypted blob in app cache | ⚠️ IndexedDB or OPFS |
| File invisible to user | ✅ Exists only in app memory/cache | ⚠️ Only in-app database cache |
| Persistent across sessions | ✅ Survives app restarts | ⚠️ Browser can clear storage |
| Firestore offline cache | ✅ Built-in persistence | ⚠️ `enableMultiTabIndexedDbPersistence()` |

### 4.2 Platform-Adaptive Storage Interface

```dart
// Abstract interface — implementations per platform
abstract class SecureFileStorage {
  /// Save an encrypted PDF for offline access
  Future<void> saveFile(String examId, Uint8List pdfBytes);

  /// Retrieve a previously saved PDF
  Future<Uint8List?> getFile(String examId);

  /// Check if a file is available offline
  Future<bool> hasFile(String examId);

  /// Delete a specific cached file
  Future<void> deleteFile(String examId);

  /// Get total storage used
  Future<int> getStorageUsedBytes();
}
```

### 4.3 Mobile Implementation Strategy

```
PDF bytes from server
    → AES-256 encrypt with device-bound key (flutter_secure_storage)
    → Write to app internal cache/sandbox storage as encrypted blob
    → File is only accessible inside the app (never exists as extractable file in file manager)
    → On read: decrypt in memory → render in in-app PDF viewer
    → On app cache clearing: removed automatically / on uninstall
```

### 4.4 Web Implementation Strategy

```
PDF bytes from server
    → AES-256 encrypt with key stored in IndexedDB (via web crypto API)
    → Store encrypted blob in IndexedDB or Origin Private File System (OPFS)
    → File never appears in browser downloads
    → On read: decrypt in memory → render in PDF viewer widget
    → Graceful fallback: if storage unavailable, stream-only mode
```

### 4.5 Web Fallback Behavior

If the browser cannot provide persistent private storage (e.g., incognito mode, restrictive browser, storage quota exceeded):

1. Contributor users get **streaming access** instead of encrypted local access.
2. A subtle banner informs: *"Offline viewing is not available in this browser. Files will be streamed instead."*
3. No functionality is lost — only the offline caching benefit.

### 4.6 Drive API on Web — CORS & Security

| Concern | Solution |
|---|---|
| Service account in browser | **Never.** All Drive operations go through a server boundary. |
| CORS on file downloads | Server boundary (Cloud Function) proxies the file with correct CORS headers. |
| Signed URLs | Server generates short-lived signed URLs; client fetches directly from Drive with the signed URL. |
| Range requests | Server proxy must support `Range` headers for PDF progressive loading. |

---

## 5. Server Boundary Design `#api #infra #web`

### 5.1 Why a Server Boundary Is Required

The spec requires Google Drive API operations that need service account credentials. These credentials **cannot exist in the Flutter web bundle**. A server boundary is mandatory.

### 5.2 Recommended: Firebase Cloud Functions

| Function | Trigger | Purpose |
|---|---|---|
| `uploadExam` | HTTPS callable | Receives PDF/image + metadata → watermarks once during upload processing → stores in deep path under `/Pending` or `/Approved` |
| `approveExam` | HTTPS callable (mod/admin) | Moves file from `/Pending/{...}` to `/Approved/{...}` → updates Firestore atomically → sends notification |
| `rejectExam` | HTTPS callable (mod/admin) | Moves file from `/Pending/{...}` to `/Rejected/{...}` → deletes Firestore record → sends notification |
| `getDownloadUrl` | HTTPS callable | Verifies user quota/role → generates signed URL or proxies encrypted file stream |
| `onUserCreate` | Auth trigger | Creates Firestore user document with default Standard values |

### 5.3 Function Security

```
Every callable function:
    → Verify Firebase ID token (automatic with callable)
    → Check custom claims for moderator-only functions
    → Validate request parameters
    → Execute Drive/Firestore operations with Admin SDK
    → Return result to client
```

### 5.4 Alternative: Cloud Run

If Cloud Functions cold-start latency is unacceptable for file downloads:
- Deploy a lightweight Cloud Run service for file proxying.
- Cloud Functions handle metadata operations (uploads, approvals).
- Cloud Run handles streaming delivery with watermarking.

---

## 6. UX Gaps to Close Before Launch `#ui`

| Gap | Expected Behavior |
|---|---|
| Cached browse results | Show immediately on app open, then sync in background. Never a blank screen. |
| Empty states | Dedicated views for: no exams found, no filter matches, no internet, first-time empty cache. |
| Limit-reached flow | Feels like a next step, not a dead end. Clear "Upload Now" CTA. Shows remaining count before hitting limit. |
| Upload progress | Multi-step indicator: form validation → file uploading → "Under review" confirmation. |
| Moderation states | Clear visual distinction for pending, approved, and rejected exams in mod panel. |
| Offline indicator | Subtle badge when operating on cached data. Not alarming. |
| Download type indicator | User understands all files are encrypted in-app and counters continue tracking for statistics. |
| Web download errors | Readable messages for CORS failures, auth errors, and storage quota issues on web. |

---

## 7. Backend & Data Layer Action Items `#data #api #infra`

| # | Item | Tags | Priority |
|---|---|---|---|
| 1 | Define Firestore security rules (baseline in `data-integrations.md`) | `#data #auth` | Critical |
| 2 | Create Cloud Functions for all Drive operations | `#api #infra` | Critical |
| 3 | Implement atomic approval flow (Drive move + Firestore update + user update) | `#data #api` | Critical |
| 4 | Split environment variables into client-safe and server-only | `#infra` | Critical |
| 5 | Set up composite Firestore indexes for filter queries | `#data` | High |
| 6 | Implement download count increment as atomic server-side operation | `#data #auth` | High |
| 7 | Configure FCM for upload approval/rejection notifications | `#infra` | Medium |
| 8 | Design seed data import process for initial 200 exams | `#data` | Medium |
| 9 | Set up watermarking pipeline in server function | `#media #api` | Medium |

---

## 8. Refactor Opportunities `#logic #infra`

Since this is greenfield, "refactor" means "build correctly from the start":

| # | Principle | Application |
|---|---|---|
| 1 | Split `main.dart` immediately | Bootstrap, shell, routing, and feature screens in separate files from day one. |
| 2 | Repository pattern for all data access | `ExamRepository`, `AuthRepository`, `DownloadRepository` — UI never talks to Firestore directly. |
| 3 | Platform-adaptive storage | Define interface first, implement per-platform. Don't hardcode mobile assumptions. |
| 4 | Externalize all strings | Even in single-language MVP, use constants or `.arb` files. No hardcoded strings in widgets. |
| 5 | Thin providers | Providers orchestrate; they don't contain business logic. Business rules live in repositories and services. |

---

## 9. MVP Readiness Checklist

### 9.1 Core Functionality

- [ ] App boots cleanly from fresh install and initializes Firebase
- [ ] Email registration creates account + Firestore user document
- [ ] Email login restores session + loads user state from cache
- [ ] Browse screen shows approved exams from cached Firestore data
- [ ] Filters (university, department, module, year, semester, type) work correctly
- [ ] Total exam count displayed as social proof
- [ ] Exam detail screen shows all metadata fields
- [ ] Standard users can download up to 10 files (encrypted in-app)
- [ ] Download count increments atomically after successful delivery
- [ ] 11th download attempt shows limit modal with upload CTA
- [ ] Contributors/Professor/Moderator/Admin can download unlimited files to encrypted in-app cache
- [ ] Upload form validates all mandatory fields + supports PDF/JPG/PNG
- [ ] Upload submits to deep path under `/Pending/{Country}/{University}/{Speciality}/{ExamType}` + creates Firestore record
- [ ] Moderation panel shows pending submissions (mod accounts only)
- [ ] Approve flow: moves file from `/Pending/{...}` to `/Approved/{...}`, updates exam, unlocks contributor, sends notification
- [ ] Reject flow: moves file from `/Pending/{...}` to `/Rejected/{...}`, deletes Firestore record, sends notification
- [ ] Watermark applied once during upload processing
- [ ] Push notifications work for approval and rejection

### 9.2 Platform Quality

- [ ] Web: no service account credentials in client bundle
- [ ] Web: Drive downloads work through server boundary with correct CORS
- [ ] Web: encrypted storage fallback works (stream-only if storage unavailable)
- [ ] Mobile: encrypted files exist only inside the app cache (no standalone PDFs on the device)
- [ ] Offline: cached metadata displays immediately, background sync reconciles
- [ ] Offline: download count syncs correctly after reconnection
- [ ] Error states: offline, stale cache, Drive failure, storage failure all handled

### 9.3 Testing

- [ ] Auth state transitions tested (anonymous → standard → contributor)
- [ ] Download guard tested (under limit, at limit, over limit, contributor bypass)
- [ ] Cache refresh tested (fresh, stale, offline, error states)
- [ ] Route guards tested (anonymous redirect, mod-only protection)
- [ ] Upload form validation tested (all fields required, PDF/JPG/PNG)

---

## 10. Open Decisions Requiring Owner Input

| # | Question | Options | Impact |
|---|---|---|---|
| 1 | **Server boundary technology** | Cloud Functions (simplest) vs. Cloud Run (better for file streaming) vs. hybrid | Affects cold-start latency, file delivery speed, and deployment complexity |
| 2 | **Web encrypted storage target** | IndexedDB (widely supported) vs. OPFS (newer, better for files) vs. streaming-only on web | Affects Contributor experience on web |
| 3 | **Download count increment timing** | After completed transfer (safest) vs. on stream open (simpler) | Affects edge cases where transfer fails mid-stream |
| 4 | **Firestore cache freshness policy** | Time-based TTL vs. version-based vs. Firestore real-time listeners | Affects read costs and data freshness |
| 5 | **Moderator role mechanism** | Firebase custom claims (more secure) vs. `moderators` Firestore collection (simpler to manage) | Affects admin onboarding workflow |
| 6 | **PDF watermarking location** | Server-side (in Cloud Function) vs. client-side (before display) | Affects server load vs. client complexity |
| 7 | **Seed data import method** | Admin script (one-time) vs. admin panel bulk upload vs. direct Firestore import | Affects pre-launch preparation workflow |

---

## 11. Execution Order

### Sprint Sequence

| Sprint | Focus | Deliverable |
|---|---|---|
| **Sprint 1** | Firebase setup + Auth + User model | Working login/register → Firestore user document → auth state provider |
| **Sprint 2** | Browse + Exam model + Caching | Firestore exam queries → cached browse screen with filters |
| **Sprint 3** | Download system + Guard | Download pipeline → 10-file guard → limit modal → streaming delivery |
| **Sprint 4** | Upload flow | 9-field form → Drive upload → Firestore record → pending state |
| **Sprint 5** | Moderation panel | Mod-only access → pending list → approve/reject → atomic state updates |
| **Sprint 6** | Server boundary + Watermark | Cloud Functions → secure Drive delivery → PDF watermarking |
| **Sprint 7** | Encrypted storage + Web | Platform-adaptive storage → web fallback → CORS handling |
| **Sprint 8** | Polish + Notifications + Testing | Push notifications → empty states → error handling → test coverage |
| **Sprint 9** | Seed data + Pre-launch | Import 200 exams → final QA → distribution preparation |

---

*This plan is the engineering launch map. Every build decision traces back to a documented requirement. No feature is built that isn't in the MVP scope. No deferred feature is accidentally blocked.*