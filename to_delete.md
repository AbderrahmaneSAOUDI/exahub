`| **Watermarking** | App name stamped on every PDF before delivery | Leaked files become organic advertising |` --> I think file should watermarked in uploading, do once for everytime else.

`| **Account integrity** | Contributor status tied to upload history | Sharing credentials loses contributor verification |` --> remove this, no need for that.

`- Files go to Drive `/Pending` folder → Firestore record created with `approvalStatus: false`.` --> No, for better structure, it should do folders deep to put the file, like `/Pending/Ghardaia/University_name/Speciality/type/file_name`.

`Screenshot prevention` --> I cancelled that decision, so its totally okay to screenshot anywhere anytime.

`- Flutter version updates (~every 6 months)` --> update when needed not 6 months.

`Dean endorsement, word-of-mouth within known community.` --> forgot totally this point, unneeded, I will take care of it myself, never mention it again.

`> WhatsApp-first because it is the primary discovery mechanism for Algerian students.` --> Telegram not WhatsApp.

`### 1.2 `users` Collection` --> Add `isModerator` for users that can approve/reject submissions, managed by admin.

`Moderator approves → isContributor: true, uploadCount++
    │       │                        (downloadCount becomes irrelevant)` --> for this I think its better to make to keep increasing downloadCount but stay hidden after one upload approved. we need that statistic later.

`### 2.1 Folder Structure` --> Update this also as I said before. Pending/Approved/Rejected all have the same subfolders structure.

`### 2.3 Upload & Review Pipeline` --> Add rejection part, and then, create a separate file for sequence diagrams, put in it all possible diagrams.

`| File format | PDF only — reject all others at file picker |` --> Accept also pictures, almost of students will scan their exams using camera so we have to accept pictures.

`| Method | Email + Password |` --> Add the name, I need it later in the score system.

`2. User States & Privileges` -->
* Add a Professor role who can upload files and auto-approved.
* Add the possibility to upload files by moderators and make them auto-approved. (keep the download/upload counters increase anyways, shown in profile).
* Add admin role who have full access on the app + promoting users to moderators.

`4.1 Guard Logic -> If isContributor → serve file → no count change` --> the download/upload counters increase anyways.

`The modal is not dismissable by tapping outside — user must choose "Upload Now" or "Close".` --> I think its better to make it closable. Also, I need to change "Close" to something like "Upload later" or something.

`The download button on exam cards shows a lock icon and "Upload to unlock" tooltip when limit is reached.` --> No, keep the button and show the popup each click on download for Standard users, the popup should have two options "Upload Now" and "Upload later".

`6.2 Encrypted In-App Cache (Contributor Only)` --> Not contributors only, I will let Standard users also download streaming files, but encrypted. so remove this part from the md and keep all files encrypted everywhere. And also update the file download sequence diagram.

`- [ ] Disable all other providers (Google, Facebook, Phone, Anonymous)` --> Keep only Google and Email/Password.

update them everywhre.
