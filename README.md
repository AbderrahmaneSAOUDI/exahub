# ExaHub

ExaHub is a free, community-powered Flutter app for organizing and sharing past university exam papers in one searchable place. It is designed as an archive for students who need fast access to previous exams, filtered by university, department, module, year, semester, and exam type.

## Core Features

- **Browse & Filter**: Find approved exam papers effortlessly.
- **Fair Access Model**: Standard users get 10 downloads. Uploading a single approved exam unlocks unlimited contributor access.
- **Secure File Delivery**: Exams are stored in an encrypted in-app cache to mitigate organic piracy.
- **Uploads & Moderation**: Users can upload new papers (PDF, PNG, JPG) to a pending queue for review by student moderators.

## Tech Stack & Architecture

ExaHub follows a **Feature-First Architecture** inside the Flutter codebase. 

- **Frontend**: Flutter (Mobile + Web)
- **State Management**: Riverpod (`@riverpod` generation)
- **Authentication**: Firebase Auth (Google & Email/Password)
- **Database**: Cloud Firestore (Aggressively cached locally)
- **File Storage & Delivery**: Google Drive API (For zero-cost high-volume file delivery)

## Getting Started

1. **Install Dependencies**
   ```bash
   flutter pub get
   ```

2. **Configure Firebase**  
   We have Firebase configured for Web and Android. Ensure you have the FlutterFire CLI installed:
   ```bash
   dart pub global activate flutterfire_cli
   ~/.pub-cache/bin/flutterfire configure --project=exahub-firebase
   ```

3. **Run the App**
   ```bash
   flutter run -d chrome  # Run on web
   # OR
   flutter run            # Run on available Android device/emulator
   ```

## Project Status & Documentation

This repository is the main Flutter app for ExaHub. 
- The **source of truth** for the overarching vision and rules is `exahub_description.md`.
- Tactical specs, architecture blueprints, data models, and access guard logic are isolated within the `project-docs/` folder.

## License

See the [LICENSE](LICENSE) file for details.
