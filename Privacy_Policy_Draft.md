# PDF Changer — Privacy Audit and Privacy Policy Draft

> Updated 8 November 2026: The AdMob addendum and user-facing policy below reflect the newly added ads SDK. The 5 November audit is historical; its absence-of-ads and provisional zero-collection conclusions are superseded. Production SDK behavior, consent configuration, and store declarations still require release verification.

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

## AdMob update — 8 November 2026

This focused source review supersedes the historical findings above where advertising changes data handling. Existing working-tree edits were preserved; no app code was changed for this policy update.

- **File permission:** Network/advertising permissions must be checked in the final manifest. The older debug manifest's absence of advertising ID permission is no longer current evidence. No new document-access purpose is introduced by ads.
- **Data collection:** `mobile/package.json` and the lockfile now include `react-native-google-mobile-ads` 17.4.0. Do not submit the former provisional “no data collected/shared” answers for an ads-enabled release. Review approximate location, app interactions, diagnostics/performance, and device/other identifiers against the shipped settings. Sources: [Google SDK disclosure](https://developers.google.com/admob/android/privacy/play-data-disclosure), [Play Data safety guidance](https://support.google.com/googleplay/android-developer/answer/10787469?hl=en).
- **Server upload:** Document processing remains local in inspected source; ad requests introduce off-device advertising traffic. Development banners use Google test unit IDs and release banners use the configured production unit. Neither banner receives document contents or filenames (`mobile/src/components/TestBannerAd.tsx`, `mobile/src/components/MergeTestBannerAd.tsx`). Test ads do not establish an exemption from SDK disclosures.
- **Third-party SDKs:** Installed wrapper metadata lists Android Google Mobile Ads 25.5.0 / UMP 4.0.0 and iOS Google Mobile Ads 13.10.0 / UMP 3.1.0. These are dependency defaults, not proof of the built release versions. No mediation adapters were identified in the direct package list. Re-review if more ad networks are added. [iOS SDK disclosure](https://developers.google.com/admob/ios/privacy/data-disclosure) currently describes a later SDK version; verify version-specific behavior before iOS release.
- **Data Storage:** Local PDF/cache retention and backup caveats remain applicable. Advertising/consent storage is additional. Google-managed data retention and deletion are separate from deleting a local PDF; do not promise a developer-controlled fixed SDK retention period.
- **Privacy Policy:** The public policy now identifies AdMob, ad-related processing, purposes, recipients, controls, and third-party retention. Effective date: 8 November 2026. Developer/contact: Robindev / rozannainghangu@gmail.com. Public URL: https://devdrozan.github.io/pdfchanger/.

### Consent and release checks still required

`mobile/App.tsx` uses `AdsConsent.gatherConsent()` and checks `canRequestAds` before initialization; `mobile/app.json` sets `delayAppMeasurementInit: true`. These do not prove that the required messages are configured or that every region's choices work. Settings provides Privacy Choices when UMP reports that the entry point is required. Verify the configured consent messages and entry point in the release app. See [Google UMP guidance](https://developers.google.com/admob/android/privacy).

The Library and Merge banners use Google test units in development and the configured production banner unit in release. No explicit non-personalized-only request option was found, so the policy does not promise exclusively non-personalized ads. Verify regional consent/rejection, advertising ID handling, applicable iOS tracking authorization, audience/child-directed settings, AdMob Privacy & messaging setup, and actual release traffic before distribution. Update Play Console's Data safety, advertising ID, and “Contains ads” declarations to match the released app. In-app privacy-policy access and Android backup restrictions are implemented; verify them against the release artifact. This text update does not certify compliance or submit store forms.

## User-facing privacy policy

### Privacy Policy for PDF Changer

Effective date: **8 November 2026**

Developer: **Robindev**

Privacy contact: **rozannainghangu@gmail.com**

Public URL: https://devdrozan.github.io/pdfchanger/

PDF Changer creates and merges PDFs on your device. The app also uses Google AdMob for advertising, which involves network communication and advertising-related data. You do not need a PDF Changer account to use the app.

### 1. Information the app accesses and uses

PDF Changer, developed by Robindev, helps you photograph pages, turn images into PDFs, merge PDFs, and manage saved PDFs. Its document creation and merging features process files on your device and work offline.

When you choose a feature, the app accesses the photos, images, PDFs, or camera captures you select. It uses their contents and file information, such as names, sizes, image dimensions, and page counts, to display, arrange, edit, create, merge, and manage documents. Your documents may contain personal or sensitive information depending on what you select.

The app saves completed PDFs, a Recent list of saved PDF names, and your language preference in its local app storage. Original imported files are not changed by PDF creation or merging.

### 2. Permissions and your choices

Camera permission is requested when you choose to photograph a page. You can decline it and use file or photo import instead. System pickers let you select images and documents. You can manage app permissions in your device settings.

### 3. Google AdMob advertising

PDF Changer uses Google's AdMob service through the Google Mobile Ads SDK to display ads. Ad requests require an internet connection. The app does not provide your document contents, selected images, or PDF filenames to the advertising SDK.

The SDK collects and shares advertising-related data with Google, including IP addresses (which can indicate approximate location), ad and app interactions, diagnostic and performance information, and device or account identifiers where available, such as Android advertising ID and app set ID. These are used for advertising, measurement and analytics, and fraud prevention. Google states that SDK data is encrypted in transit. Actual handling depends on platform, consent, settings, and SDK configuration. See [Google's SDK disclosure](https://developers.google.com/admob/android/privacy/play-data-disclosure) and [Google's Privacy Policy](https://policies.google.com/privacy).

Ads may be personalized or non-personalized depending on your choices and applicable settings. Non-personalized ads can still involve data processing. Google may process information through its service providers and advertising partners as described in [how Google uses information from partner apps](https://policies.google.com/technologies/partner-sites).

The app uses Google's consent service to check whether ads can be requested and to display consent messages when required by the configured service. Review the choices in any consent message you receive. You can also use your device's privacy settings to manage advertising identifiers and, on supported Apple devices, tracking permissions. Available controls vary by device and region; limiting an advertising identifier does not stop all advertising-related processing. For details, see [Google's advertising controls](https://policies.google.com/technologies/ads).

### 4. Uploads, external services, and sharing

PDF Changer's document-processing features do not upload your documents to a developer-operated server or provide a PDF Changer cloud-sync service. Advertising-related data is transmitted to Google as described above; offline document processing does not mean the entire app operates without network communication.

On Android, preview can open a PDF in an external viewer. If preview is unavailable, the app can offer your device's share sheet. If you open or send a PDF through another app, that app receives access to the file and handles it under its own terms and privacy policy. File and photo providers selected through system pickers may also offer cloud-backed content.

Operating-system backup or device-transfer features may copy or restore saved app data depending on your device and settings. PDF Changer does not operate those services. Copies held by external apps, selected providers, or backup services are outside the app's local library controls.

### 5. Storage, retention, and deletion

Completed PDFs stay in the app library until you delete them or the app's local data is removed. Deleting a PDF removes that saved app copy and its Recent entry. It does not delete original imported files or copies held elsewhere.

Working copies of imported files and edited images can remain in the app's cache after an operation. Cache removal timing depends on the app and operating system; immediate deletion is not guaranteed. The app also keeps your language preference and Recent metadata locally.

Device controls for clearing app data or uninstalling can remove local app data. External copies and backups are managed separately and may later be restored.

Advertising data is retained by Google under its own policies and retention schedules. Deleting a PDF or uninstalling PDF Changer does not by itself delete data already received by Google. See [Google's retention policy](https://policies.google.com/technologies/retention) for its retention and deletion practices. For privacy requests concerning PDF Changer, contact Robindev using the address below.

### 6. Security

The app uses platform app storage and system file-selection mechanisms. Opening or sharing a PDF grants access to external software. PDF Changer does not provide app-level PDF encryption or guaranteed secure erasure. Protect your device and consider the information in a document before opening it with another app or sharing it.

### 7. Changes and contact

We may update this policy as the app's data handling changes. The effective date above identifies the current policy.

Robindev
For privacy questions, email [rozannainghangu@gmail.com](mailto:rozannainghangu@gmail.com).

If you email us, we receive the information you include and use it to respond to your request. Please avoid sending sensitive documents with a privacy inquiry.
