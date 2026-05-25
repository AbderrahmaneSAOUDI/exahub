# Authentication & Access Control System

## 1. User States & Privileges
The Exam Archive implements a simple but powerful dual-state access model. There are no tier lists, no premium subscriptions, and no financial transactions. Access is governed purely by community contribution.

### Standard User
* **Method of Entry**: Register a new account via email/password.
* **Privileges**:
  * Browse the complete directory of approved exam papers.
  * Apply filters to search metadata.
  * **Download limit**: Up to **10 total files** in their lifetime.
  * **Download style**: Online streaming preview only. Files are not saved to the device's storage.
* **Upgrade Path**: Submit an exam paper that successfully passes moderation. Once approved, the user is permanently promoted to Contributor.

### Contributor User
* **Method of Entry**: Have at least one (1) uploaded exam approved by a moderator.
* **Privileges**:
  * Browse the complete directory of approved exam papers.
  * **Download limit**: **Unlimited** downloads.
  * **Download style**: Encrypted local download. Files are saved securely in app-exclusive storage (invisible in standard gallery or system file managers) for robust offline viewing.
* **Revocation Policy**: **Permanent status**. Once a user becomes a Contributor, they remain a Contributor forever.

---

## 2. Access States Summary

| Feature | Standard User | Contributor User |
|---|---|---|
| **Limit Cap** | 10 Downloads | Unlimited Downloads |
| **Download Type** | Web/API Online Streaming Only | Encrypted Local / Offline File Cache |
| **Cost** | Free (Email sign up) | Free (1 Approved Contribution) |
| **Duration** | Temporary (Until 10 downloads) | Permanent |

---

## 3. The 10-Download Guard Logic
The app maintains a strict client-side and server-side verification pattern to ensure download limits are respected.

### Guard Pseudocode / Application Logic

```javascript
async function handleExamDownloadRequest(user, exam) {
  // 1. Fetch current user data from Firestore cache/source
  const userDoc = await firestore.collection('users').doc(user.uid).get();
  const userData = userDoc.data();

  // 2. Check if user is a Contributor
  if (userData.isContributor === true) {
    // Unlimited Access: Proceed with offline encrypted file download
    return proceedWithEncryptedDownload(exam);
  }

  // 3. If standard user, verify the download count
  if (userData.downloadCount >= 10) {
    // Guard triggered: Reject download and prompt contribution
    showLimitReachedModal();
    return;
  }

  // 4. If count is < 10, proceed with streaming download
  const downloadSuccess = await proceedWithOnlineStream(exam);
  
  if (downloadSuccess) {
    // 5. Increment count in both Firestore and local state
    await firestore.collection('users').doc(user.uid).update({
      downloadCount: admin.firestore.FieldValue.increment(1)
    });
  }
}

function showLimitReachedModal() {
  showModal({
    title: "Limit Reached",
    body: "You've reached your 10-file limit. Upload one approved exam to unlock unlimited downloads forever.",
    actionButtonText: "Upload Now",
    actionRoute: "/upload"
  });
}
```

### UX Design Pattern on Cap Trigger
When the Standard user reaches their 11th download attempt, the app blocks the file delivery and presents a dialog stating:
> **"You've reached your 10-file limit. Upload one approved exam to unlock unlimited downloads."**

This dialog provides a primary CTA button that navigates directly to the Upload Screen.

---

## 4. Anti-Piracy & Integrity Safeguards

### Watermarking
Every single PDF paper served by the platform undergoes server-side/service-layer watermarking using a PDF manipulation library before it is streamed or cached locally. 
- The watermark prominently stamps the **Exam Archive** logo and support text along the margins of each page.
- **Strategic Value**: If a student circumvents local device locks and shares a PDF manually over WhatsApp or Telegram, it acts as organic promotion for the platform.

### Encrypted Local Downloads
For Contributors, downloaded exams are kept secure using AES encryption or Flutter's secure storage keys (`flutter_secure_storage` combined with directory isolation). 
- Files are stored in the application's private documents directory.
- Files do not show up in the Android/iOS built-in download history or photo gallery, preventing easy bulk forwarding.

### Credential Protection
Since the contributor status is tied directly to the registered email account ID and its specific upload records, users have a strong incentive not to share credentials (as doing so risks their account lockout or credentials resetting).
