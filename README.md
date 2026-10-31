# XTG AI — public update channel

This repository is the self-update channel of the **XTG AI** Android app (`com.ishaq.xtgai`). It contains only public files:
signed APKs and `update.json` records. **No keys, tokens or secrets.**

## How the app updates
On launch and every 4 hours the app reads the update record from several independent sources and uses the highest valid
`versionCode`:

1. `https://github.com/abedinhardteam-alt/xtgai-releases/releases/latest/download/update.json` (GitHub Release asset)
2. `https://raw.githubusercontent.com/abedinhardteam-alt/xtgai-releases/main/update.json`
3. `https://cdn.jsdelivr.net/gh/abedinhardteam-alt/xtgai-releases@main/update.json` (jsDelivr CDN)
4. Firestore `xtgai_config/app_update` (field `json`, public read-only) in project `test-media-app-3415`

The APK is downloaded from the `urls` list in order (GitHub Release → raw.githubusercontent → jsDelivr). Before installing,
the app checks the **SHA-256**, the **package name**, a **higher versionCode**, and that the APK is signed with the official
certificate `707a836263f8e1804cfdef3adbae8a93e31ca01c1c11b874388d6e58670e9525`. Android then asks the user for one
"Install" tap (Android does not allow silent updates for normal apps).

## update.json
```json
{"versionCode": 6, "versionName": "1.1.3", "package": "com.ishaq.xtgai", "sha256": "…", "size": 0,
 "cert": "707a8362…9525", "notes": "…", "urls": ["https://…", "https://…", "https://…"]}
```

## Publishing a new version
Push `incoming/part-00…` (base64 chunks of the signed APK) and then `incoming/release.json`
(`{"versionCode":…, "versionName":"…", "sha256":"…", "notes":"…"}`). The `publish` workflow reassembles the APK, refuses it
unless the SHA-256, signing certificate, package and version all match, then commits it under `apk/`, writes `update.json`,
tags `vX.Y.Z` and creates the GitHub Release. Then update the Firestore record (`xtgai_config/app_update`).
