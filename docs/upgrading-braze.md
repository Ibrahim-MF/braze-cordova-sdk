# Upgrading the Braze Cordova SDK

This guide covers how to rebase company-specific changes onto a new upstream Braze Cordova SDK release.

## Rule

**Never mix an upstream version bump and a COMPANY change in the same commit.**

## Files we own (COMPANY diff)

These are the only files that contain company-specific changes. Check each one during a rebase:

| File | What to watch for |
|---|---|
| `www/BrazePlugin.js` | Upstream may add new methods near the end of the file; ensure `initialize` is still present after the last upstream method |
| `src/android/BrazePlugin.kt` | Check `pluginInitialize()`, `execute()`, `onStart/Stop/Resume/Pause()`, and `companion object` for conflicts with upstream changes |
| `src/ios/BrazePlugin.m` | Check `pluginInitialize`, `didFinishLaunchingListener:` guard, and the new `initializeBraze:` method |

## Step-by-step upgrade

### 1. Fetch the new upstream release

```bash
git fetch upstream
git log upstream/<NEW_TAG> --oneline -10   # review what changed
```

### 2. Review the upstream diff against our base version

```bash
git diff 17.0.0..upstream/<NEW_TAG> -- src/android/BrazePlugin.kt src/ios/BrazePlugin.m www/BrazePlugin.js plugin.xml
```

Pay special attention to:
- Any new initialization paths in `pluginInitialize()` (Android or iOS)
- New `execute()` action handlers that might conflict with our `"initialize"` guard
- New lifecycle hook logic
- iOS: any new observers added in `pluginInitialize`

### 3. Create an upgrade branch

```bash
git checkout -b upgrade/braze-<NEW_VERSION>
```

### 4. Merge upstream (non-interactive)

```bash
git merge upstream/<NEW_TAG> --no-commit
```

Resolve any conflicts. Our COMPANY changes are clearly labeled with `// COMPANY:` comments, making
them easy to identify against upstream code.

### 5. Verify COMPANY changes are intact

```bash
grep -r "COMPANY:" src/ www/
```

All `// COMPANY:` markers should still be present. If any are missing, re-apply them from the
previous version's diff.

### 6. Commit the upgrade

```bash
git add -A
git commit -m "COMPANY: Upgrade upstream Braze Cordova SDK from 13.0.0 to <NEW_VERSION>

- Rebased country-aware initialization changes onto <NEW_VERSION>
- Resolved conflicts in: <list files>
- No behavioral changes to COMPANY logic

Co-Authored-By: <author>"
```

### 7. Update documentation

- Update `UPSTREAM.md` — change the upstream version number
- Add an entry to `COMPANY_CHANGELOG.md` under a new version heading

### 8. Update the app

In the MoneyFellows app repo, update `package.json`:

```json
"braze-cordova-sdk": "github:<YOUR_ORG>/braze-cordova-sdk#<NEW_BRANCH_OR_TAG>"
```

Then run:

```bash
npm install
npx cap sync android
npx cap sync ios
```

Verify that the synced files in `android/capacitor-cordova-android-plugins/` and
`ios/capacitor-cordova-ios-plugins/` reflect the new version.

## Identifying all COMPANY commits at any time

```bash
git log --oneline | grep "COMPANY:"
```

## Emergency rollback

If the new version causes issues, pin the app back to the previous fork tag:

```json
"braze-cordova-sdk": "github:<YOUR_ORG>/braze-cordova-sdk#company-17.0.0"
```
