Read this before merging upstream. Every item must still work after the merge.

# Fork features

What `djnesmith/Nocturne` adds over `gabriele-rizzo/Nocturne`. The list comes from
`git log upstream/main..main`. Each new feature adds a line here.

After any merge, run the tests and build, then install:

```
xcodebuild test -project Nocturne.xcodeproj -scheme Nocturne -destination 'platform=macOS'
xcodebuild build -project Nocturne.xcodeproj -scheme Nocturne -configuration Release -derivedDataPath build/derived.noindex
ditto build/derived.noindex/Build/Products/Release/Nocturne.app /Applications/Nocturne.app
```

"In the app" checks need the built app at `/Applications/Nocturne.app`.

GitHub Actions are disabled on the fork. Upstream's `release.yml` would publish a release on
every push to main. Keep them off: `gh api repos/djnesmith/Nocturne/actions/permissions`
shows `"enabled":false`.

## Security

### 1. No in-app updates
Sparkle is gone, so the app never checks for, downloads or installs updates. Upstream's
release tooling (`Scripts/make-appcast.sh`, `.github/workflows/release.yml`) and the README's
Updates section still mention Sparkle. They are not part of the app.
- Commits: e9be58c
- Files: `Nocturne.xcodeproj/project.pbxproj`, `Info.plist`, `Signing.xcconfig`,
  `Nocturne/NocturneApp.swift`, `Nocturne/SettingsView.swift`
  (`Nocturne/Managers/UpdateManager.swift`, `Nocturne/UpdatesSection.swift` and
  `Package.resolved` deleted)
- Check: `grep -ri sparkle Nocturne Nocturne.xcodeproj Info.plist Signing.xcconfig` finds
  nothing. In the app: `lsof -nP -i -a -p $(pgrep -x Nocturne)` shows no sockets.

### 2. No library-validation exemption
`com.apple.security.cs.disable-library-validation` is removed. The app has no entitlements.
The backlight still works, because CoreBrightness is an Apple platform library.
- Commits: e9be58c, 26fe5dd (Release builds skip Xcode's injected `get-task-allow`)
- Files: `Nocturne/Nocturne.entitlements`, `Signing.xcconfig`
- Check: `codesign -d --entitlements - /Applications/Nocturne.app` prints an empty dict. In the
  app, the menu shows the Schedule items, not "Unavailable on this Mac".

### 3. David's signing identity and bundle ID
Builds are signed as "Apple Development: David Nesmith (JFGJA89Z8H)" with the hardened
runtime. The bundle ID is `com.djnesmith.Nocturne`, and the tests use
`com.djnesmith.NocturneTests`.
- Commits: 26fe5dd
- Files: `Signing.xcconfig`, `Nocturne.xcodeproj/project.pbxproj` (bundle IDs; the targets'
  `CODE_SIGN_STYLE = Automatic` lines are removed so the xcconfig applies)
- Check: `codesign -dv --verbose=2 /Applications/Nocturne.app` shows
  `Identifier=com.djnesmith.Nocturne`, `TeamIdentifier=CHJ32A2B4N` and
  `flags=0x10000(runtime)`.
