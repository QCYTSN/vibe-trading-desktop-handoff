# Vibe-Trading Desktop Community Handoff

> Unofficial Windows desktop prototype for [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading). This repository is a review handoff, not an official HKUDS release.

[English handoff](README.en.md) · [中文交接说明](README.zh-CN.md)

![Desktop preview](preview/desktop-0.3.0-window.png)

## Status

- Upstream package baseline: Vibe-Trading `0.1.11`
- Desktop prototype: `0.3.0 Alpha`
- Target: Windows 10/11 x64
- Packaging: Electron + NSIS with an embedded Python 3.12 runtime
- Signature: unsigned community Alpha

The original implementation was developed from an extracted source snapshot without `.git` history. `source-overlay/` preserves the known added and modified files at their upstream-relative paths. It is intended for architecture review and later reconstruction as focused, DCO-signed commits against current upstream `main`; it must not be applied blindly as a patch.

## Repository layout

```text
.
├── README.md
├── README.en.md
├── README.zh-CN.md
├── LICENSE
├── NOTICE
├── preview/
├── release-assets/       # local upload staging; large binaries are gitignored
└── source-overlay/       # known desktop/frontend/backend/workflow changes
```

## Installer

The Windows installer is intentionally excluded from Git history because it is approximately 316 MiB. It should be attached to the GitHub Release `desktop-v0.3.0-alpha` together with `SHA256SUMS-0.3.0.txt`.

Expected installer SHA-256:

```text
c27dfd2408c5b1218c948a7f775e948b9885c548a072ce6f426fa6099f88e3d1
```

The local working copy may contain the installer under `release-assets/`, but Git will ignore it. No API keys, user credentials, or signing keys are included.

## Review

See the language-specific handoff documents for architecture, implemented features, validation results, known limitations, code-signing options, build commands, and the proposed upstream contact text.

This project preserves the upstream MIT license and remains clearly labeled Community/Unofficial unless HKUDS explicitly approves an official distribution.
