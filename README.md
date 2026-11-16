# pd-ota

OTA JS bundle host for the Pandar app (`react-native-ota-hot-update`).

The app fetches `update.json` from
`https://raw.githubusercontent.com/pandar-devops/pd-ota/main/update.json`
on every release-build launch (`src/hooks/useCheckVersion.ts` in the app repo).

## How the app decides to update

For its own platform block (`ios` / `android`), the app downloads `url` when:

1. its store version (`CFBundleShortVersionString` / `versionName`) is in
   `allowedAppVersions`, **and**
2. `version` is greater than the OTA bundle version it already has (0 on a
   fresh install).

The bundle is unzipped and the app restarts into it. iOS and Android are gated
independently — bump only the platform you are shipping.

An empty `allowedAppVersions` turns OTA off for that platform.

## Shipping an update

From the Pandar app repo, on the commit you want to ship:

```bash
npm run export-ios       # -> ios/main.jsbundle.zip + ios/sourcemap.zip
npm run export-android   # -> android/index.android.bundle.zip + android/sourcemap.zip
```

Then here:

1. Copy the zip into `ios/` or `android/` with the new version in its name —
   e.g. for version 3: `ios/main-v3.jsbundle.zip`,
   `android/index.android.bundle-v3.zip`. Delete the previous version's zip.
2. In `update.json`, bump that platform's `version` to match, point its `url`
   at the new filename, and make sure the store version you're targeting is
   in `allowedAppVersions`.
3. Commit and push to `main`.

Only ship JS-only changes. If a change touches native code (new native module,
pod, gradle dep, Info.plist/manifest), it needs a store release — and the
store version it ships in must **not** be added to the allowlist of a bundle
built against the old native code.

`raw.githubusercontent.com` caches each file for up to ~5 minutes. The app
cache-busts `update.json`, and a new filename per version means a device can
never be served the previous version's zip from cache — so never overwrite a
zip in place.

## Rolling back

Re-upload the previous good bundle under the next version's filename and bump
`version` again (devices only move forward), or clear `allowedAppVersions` to stop further downloads.
