# AI Story Studio (Android / Kotlin + Jetpack Compose)

This repository is an **Android native Kotlin** project, not a Flutter project. It includes a Cloud Functions backend under `functions/`.

## Before building

1. Create/select a Firebase project.
2. Add an Android app whose package name is `com.example.aistorystudio`.
3. Enable **Authentication** (Google provider; configure the SHA-1/SHA-256 fingerprints for your signing setup), **Cloud Firestore**, and **Cloud Storage**.
4. Download that app's `google-services.json` and place it at `app/google-services.json`. Do not use a config file from a different Firebase project.
5. In Firebase Console, configure the OAuth web client for Google Sign-In. The generated `default_web_client_id` resource must be present.
6. Install Node.js 20 and Firebase CLI for backend deployment.
7. Set the OpenAI secret for Cloud Functions:

   ```bash
   firebase functions:secrets:set OPENAI_API_KEY
   ```

   Enter the key interactively. Never place the OpenAI API key in Android resources, source code, or the APK.

## Build an APK from a phone with GitHub Actions

A cloud workflow is included at `.github/workflows/android-apk.yml`. It builds a **debug APK for testing** and uploads it as an Actions artifact.

1. Create a private GitHub repository and upload the extracted project files (including the hidden `.github` folder).
2. In Firebase Console, download the matching `google-services.json` for package `com.example.aistorystudio`.
3. Convert that file to Base64 using a trusted local method. On a device with Termux, for example: `base64 -w 0 google-services.json`.
4. In GitHub, open **Settings → Secrets and variables → Actions → New repository secret**. Name it `FIREBASE_GOOGLE_SERVICES_JSON_BASE64` and set its value to the Base64 text.
5. Open the **Actions** tab, select **Build Android APK**, and tap **Run workflow**.
6. When the workflow succeeds, open that run and download the `ai-story-studio-debug-apk` artifact. Extract the artifact ZIP to get `app-debug.apk`.

If the workflow fails, open the run log and fix the first reported error before rerunning. Do not paste private keys or service-account credentials into ChatGPT or source files.

For a local build, the Gradle Wrapper was not included in the supplied archive. Generate it from a trusted Gradle installation compatible with the Android Gradle Plugin configured in `gradle/libs.versions.toml`:

```bash
gradle wrapper --gradle-version 9.3.1
./gradlew assembleDebug
```

Expected debug APK: `app/build/outputs/apk/debug/app-debug.apk`.

## Deploy backend and rules

```bash
cd functions && npm ci && cd ..
firebase deploy --only functions,firestore:rules,storage
```

Before deploying, review `firestore.rules` and `storage.rules` against your Firebase data model. Cloud Functions use the Admin SDK and server-side `OPENAI_API_KEY` secret. AI provider availability, billing, quotas, IAM, and Firebase project configuration must be verified in your own project.

## Important limitations

- This is a native Android app with Jetpack Compose; Flutter commands do not apply.
- Firebase authentication and story upload require a real `google-services.json` and enabled Firebase providers/services.
- A successful build alone does not prove that the complete AI workflow works; test sign-in, video upload, job processing, script, voice, subtitles, rendering, and download in your Firebase project.
