# Build AU Plugins Marketplace

This repository is a Codex plugin marketplace for audio plugin development workflows.

## Included Plugins
- `build-au-plugins`: Codex skills for Apple AUv3 architecture, SwiftUI plugin editors, host validation, distribution, real-time DSP, filters, dynamics, testing, and optimization.

## Install In Codex
Add this marketplace from GitHub:

```bash
codex plugin marketplace add rnasound/build-au-plugins
codex plugin marketplace upgrade rnasound-au-plugins
```

Then enable `Build AU Plugins` in the Codex plugin UI and restart or refresh plugin discovery.

## Local Development
The marketplace manifest is [.agents/plugins/marketplace.json](.agents/plugins/marketplace.json). The plugin bundle lives at [plugins/build-au-plugins](plugins/build-au-plugins).

After editing plugin metadata or skill files, commit and push:

```bash
git add .
git commit -m "Update Build AU Plugins"
git push
```
