---
name: auv3-validation-distribution
description: Use when building, signing, validating, troubleshooting, packaging, or distributing AUv3 plugins across Logic, Ableton Live, iPad AUv3 hosts, TestFlight, App Store, or direct macOS distribution.
---

# AUv3 Validation And Distribution

Use this skill when an AUv3 plugin does not appear in hosts, fails `auval`, behaves differently across hosts, or needs release packaging.

## Build defaults
- Build macOS plugins as universal binaries when Intel support matters.
- Use arm64 for iPadOS.
- Use automatic signing unless there is a concrete reason to manage signing manually.
- Keep bundle identifiers unique per target.
- Add capabilities only when they are required and documented.
- Use App Groups only when the containing app and extension need shared files or state.

## Baseline commands
```bash
xcodebuild \
  -project MyPlugin.xcodeproj \
  -scheme MyPlugin \
  -configuration Debug \
  build test

xcodebuild \
  -project MyPlugin.xcodeproj \
  -scheme MyPlugin \
  -configuration Release \
  -destination "generic/platform=macOS" \
  archive \
  -archivePath build/MyPlugin.xcarchive

xcodebuild \
  -exportArchive \
  -archivePath build/MyPlugin.xcarchive \
  -exportPath build/export \
  -exportOptionsPlist ExportOptions.plist

pluginkit -m -A -D -p com.apple.AudioUnit-UI -v
pluginkit -m -i com.example.MyPlugin.Extension -v

auval -v aufx gain Acme

xcrun notarytool submit build/export/MyPlugin.zip --wait
xcrun stapler staple build/export/MyPlugin.app
xcrun stapler validate build/export/MyPlugin.app
```

## Host compatibility
- Logic Pro supports classic Audio Units and AUv3 extensions, but test the exact Logic version.
- Ableton Live supports AUv3 on macOS in Live 11.2 and later; verify Apple silicon, Rosetta, and host architecture behavior.
- iPad plugins are AUv3 app extensions hosted by compatible apps such as AUM or Loopy Pro.
- Compatibility claims must include host version, CPU architecture, and whether the host runs native or translated.

## Failure modes
| Failure mode | Typical cause | Fastest fix |
|---|---|---|
| Extension never appears in host | Wrong extension point, bad `AudioComponents`, app not installed or launched | Verify plist keys, reinstall containing app, query with `pluginkit` |
| UI fails to load | Principal class, storyboard, or UI-extension mismatch | Use `AUViewController` and keep storyboard/class wiring consistent |
| `auval` fails | Bad identifiers, bus format bugs, render-resource bugs | Recheck 4-character codes, bus formats, `allocateRenderResources`, reset paths |
| Intel works but Apple silicon fails | Missing arm64 slice or incompatible low-level code | Build native/universal binaries and retest in native host mode |
| Logic works but Live misses plugin | Live version or architecture mismatch | Verify Live 11.2+, Apple silicon behavior, and compatibility service behavior |
| State sharing fails | Missing App Group or bad entitlements | Add the minimum shared-group entitlement and retest export |

## Distribution guidance
- For App Store and TestFlight, distribute the containing app with the embedded extension.
- For direct macOS distribution, archive, export, notarize, staple, and verify the containing app.
- A signed `.app` in a zip or DMG is usually sufficient for direct AUv3 distribution.
- Use a signed `.pkg` only when there is a real installation requirement.
- Archive once and branch packaging by distribution channel.

## Release checklist
1. Build and test Debug.
2. Archive Release.
3. Export a distribution-signed app.
4. Reinstall the containing app cleanly.
5. Confirm PlugInKit registration.
6. Run `auval` against the final identifiers.
7. Test Logic and Ableton Live on the host versions you claim to support.
8. Test at least one iPad AUv3 host when iPadOS is in scope.
9. Notarize and staple for direct macOS distribution.
10. Document architecture support and known host limitations.
