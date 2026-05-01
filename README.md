# Build AU Plugins

Build AU Plugins is a Codex plugin with focused skills for Apple AUv3 plugin development, SwiftUI editor integration, host validation, real-time DSP, filters, dynamics, testing, and optimization.

## Skills
- `auv3-architecture`: AUv3 app-extension architecture, SwiftUI editor hosting, plist metadata, and parameter trees.
- `auv3-validation-distribution`: `pluginkit`, `auval`, signing, notarization, and host compatibility.
- `dsp-realtime-control`: render-thread safety, parameter handoff, smoothing, and anti-aliasing boundaries.
- `dsp-filters-dynamics`: modulation-safe filters, compressors, limiters, detector choices, and true-peak behavior.
- `dsp-testing-optimization`: regression renders, loudness checks, profiling, Accelerate/vDSP/SIMD, and numerical hardening.

## Install Locally
Clone this repository into your global Codex plugins directory:

```bash
mkdir -p ~/plugins
git clone https://github.com/rnasound/build-au-plugins.git ~/plugins/build-au-plugins
```

Then add it to your global marketplace at `~/.agents/plugins/marketplace.json`:

```json
{
  "name": "arenakian-local",
  "interface": {
    "displayName": "Arenakian Local"
  },
  "plugins": [
    {
      "name": "build-au-plugins",
      "source": {
        "source": "local",
        "path": "./plugins/build-au-plugins"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Coding"
    }
  ]
}
```

Restart or refresh Codex plugin discovery after installing.
