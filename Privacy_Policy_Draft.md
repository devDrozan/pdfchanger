# PDF Changer — Privacy Audit and Privacy Policy Draft

Review date: **5 November 2026 (Asia/Seoul)**
App: **PDF Changer 0.1.0**, Android package `com.pdfchanger.mobile`
Source baseline: `ef5c1ffc31b14054efc9c96650929b9804097de3`; working tree was clean before this audit.

**Release assessment: privacy review incomplete; resolve the publication blockers below before submission.** Local PDF processing is supported by source evidence. This is a static repository audit, not confirmation of Google Play approval or a runtime network test.

## Scope and evidence limits

Reviewed app source, Expo config, EAS profiles, npm lockfile, installed native module source/manifests, and the existing generated Android debug manifest. A delegated compliance reviewer independently checked native SDK and release/debug distinctions. No documents were uploaded, app behavior changed, or store declarations submitted.

The available merged manifest at `mobile/android/app/build/intermediates/merged_manifests/debug/processDebugManifest/AndroidManifest.xml` was generated on 31 October at 20:26, before the current `mobile/app.json` timestamp of 21:29 that day. It targets API 36 but **does not establish the current production AAB's permissions or SDK inventory**. No release AAB was inspected. Play Console declarations, hosted policy, runtime permission behavior, network traffic, and backup/restore behavior remain unverified. Google Play policy was checked against official pages on the review date; announced future changes are not treated as current obligations.

## 1. File permission

| Finding | Evidence | Assessment and next action |
| --- | --- | --- |
| Camera access is requested when the user chooses Camera photo | `mobile/App.tsx:132–140` | Supported purpose and timing. Denial produces an error; image import remains available. Verify denial/revocation on a device. |
| Photos use the system picker; files use document selection | `mobile/App.tsx:123–148,179–188`; installed `expo-image-picker/.../contracts/ImageLibraryContract.kt:36–79`; `expo-document-picker/.../DocumentPickerModule.kt:42–52` | Appropriate limited selection design. App code does not request full photo-library permission. |
| Legacy storage permissions are still declared | Existing debug manifest: `READ_EXTERNAL_STORAGE` and `WRITE_EXTERNAL_STORAGE`, both capped at API 32; installed picker and filesystem manifests also declare them | **Medium, least-privilege review.** Determine whether supported older devices need either permission for these flows. Block unnecessary permissions through Expo configuration and test compatibility. A declaration alone does not prove runtime access. |
| Development and network capabilities require release verification | Debug manifest includes `SYSTEM_ALERT_WINDOW`, `INTERNET`, `VIBRATE`, `ACCESS_NETWORK_STATE`, `CHANGE_WIFI_MULTICAST_STATE`, and `ACCESS_LOCAL_NETWORK` | **Medium, unverified release state.** Trace each release permission to its module and purpose. Remove unnecessary development permissions through Expo config/plugins. `INTERNET` alone is not evidence of data upload. |
| Microphone, broad modern media, all-files, and ad-ID access | `mobile/app.json:24–29` disables microphone permission; inspected debug manifest has no effective `RECORD_AUDIO`, `READ_MEDIA_IMAGES`, `READ_MEDIA_VIDEO`, `MANAGE_EXTERNAL_STORAGE`, or `com.google.android.gms.permission.AD_ID` | Supported only for inspected config/debug evidence. Recheck final release manifest, including merged dependencies. |

Google restricts broad photo/video and all-files permissions when system pickers can meet core functionality. This app's selection flows provide a strong reason to avoid broad access; being a PDF app does not itself justify it. See [Google Play sensitive permissions policy](https://support.google.com/googleplay/android-developer/answer/16558241?hl=en).

## 2. Data collection

**Supported behavior:** The app accesses user-selected images/PDFs and camera captures to create or merge documents. It reads filenames, file URIs, size, creation time, image dimensions, and PDF page counts. It persists PDF filenames in Recent and the English/Burmese language preference. Evidence: `mobile/App.tsx:123–188`, `mobile/src/files.ts:3–10,31–36`, `mobile/src/images.ts:18–21`, `mobile/src/mergeRecent.ts:4–29`, `mobile/src/i18n.ts:81–98`.

No account flow, advertising SDK, analytics integration, crash-reporting integration, or document-content telemetry was found in the app source/direct dependencies. That observation is **not proof of zero SDK collection**; see section 4.

For Data safety, Google defines collection around off-device transmission; local-only processing is excluded. SDK transmissions count. Transfers to other apps can count as sharing, with an exception for qualifying user-initiated transfers where sharing is reasonably expected. See [Google Play Data safety guidance](https://support.google.com/googleplay/android-developer/answer/10787469?hl=en).

**Provisional declaration guidance:** “No data collected” and “No data shared” are plausible only after verifying the production SDKs, traffic, backup handling, and preview/export flow. Do not submit these answers based solely on the absence of JavaScript network calls. No account-deletion flow is indicated by the inspected app because it has no account creation. Do not claim encryption in transit or at rest without establishing what actually occurs in the release.

## 3. Server upload

**Supported behavior:** No app-authored `fetch`, WebSocket, upload endpoint, remote document-processing API, or cloud-sync flow was found in `mobile/App.tsx` or `mobile/src/`. Image conversion and PDF creation/merging use local bytes (`mobile/src/images.ts:24–44`, `mobile/src/pdfCore.ts:27–55`, `mobile/src/files.ts:39–55,73–80`).

**High, product/privacy claim gap:** OS backup may transfer saved app data despite offline processing; this is separate from an app-operated upload server. See section 5. Avoid an unconditional “files never leave your device” promise.

**Medium, disclosure gap:** Preview passes files to another app on Android. `mobile/src/files.ts:79–80` calls native preview; installed `expo-file-system/.../FileSystemModule.kt:62–75` uses `ACTION_VIEW` and read URI grants. If preview fails, `mobile/App.tsx:231–236` opens the share sheet. The external viewer or selected recipient may handle or upload the document under its own policy. Explain this behavior and review whether a button labelled Preview makes the fallback sharing action sufficiently clear. This is a disclosure/usability recommendation, not a demonstrated automatic upload.

## 4. Third-party SDKs

Versions below come from `mobile/package.json:24–34`; `pdf-lib` resolves to 1.17.1 in the lockfile.

| Component | Role / evidence | Privacy review result |
| --- | --- | --- |
| Expo 58.0.19; React 19.3.0; React Native 0.88.0 | Application/runtime framework | Core infrastructure; inspect resolved release native dependencies and network activity before asserting non-collection. |
| expo-document-picker 58.0.4 | System file selection and cached input copies | Local import path observed; selected providers can include external/cloud providers. |
| expo-image-picker 58.0.10 | Camera and photo selection | Local capture/import observed. Native dependencies include AndroidX and `com.vanniktech:android-image-cropper:4.7.0` (`android/build.gradle:22–28`). |
| expo-image-manipulator 58.0.15 | Rotate, crop, JPEG conversion | Local transforms and cached outputs observed. |
| expo-file-system 58.0.12 | Local persistence/read/delete and native preview | Local operations observed; library also contains network APIs, but no app call to those APIs was found. Native module declares network/legacy storage permissions. |
| expo-sharing 58.0.20 | System share-sheet fallback | User can hand a PDF to another app; disclose recipients' independent handling. |
| pdf-lib 1.17.1 | PDF creation, parsing, page copying | In-memory/local-byte processing in the app. |
| expo-dev-client 58.0.15, transitive dev launcher/menu and updates interface | Development tooling | Must distinguish debugging from store behavior. An updates interface package alone does not establish OTA uploads or analytics. |
| Google Play code scanner 16.1.0; ML Kit barcode scanning 17.3.0 | Present in existing debug native dependency graph/manifest | `expo-dev-launcher/android/build.gradle:126–127` declares them through `debugOnly`. Release inclusion is conditional on `expo.devlauncher.configureInRelease` (`:16,93–95`); no enabling property was found in current Gradle properties/EAS config. Expected to be excluded from normal release, but not verified in an AAB. |

**High, release evidence blocker:** Debug includes ML Kit initialization and Google DataTransport components. These are not proof that the production app collects analytics, nor should Firebase component registration be mistaken for a Firebase Analytics integration. If retained/initialized in release, evaluate the vendor's diagnostic and identifier handling and update disclosures. See [ML Kit Android data disclosure](https://developers.google.com/ml-kit/android-data-disclosure).

Inspect the final AAB and resolved release dependency graph, SDK Index/Play warnings, SDK initialization, and observed traffic. No full vulnerability or SDK-certification audit was performed.

## 5. Data Storage

| Data | Observed location and retention | Assessment |
| --- | --- | --- |
| Completed PDFs | `Paths.document/pdfs`, `mobile/src/files.ts:10,31–55` | Persistent local library. User deletion removes the saved PDF (`:67–70`); source imports are preserved. |
| Recent PDF names | `Paths.document/merge-recent.json`, `mobile/src/mergeRecent.ts:4,25–29` | Persistent metadata. Rename/delete update Recent (`mobile/App.tsx:239–260`). |
| Language choice | `Paths.document/language.json`, `mobile/src/i18n.ts:81–98` | Persistent local preference. |
| Imported PDFs/images and transformed image outputs | Picker cache copies (`mobile/App.tsx:145,181`), image manipulator outputs (`mobile/src/images.ts:24–44` and installed native modules) | **Medium, retention gap.** Removing a page or saving clears React state but does not explicitly delete those working files (`mobile/App.tsx:172,224,323`). No complete flow-level cache cleanup was found. Do not promise immediate removal or a fixed retention period. |
| Temporary PDF output | `Paths.cache`, `mobile/src/files.ts:45–54` | Moved to library on success; deleted on a handled failure. This cleanup does not cover all imported/transformed files. |

**High, product-boundary mismatch:** Existing Android main/debug manifests set `android:allowBackup="true"`, with no explicit backup/extraction rules found. App files can therefore be eligible for OS-managed backup. Expo config does not currently set a backup restriction. This is eligibility evidence, not proof a user's files have actually been backed up. Android backs up most app files by default, excludes cache, and supports cloud and device-transfer controls; manufacturer behavior can differ. See [Android Auto Backup documentation](https://developer.android.com/identity/data/autobackup).

Recommended correction for the repository's offline/no-cloud boundary: configure backup exclusion through Expo config and a config plugin as needed, covering supported legacy backup and modern cloud/device-transfer paths; verify the generated release rules and restore behavior. `allowBackup=false` alone does not establish universal prevention of device migration. Check iOS backup behavior before making the same claim across platforms.

Local files use platform app storage; no app-level PDF encryption, password protection, or secure-erasure mechanism was found. Deletion is ordinary filesystem deletion, not a guarantee of forensic erasure or deletion of exported/backup copies. Add scoped temporary-file cleanup after save, removal, cancellation, and recovery without deleting source documents or active working inputs. These are privacy/product recommendations, not a blanket claim that every cache file violates Play policy.

## 6. Privacy Policy

**High, observed publication blocker:** No privacy policy text, URL, or navigation entry exists in the inspected app. No existing policy artifact was found in the tracked repository. A hosted policy or Play Console link outside this repository was not verified.

Google requires a privacy policy in Play Console and a link or text inside the app, even for apps without sensitive-data collection. It must describe data handling, security, retention/deletion, developer identity/contact, and be available at a public, active, non-geofenced, non-editable URL rather than a PDF. See [Google Play User Data — Privacy Policy](https://support.google.com/googleplay/android-developer/answer/10144311?hl=en).

Before publication: supply developer identity, privacy contact, effective date, and public policy URL; settle backup handling; verify production SDK behavior; then align the policy, in-app behavior, and Data safety answers. Make the policy easy to find for the app's English/Burmese audience. Translation is a usability recommendation here.

## Release verification checklist

- [ ] Finalize the policy fields below, publish the user-facing policy, add it inside the app, and enter the URL in Play Console.
- [ ] Inspect a fresh production AAB: permissions, debug flags, cleartext allowance, native SDKs, initialization providers, backup rules, and target API. Do not use the old debug manifest as release evidence.
- [ ] Verify release traffic during cold start, idle, capture, photo/file import, edits, PDF save/merge, preview/share, and deletion. Investigate every destination and SDK transmission before finalizing Data safety.
- [ ] Resolve backup behavior and test supported backup/restore/device-transfer paths; correct any device-only claims.
- [ ] Add or accurately disclose working-file cleanup/retention; test save/failure/cancel/remove/relaunch without altering sources.
- [ ] Test camera denial/revocation, Android picker paths on supported versions, and external viewer/share behavior.
- [ ] Review and submit accurate Data safety declarations for the actual distributed app and relevant active versions. Record the build/version used for verification.

## User-facing privacy policy draft

**Editorial status: not ready to publish.** Replace the bracketed identity/contact/date fields. Resolve release SDK behavior and backup controls first. The wording below describes the observed local-processing implementation and current backup uncertainty; revise it if those behaviors change. Publish only this user-facing section after verification, not the internal audit/checklist.

### Privacy Policy for PDF Changer

Effective date: **[INSERT EFFECTIVE DATE]**
Developer: **[INSERT DEVELOPER OR LEGAL ENTITY NAME]**
Privacy contact: **[INSERT WORKING PRIVACY EMAIL OR CONTACT URL]**

PDF Changer helps you photograph pages, turn images into PDFs, merge PDFs, and manage saved PDFs. Its document creation and merging features process files on your device and work offline. You do not need a PDF Changer account.

### Information the app accesses and uses

When you choose a feature, the app accesses the photos, images, PDFs, or camera captures you select. It uses their contents and file information, such as names, sizes, image dimensions, and page counts, to display, arrange, edit, create, merge, and manage documents. Your documents may contain personal or sensitive information depending on what you select.

The app saves completed PDFs, a Recent list of saved PDF names, and your language preference in its local app storage. Original imported files are not changed by PDF creation or merging.

### Permissions and your choices

Camera permission is requested when you choose to photograph a page. You can decline it and use file or photo import instead. System pickers let you select images and documents. You can manage app permissions in your device settings.

### Uploads, external services, and sharing

PDF Changer's document-processing features do not upload your documents to a developer-operated server or provide a PDF Changer cloud-sync service.

On Android, preview can open a PDF in an external viewer. If preview is unavailable, the app can offer your device's share sheet. If you open or send a PDF through another app, that app receives access to the file and handles it under its own terms and privacy policy. File/photo providers selected through system pickers may also offer cloud-backed content.

Operating-system backup or device-transfer features may copy or restore saved app data depending on your device and settings. PDF Changer does not operate those services. Copies held by external apps, selected providers, or backup services are outside the app's local library controls.

### Storage, retention, and deletion

Completed PDFs stay in the app library until you delete them or the app's local data is removed. Deleting a PDF removes that saved app copy and its Recent entry. It does not delete original imported files or copies held elsewhere.

Working copies of imported files and edited images can remain in the app's cache after an operation. Cache removal timing depends on the app and operating system; immediate deletion is not guaranteed in the reviewed version. The app also keeps your language preference and Recent metadata locally. Device controls for clearing app data or uninstalling can remove local app data; external copies and backups are managed separately and may later be restored.

### Security

The app uses platform app storage and system file-selection mechanisms. Opening or sharing a PDF grants access to external software. The reviewed version does not provide app-level PDF encryption or guaranteed secure erasure. Protect your device and consider the information in a document before opening it with another app or sharing it.

### Changes and contact

We may update this policy as the app's data handling changes. The effective date above identifies the current policy. For privacy questions, contact **[INSERT THE SAME PRIVACY CONTACT]**.

**Publication gate — remove after resolution:** Verify all production SDK collection/sharing and add any required recipient, data-type, purpose, retention, security, or user-control disclosures. The absence of advertising/analytics code in the app is not sufficient to publish an unconditional zero-collection claim. Confirm that the backup and temporary-file paragraphs still match the shipped version.
