# Vibe-Trading Desktop Community 0.3.0

[简体中文](README.zh-CN.md) | [Short overview](README.md)

## Overview

Vibe-Trading Desktop Community is an unofficial Windows desktop prototype for [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading). It preserves the upstream React interface and Python/FastAPI agent, then adds a desktop host, Windows packaging, local lifecycle management, and desktop-focused usability and security improvements.

- Upstream package baseline: Vibe-Trading `0.1.11`
- Desktop version: `0.3.0 Alpha`
- Supported platform: Windows 10/11 x64
- Desktop stack: Electron + React/Vite + FastAPI + embedded Python 3.12 runtime

This is a community Alpha, not an official HKUDS desktop application. It is intended for research and evaluation and does not provide investment advice.

![Vibe-Trading Desktop home](preview/desktop-home.png)

## What has been implemented

### Desktop host and packaging

- Starts the existing local Vibe-Trading backend automatically and loads it in a native desktop window.
- Uses a random `127.0.0.1` port and a per-launch authentication secret.
- Waits for backend health before showing the application.
- Provides single-instance behavior, startup diagnostics, log capture, and controlled shutdown.
- Cleans up the embedded Python process tree when the application exits.
- Builds a Windows installer containing the Electron application, frontend assets, and an isolated Python runtime.
- Uses the upstream project icon and includes license and notice files.

### Credentials and local security

- Migrates desktop API credentials to Electron `safeStorage` on Windows, backed by Windows user-level encryption.
- Injects credentials into the backend process without displaying secrets in the interface or normal logs.
- Adds a first-run disclosure covering local data, credential handling, unofficial project status, and financial risk.
- Keeps the backend bound to loopback instead of exposing it to the local network by default.

### Models and chat experience

- Loads models from the selected provider after the user supplies a valid key.
- Displays the complete discovered model list instead of filtering it by the current text value.
- Replaces the browser-style model list with a consistent searchable selector while preserving manual model entry.
- Shows the configured provider, model, and reasoning setting in the chat interface.
- Records the provider-returned model identifier when available and shows response duration.
- Fixes the reply copy button while retaining normal text selection and `Ctrl+C` behavior.

The model's natural-language answer about its own identity is not treated as runtime evidence. The desktop UI uses actual configuration and response metadata instead.

### Loading and navigation

- Preloads main route bundles after startup to reduce the delay on the first visit to each page.
- Preserves same-origin HTTP/SSE behavior by loading the UI through the local FastAPI service rather than `file://`.

### IM channel center

- Uses the upstream adapter registry rather than hard-coding a single platform.
- Surfaces supported adapters, dependency status, configuration state, runtime state, and pairing guidance.
- Includes the existing WeChat QR/pairing path while keeping other upstream-supported adapters available when their optional dependencies and credentials are installed.

### Release and update foundation

- Includes a draft GitHub Actions workflow for reproducible Windows builds and release assets.
- Includes a manual GitHub Releases update-check foundation.
- The local `0.3.0` build intentionally leaves the update feed disabled until the project has a stable release repository and a real upgrade path has been tested.

## Architecture

```text
Windows desktop application
        |
        +-- Electron main process
        |     +-- window and application lifecycle
        |     +-- encrypted credential storage
        |     +-- backend health checks and logs
        |     +-- update-check foundation
        |
        +-- embedded Python runtime
              +-- local FastAPI service on 127.0.0.1:<random-port>
              +-- existing Vibe-Trading agent, tools, data sources, and connectors
              +-- serves the compiled React application
```

Keeping the frontend and API on the same local origin preserves existing REST, SSE, upload, and SPA routing behavior.

## Installation

The public Git repository does **not** include an EXE installer. The source tree and checksum metadata are versioned in Git; generated installers must be attached separately to a GitHub Release.

1. Open this repository's [Releases page](https://github.com/QCYTSN/vibe-trading-desktop-handoff/releases).
2. Download `Vibe-Trading-Desktop-Community-0.3.0-x64.exe` and the matching checksum file when the release is available.
3. Verify the SHA-256 checksum:

   ```powershell
   Get-FileHash .\Vibe-Trading-Desktop-Community-0.3.0-x64.exe -Algorithm SHA256
   ```

4. Run the installer. The current build is unsigned, so Windows may display a SmartScreen warning.
5. Complete the first-run disclosure and configure your own provider/API credentials in Settings.

Never publish API keys in issues, logs, screenshots, or repository files.

## Validation status

Completed on the development machine:

- 282/282 frontend tests pass.
- 57/57 focused backend settings, channel, QVeris, and runtime-metadata tests pass.
- Embedded-backend smoke testing passes.
- Interface smoke testing covers 68 OpenAPI operations plus the main page, static assets, SSE ticket flow, and configuration paths.
- The unpacked desktop application starts, loads the interface, exits normally, and leaves no packaged Electron or Python process behind.
- Core provider/model discovery, chat completion, tool-call visibility, detailed replies, and task completion were manually exercised with a live API.
- An earlier installer build was successfully installed and launched in a clean Windows 11 VirtualBox environment.

Still required before calling the release production-ready:

- Repeat clean-Windows installation and regression testing with the exact final `0.3.0` artifact.
- Perform a real `0.3.0 -> 0.3.1` updater test, including download, verification, restart, and rollback behavior.
- Test optional IM adapters with real accounts and their platform-specific SDKs or credentials.
- Complete a final dependency-license, bundled-binary, and SBOM audit.

## Known limitations and remaining work

- **Unsigned installer:** SmartScreen warnings are expected until the project owner adopts a suitable Windows code-signing process.
- **Package size:** the installer is approximately 316 MiB and the unpacked application approximately 1.17 GiB. Python scientific and connector dependencies are the main optimization target.
- **Cold paths:** route preloading improves repeated navigation, but cold start and first-use latency need further profiling and staged loading.
- **Updater:** update plumbing exists, but live updates are disabled until a stable release repository and signed release process are defined.
- **Optional integrations:** not every connector or IM adapter can work without its own SDK, account, key, or regional availability.
- **Windows only:** macOS and Linux packaging have not been implemented.
- **Source provenance:** the working baseline came from an extracted source snapshot without `.git` history. The `source-overlay` directory preserves the resulting files but is not a clean upstream commit series.

## Repository layout

```text
.
├── README.md
├── README.en.md
├── README.zh-CN.md
├── LICENSE
├── NOTICE
├── preview/
│   ├── desktop-home.png
│   └── desktop-onboarding.png
├── release-assets/
│   └── SHA256SUMS-0.3.0.txt
└── source-overlay/
    ├── .github/       # draft Windows release workflow
    ├── agent/         # backend additions and changes
    ├── assets/        # desktop assets and notices
    ├── desktop/       # Electron host and packaging
    └── frontend/      # desktop-focused UI changes
```

The installer is kept out of normal Git history and should be attached to a GitHub Release. Generated runtimes, dependency directories, and build outputs are also intentionally excluded.

## Development and build notes

> **Build prerequisite:** this public repository is an overlay, not a complete buildable Vibe-Trading checkout. Before running any build command, restore these files into a complete upstream Vibe-Trading source tree matching the `0.1.11` baseline. Do not build from the standalone overlay repository root.

After restoring the overlay, the desktop build expects:

- Node.js/Bun dependencies for the React and Electron projects
- Python 3.12 and the Vibe-Trading backend dependencies
- A built frontend under the backend's static-serving path
- An assembled isolated Python runtime for distributable Windows builds

For upstream contribution, rebuild the work on a real fork from a known upstream commit and split the work into reviewable commits. Do not treat this snapshot overlay as a ready-to-merge pull request.

## Security and financial-risk notice

- Use only API keys and trading accounts you control.
- Prefer restricted, revocable credentials and the lowest required permissions.
- Validate strategies, market data, limits, and connector behavior before any real trading.
- Model output may be incorrect, incomplete, delayed, or unsuitable for a specific market.
- The application name and repository do not imply endorsement by HKUDS or any data/model provider.

## License

The project remains under the upstream MIT License. Vibe-Trading and its original work belong to their respective authors. Redistributed third-party components remain subject to their own licenses and notices.
