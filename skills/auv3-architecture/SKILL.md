---
name: auv3-architecture
description: Use when designing or refactoring Apple AUv3 audio plugin architecture with SwiftUI, AUAudioUnit, AUViewController, AUParameterTree, app extensions, and shared DSP modules.
---

# AUv3 Architecture

Use this skill when building or reviewing an Apple Audio Unit v3 plugin for macOS, iOS, or iPadOS. The default baseline is macOS 14+, iPadOS 17+, Xcode 26.x, and Swift 6.x.

## Core model
- Treat AUv3 as an app-extension architecture with a real-time DSP core.
- Ship a containing app with one embedded `.appex` per audio unit.
- Implement the plugin as an `AUAudioUnit`.
- Use an `AUViewController` conforming to `AUAudioUnitFactory` for plugins with UI.
- Keep SwiftUI in the editor surface, not in the render path.
- Keep the DSP kernel independent of the extension UI.

## Project shape
Use this structure as the working default:

```text
MyPlugin/
|-- App/
|   |-- AppDelegate.swift
|   |-- ContentView.swift
|   `-- HostPreviewScreen.swift
|-- Extension/
|   |-- MyPluginAudioUnit.swift
|   |-- MyPluginViewController.swift
|   |-- Parameters.swift
|   |-- State.swift
|   |-- Info.plist
|   `-- MainInterface.storyboard
|-- SharedDSP/
|   |-- DSPKernel.hpp
|   |-- DSPKernel.mm
|   |-- DSPTypes.h
|   `-- Smoothers.hpp
|-- SharedUI/
|   |-- PluginEditorView.swift
|   `-- MeterModel.swift
`-- Tests/
    |-- DSPUnitTests.swift
    |-- PluginStateTests.swift
    `-- AudioRegressionTests.swift
```

## Required knowledge
- Xcode target, extension, signing, and entitlement mechanics.
- AudioToolbox and AVFAudio basics.
- `AUAudioUnit`, `AUParameterTree`, buses, render resources, latency, and tail reporting.
- Audio Component type, subtype, manufacturer, version, and discovery metadata.
- Swift/C++ or Objective-C++ interop for performance-sensitive DSP.
- Real-time DSP constraints and host automation behavior.

## Plist checklist
- Include a valid `NSExtension` dictionary.
- For UI extensions, use `NSExtensionPointIdentifier` set to `com.apple.AudioUnit-UI`.
- Include either `NSExtensionMainStoryboard` or `NSExtensionPrincipalClass`, depending on the UI setup.
- Include an `AudioComponents` entry with `description`, `manufacturer`, `name`, `sandboxSafe`, `subtype`, `tags`, `type`, and `version`.
- Use Apple AU type codes intentionally: `aufx`, `augn`, `aumu`, or `aufm`.
- Freeze 4-character manufacturer and subtype identifiers early.

## SwiftUI editor rule
Embed SwiftUI inside the required view controller shell:

```swift
import AudioToolbox
import SwiftUI

final class MyPluginViewController: AUViewController, AUAudioUnitFactory {
    private var hosting: NSHostingController<PluginEditorView>?

    override func viewDidLoad() {
        super.viewDidLoad()
        let root = PluginEditorView()
        let host = NSHostingController(rootView: root)
        addChild(host)
        view.addSubview(host.view)
        host.view.frame = view.bounds
        host.view.autoresizingMask = [.width, .height]
        hosting = host
    }

    func createAudioUnit(with componentDescription: AudioComponentDescription) throws -> AUAudioUnit {
        try MyPluginAudioUnit(componentDescription: componentDescription, options: [])
    }
}
```

## Parameter model
- Define stable `AUParameterAddress` values.
- Build an `AUParameterTree` before wiring editor controls.
- Keep parameter IDs stable across releases.
- Use readable and writable flags for host-automatable controls.
- Wire observer/provider callbacks before polishing UI.
- Treat host parameter scheduling as part of the audio contract.

## Architecture checklist
1. Start with an Xcode app project and add an Audio Unit extension target.
2. Decide whether the plugin is an effect, generator, instrument, or music effect.
3. Freeze manufacturer and subtype identifiers.
4. Build the `AUAudioUnit` and parameter tree before editor polish.
5. Keep DSP in a shared Swift, C++, or Objective-C++ module.
6. Host SwiftUI from `AUViewController`.
7. Use the containing app as a standalone preview and test harness.
8. Handle state serialization as a product feature, not an afterthought.
