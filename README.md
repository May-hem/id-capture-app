
# ID Capture (Android + Desktop)
Cross-platform app to capture customer ID via device camera, perform on-device OCR with Tesseract.js, auto-detect likely ID text patterns, store locally, and export as CSV.

## Features
- Live camera preview (Android WebView + Desktop via Electron)
- Auto-capture when ID-like text is detected (regex + confidence)
- Manual capture button
- OCR on-device with Tesseract.js (no cloud keys needed)
- Local storage in IndexedDB (Dexie)
- Search & filter
- Export CSV

## Prerequisites
- Node.js 18+
- Android Studio (for Android build) with SDK platform tools
- Java 17 (for Android Gradle)
- For Desktop: Electron runs after `npm run electron`

## Install
```bash
npm install
```

## Web Dev Preview (quick test in browser)
```bash
npm run dev
# open the URL shown (allow camera in the browser)
```

## Desktop (Electron)
```bash
npm run electron
```

## Android
```bash
npm run android:init   # one-time to add Android platform
npm run android:sync   # sync web assets and permissions
npm run android:open   # opens Android Studio; build & run from there
```

### Android Notes
- Ensure camera permission is granted.
- If preview is black, check that WebView camera permission is enabled on the device or emulator.
- Auto-scan interval can be tuned in the UI (default 1500ms).

## How Auto-Detect Works
- Every interval, we run OCR on a downscaled frame for speed.
- If we see ID-like patterns (8–14 digits or `ABC-123456` style) and confidence > 50, we auto-capture and store.
- Heuristics are in `src/lib/ocr.ts` (you can customize regexes for your country IDs).

## Data Storage
- Saved locally in IndexedDB using Dexie. No server required.
- Use Export CSV to download a spreadsheet of captured IDs.

## Packaging Desktop App
```bash
npm run electron:build
# Produces installers in dist/ per electron-builder config
```

## Customizing Parsing
- Edit `parseIdMetadata` in `src/lib/ocr.ts` to adapt to country-specific ID layouts.


---
## CI: Auto-build .exe and .apk (GitHub Actions)

1) Create a new **GitHub repository** and push this project.
2) Go to **Actions** tab — GitHub will detect the workflows.
3) Click **Run workflow** on:
   - **Build Desktop (Windows .exe)** to get the Windows installer.
   - **Build Android (.apk)** to get the Android APK.
4) After each run, open the job and download from **Artifacts**:
   - `windows-installer` → contains `.exe` installer
   - `android-apk` → contains `app-debug.apk`

### Optional: Release from a tag
- Create a tag like `v1.0.0` and push:
  ```bash
  git tag v1.0.0
  git push origin v1.0.0
  ```
- The workflows will attach files to a **GitHub Release** automatically.

### Optional: Sign a release APK
- In **Repository Settings → Secrets and variables → Actions → New repository secret** add:
  - `ANDROID_KEYSTORE_BASE64` (base64 of your `.keystore` file)
  - `ANDROID_KEYSTORE_PASSWORD`
  - `ANDROID_KEY_ALIAS`
  - `ANDROID_KEY_PASSWORD`
- Then extend the Android workflow to decode and use the keystore before `./gradlew assembleRelease`.


---
## Play Store–Ready Builds (Signed AAB + Release APK)

The Android workflow builds a **signed** `app-release.aab` (preferred for Google Play) and a `app-release.apk`.
Signing credentials are read from repository **Secrets**.

### Required GitHub Secrets
- `ANDROID_KEYSTORE_BASE64` — base64 of your `release.keystore`
- `ANDROID_KEYSTORE_PASSWORD` — keystore password
- `ANDROID_KEY_ALIAS` — key alias
- `ANDROID_KEY_PASSWORD` — key alias password

### Generate a keystore (if you don't have one)
```bash
keytool -genkeypair -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
base64 my-release-key.keystore > keystore.txt
# paste keystore.txt contents into ANDROID_KEYSTORE_BASE64 secret
```

### Running the Play Store build
- Push a tag like `v1.0.0` to trigger a release build:
  ```bash
  git tag v1.0.0
  git push origin v1.0.0
  ```
- Download artifacts from the workflow **Artifacts** section, and check the **Releases** page for attached files:
  - `app-release.aab` (upload to Google Play)
  - `app-release.apk` (for sideloading)

### Google Play Console
- Upload `app-release.aab` to **Production** or **Internal testing** track.
- Make sure `applicationId` (in `capacitor.config.ts` -> `appId`) is unique to your app in Play Console.
- Increment version when tagging new releases if needed (configure versionCode/versionName in Android `build.gradle` as desired).
