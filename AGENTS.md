# Project boundaries

- This repository contains an offline document capture and PDF app for Android and iPhone.
- The mobile app lives in `mobile/` and uses React Native, TypeScript, and Expo development builds.
- Document capture, image conversion, PDF merging, and the completed-PDF library run on the device. Do not upload user documents or add accounts, analytics of document contents, or cloud storage without an explicit product decision.
- Keep platform configuration in Expo app config and config plugins where possible. Do not hand-edit generated `ios/` or `android/` projects as the source of truth.
- `SPEC.md` contains changing user-facing requirements and acceptance criteria only. Keep technology choices and project boundaries here.
- `CLAUDE.md` points to this file rather than duplicating rules.
- The repository root and every subproject must have a README for people using or developing that project.
- Preserve source images and imported PDFs when producing output. Completed PDFs are the only persistent library items; temporary working files can be removed after a flow ends.
