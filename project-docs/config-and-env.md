# Configuration & Environment Variables

## 1. Firebase Project Setup
The Exam Archive uses a Firebase project to handle user authentication, store document metadata via Firestore, and trigger push notifications.

### Web/Mobile Configuration Details
Since the app targets a Flutter Web & Mobile tech stack, both standard native and web configurations are initialized.

#### Web Firebase Configuration Object
```javascript
const firebaseConfig = {
  apiKey: "AIzaSyA1-YOUR_WEB_API_KEY_HERE",
  authDomain: "exam-archive-ghardaia.firebaseapp.com",
  projectId: "exam-archive-ghardaia",
  storageBucket: "exam-archive-ghardaia.appspot.com",
  messagingSenderId: "SENDER_ID_HERE",
  appId: "1:SENDER_ID:web:APP_ID_HERE",
  measurementId: "G-MEASUREMENT_ID_HERE"
};
```

---

## 2. Google Drive API Service Integration
To bypass Firebase Storage free tier limits, a dedicated Google Cloud service account or secure REST client interacts with Google Drive.

### Required Permissions & OAuth Scopes
- **Scope**: `https://www.googleapis.com/auth/drive.file` (Restrictive scope to only edit and access files created or uploaded by the app).
- **Alternative (Admin Moderator Sync)**: `https://www.googleapis.com/auth/drive` (Used only under service account context in the backend review function to transfer files between the `/Pending` and `/Approved` folders).

### Service Account Credentials (`service-account.json`)
The application server/moderator backend utilizes a Google Cloud Service Account to move files securely.
```json
{
  "type": "service_account",
  "project_id": "exam-archive-ghardaia",
  "private_key_id": "xxxxxx_YOUR_PRIVATE_KEY_ID_xxxxxx",
  "private_key": "-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY_CONTENTS_HERE\n-----END PRIVATE KEY-----\n",
  "client_email": "drive-uploader@exam-archive-ghardaia.iam.gserviceaccount.com",
  "client_id": "12345678901234567890",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/drive-uploader%40exam-archive-ghardaia.iam.gserviceaccount.com"
}
```

---

## 3. Local Environment Variables
A `.env` file must be created at the root of the project directory. The Flutter build loads these values at compile-time using `--dart-define` or package integrations like `flutter_dotenv`.

### Local Development Environment Layout (`.env`)
```bash
# Firebase Credentials
FIREBASE_API_KEY=AIzaSyA1-YOUR_WEB_API_KEY_HERE
FIREBASE_AUTH_DOMAIN=exam-archive-ghardaia.firebaseapp.com
FIREBASE_PROJECT_ID=exam-archive-ghardaia
FIREBASE_STORAGE_BUCKET=exam-archive-ghardaia.appspot.com
FIREBASE_MESSAGING_SENDER_ID=SENDER_ID_HERE
FIREBASE_APP_ID=1:SENDER_ID:web:APP_ID_HERE

# Google Drive Storage Config
GOOGLE_DRIVE_PENDING_FOLDER_ID=1A2B3C4D5E6F7G8H9I0J1K2L3M4N5O6P_PENDING
GOOGLE_DRIVE_APPROVED_FOLDER_ID=1A2B3C4D5E6F7G8H9I0J1K2L3M4N5O6P_APPROVED

# App Details
APP_VERSION=1.0.0
APP_ENVIRONMENT=development
```

---

## 4. Initialization Configuration Checklist
- [ ] Create Google Cloud Platform (GCP) Console project.
- [ ] Enable the **Google Drive API** in the API library.
- [ ] Create a **Service Account** with access to the target Google Drive folders.
- [ ] Share Google Drive folder `Pending` and `Approved` with the service account email: `drive-uploader@exam-archive-ghardaia.iam.gserviceaccount.com` (granting Editor role).
- [ ] Build the Firebase project and initialize email/password auth under the Firebase Console.
- [ ] Register Android, iOS, and Web apps inside the Firebase console.
- [ ] Deploy basic Firestore security rules to secure read/writes.
