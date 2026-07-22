# Vibe-Trading Desktop Community 0.3.0 Handoff

Last updated: 2026-07-21

## 1. Overview

This is a Windows-focused community desktop prototype built around [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading). It does not replace the existing application. It adds an Electron lifecycle host, an embedded Python runtime, a Windows installer, credential storage, a release/update path, and a small set of supporting frontend and backend changes around the existing React + FastAPI + Python Agent architecture.

The current product name is **Vibe-Trading Desktop Community**. It must not be represented as an official HKUDS client unless the maintainers explicitly approve that status.

- Upstream source baseline: Vibe-Trading `0.1.11`
- Desktop layer version: `0.3.0`
- Current target: Windows 10/11 x64
- Packaging: Electron + NSIS `.exe`
- Backend: embedded relocatable Python 3.12 directory, not PyInstaller `--onefile`

The original working copy was extracted source without `.git` history. This handoff is therefore a path-preserving **source overlay**, not a Git patch. Upstream `main` has continued moving after 0.1.11, so these changes must be replayed and reviewed against current `main` instead of blindly overwriting newer files.

## 2. Handoff contents

```text
Vibe-Trading-Desktop-Handoff-0.3.0/
├── README.zh-CN.md
├── README.en.md
├── release-assets/               # local Release upload staging
│   ├── Vibe-Trading-Desktop-Community-0.3.0-x64.exe
│   └── SHA256SUMS-0.3.0.txt
├── preview/
│   └── desktop-0.3.0-window.png
└── source-overlay/
│   ├── .github/workflows/
│   ├── desktop/electron/
│   ├── frontend/src/
│   ├── agent/src/
│   ├── agent/tests/
│   ├── assets/icon.png
│   ├── LICENSE
│   └── NOTICE
```

Generated or reproducible data is intentionally excluded: `node_modules`, the expanded embedded Python runtime, caches, build logs, frontend/TypeScript output, and `win-unpacked`. The local working copy stages the Alpha EXE under `release-assets/`, but `.gitignore` prevents that 316 MiB binary from entering Git history. Publish it as a GitHub Release asset for hands-on evaluation.

The included installer is:

```text
release-assets\Vibe-Trading-Desktop-Community-0.3.0-x64.exe
```

SHA-256:

```text
c27dfd2408c5b1218c948a7f775e948b9885c548a072ce6f426fa6099f88e3d1
```

The installer is approximately 316 MiB and the unpacked application is approximately 1.17 GiB. The current artifact is unsigned and should only be used for internal acceptance or clearly labeled Alpha testing.

### Fast maintainer evaluation

1. Use Windows 10/11 x64.
2. Download the installer from the GitHub Release and verify it against `release-assets/SHA256SUMS-0.3.0.txt`.
3. Run the installer. A Windows SmartScreen warning is expected because this Alpha is not yet Authenticode-signed.
4. Launch **Vibe-Trading Desktop Community** and review the first-run disclosure.
5. Configure a provider, model, and API key in Settings. No user credentials are included in this handoff.
6. Exercise chat, real runtime model display, elapsed time, reply copy, main routes, and the IM Channel Center.
7. Close the window and confirm that the owned embedded Python backend also exits.

Use the repository for code review and the GitHub Release installer for hands-on evaluation.

## 3. Implemented work

### Desktop lifecycle and packaging

- Electron owns the embedded FastAPI/Python backend process.
- A random loopback port and per-launch `API_AUTH_KEY` are used.
- The window waits for `/health` and provides loading and startup-failure states.
- Single-instance enforcement and graceful shutdown with owned process-tree cleanup.
- Sandboxed renderer, disabled Node integration, and a narrow preload IPC surface.
- Windows NSIS installer, Start Menu/Desktop shortcuts, and the upstream project icon.
- Embedded Python 3.12, built frontend, and a minimal GTK/Pango closure for PDF/WeasyPrint support.
- Upstream MIT license, notice, and desktop privacy/security/release documents are bundled.

### Credentials and local security

- Electron `safeStorage` provides Windows DPAPI-backed encryption for supported secrets.
- Migration covers supported LLM keys, Tushare, QVeris, personal WeChat, and top-level IM channel secret fields.
- Secrets are injected into the child backend environment and are not intentionally printed to desktop logs.
- First-run acknowledgement covers community status, local data, model limitations, and live-trading risk.

This is not a complete security audit. Public release still requires a connector-by-connector secret inventory, log-redaction review, dependency review, and signed update-chain review.

### Models and chat UX

- Provider model discovery after a key is configured.
- All discovered models remain visible in the picker while custom model entry remains possible.
- Picker styling and behavior are aligned with the surrounding Settings controls.
- Chat displays the real configured provider, model, and reasoning level.
- Replies display elapsed time.
- Backend captures and persists the provider-returned `model` field instead of trusting model self-identification.
- Desktop clipboard IPC fixes the reply copy button.
- Temperature/reasoning guidance was added without changing the core financial-agent system prompt.

### Frontend loading

- Main route modules are preloaded to reduce first-navigation waiting.
- The React UI remains same-origin behind FastAPI, preserving REST, SSE, uploads, and SPA deep links.
- First-run gate, desktop update settings, desktop environment types, and local asset/font improvements.

### IM channel center

- A registry-driven desktop channel center covers all built-in adapters, not only WeChat.
- WebSocket, Telegram, Slack, Discord, Matrix, WhatsApp, Signal, QQ/NapCat, WeChat, WeCom, Feishu/Lark, DingTalk, Teams, email, and Mochat are represented.
- Availability, missing dependencies, generated configuration fields, enabled/runtime status, and recovery hints are shown.
- Start/stop and pairing controls are supported; personal WeChat has a dedicated QR-login flow.
- Adapters that require SDK extras, bot/business accounts, or platform credentials remain explicitly conditional.

### Updates and CI release flow

- Manual update check, download progress, and restart-to-install UX.
- GitHub Releases assets: installer, `latest.yml`, and `.blockmap`.
- Windows GitHub Actions workflow builds the frontend and embedded backend, runs smoke checks, packages the app, writes SHA-256, and creates a draft release.
- The current workflow supports traditional certificate secrets through `CSC_LINK` and `CSC_KEY_PASSWORD`.
- Local 0.3.0 intentionally has no update repository bound. CI binds the selected repository when a fork or upstream release home is chosen.

Never replace a published binary under the same version. Publish a higher version for every changed artifact.

## 4. Validation status

Completed:

- 282/282 frontend tests pass.
- 57/57 focused backend settings/channel/QVeris/runtime-metadata tests pass.
- Embedded backend smoke startup passes; health is reached in about 6.6 seconds.
- Interface smoke covers 68 OpenAPI operations plus main UI/API routes, assets, and SSE ticket behavior.
- The unpacked Windows app starts the embedded backend and renders the first-run UI.
- Closing the window leaves zero Electron/Python processes.
- An earlier build passed clean Windows 11 VM installation and main route/LLM/tool-call acceptance.

Still required:

- Final 0.3.0 clean Windows 11 snapshot installation regression.
- Real GitHub repository `N -> N+1` updater test.
- Trusted Authenticode signing and SmartScreen observation.
- Real-account end-to-end coverage for all 16 IM adapters.
- Complete third-party license/SBOM review.
- A second focused pass on installer size and cold navigation/startup performance.

The full local Python suite was blocked by an Anaconda NumPy 2.3.5 versus older pandas/pyarrow ABI mismatch. That is not a known failure caused by this overlay, but the full suite must be rerun in an isolated lockfile-based environment or upstream CI before a public PR.

## 5. Reconstructing reviewable Git history

Start from a real clone:

```powershell
git clone https://github.com/HKUDS/Vibe-Trading.git
cd Vibe-Trading
git switch -c feat/windows-desktop-shell
```

Replay the overlay in reviewable slices:

1. Independent `desktop/electron/` shell.
2. Windows release workflow.
3. Minimal backend API/runtime metadata changes.
4. Frontend Settings/model/chat/channel changes.
5. Tests and documentation.

Run focused checks for each slice. Upstream requires a DCO `Signed-off-by:` trailer on every community commit:

```powershell
git commit -s -m "feat(desktop): add Windows Electron shell"
```

The upstream contribution guide also says not to add AI `Co-Authored-By` trailers or AI attribution lines to commits or PR descriptions.

## 6. Local build

```powershell
cd frontend
npm ci
npm run build

cd ..\desktop\electron
npm ci
npm run runtime:win
npm run smoke:backend
npm run smoke:interfaces
npm run pack:win
npm run installer:win
npm run verify:release
```

Important outputs:

- `release/win-unpacked/`
- `release/Vibe-Trading-Desktop-Community-<version>-x64.exe`
- `release/latest.yml`
- `release/*.blockmap`
- `release/SHA256SUMS.txt`

## 7. Windows code signing

### Ownership first

Do not buy or provision a certificate before deciding who owns the product name, release repository, and updater feed. If upstream accepts an official desktop client, HKUDS or its approved legal entity should own the signing identity and CI authorization. Private keys or PFX files must never be committed or casually transferred between contributors.

### Practical options

1. **SignPath Foundation** — the first option to investigate for a qualifying public open-source project. It offers sponsored managed signing and keeps signing inside a controlled pipeline: https://signpath.io/solutions/open-source-community
2. **Commercial OV certificate** — purchase from a CA in the Microsoft Trusted Root Program. The CA validates the publisher; modern OV private keys normally require a hardware token or cloud HSM. Confirm current regional, individual/organization, and CI support with the CA before purchase.
3. **Azure Artifact Signing** — Microsoft's preferred non-Store cloud service, but current public eligibility is limited to organizations in the US, Canada, EU, and UK, and individuals in the US and Canada. It is not the first choice for an individual publisher in mainland China. If an eligible upstream entity owns it, electron-builder can use `win.azureSignOptions` plus Azure identity variables.
4. **Microsoft Store MSIX** — Microsoft re-signs Store-submitted MSIX packages at no certificate cost. The current NSIS/Python-sidecar package is not MSIX and would require separate Store-policy and filesystem/loopback validation.
5. **Self-signed certificate** — development or managed-enterprise use only. It does not create public Windows trust and does not solve SmartScreen for normal users.

For a traditional PFX workflow, set protected CI secrets `CSC_LINK` and `CSC_KEY_PASSWORD`; electron-builder then signs the app binaries and NSIS installer. Use SHA-256 and an RFC 3161 timestamp, and verify both the installed application and installer:

```powershell
Get-AuthenticodeSignature .\Vibe-Trading-Desktop-Community-0.3.0-x64.exe
signtool verify /pa /all /v .\Vibe-Trading-Desktop-Community-0.3.0-x64.exe
```

A public stable release should report `Valid`. A trusted signature proves publisher identity and file integrity, but new files may still need time and downloads to build SmartScreen reputation. Current Microsoft guidance no longer recommends paying for EV solely to obtain an immediate SmartScreen bypass.

## 8. How to contact the maintainers

The repository README and contribution guide do not publish a project email address. The preferred transparent route is:

1. Open a post in [GitHub Discussions](https://github.com/HKUDS/Vibe-Trading/discussions), category **Ideas**.
2. If there is no response after several days, use the repository's [feature request entry](https://github.com/HKUDS/Vibe-Trading/issues/new/choose).
3. After the maintainers choose a direction, create a focused DCO-signed draft PR from a real fork.

`@warren618` (Haozhe Wu) is visibly active in repository announcements and discussions and can be politely tagged. Do not lead with an unexplained executable or call the project official. Lead with source, architecture, validation, limitations, and the decisions needed from maintainers.

### Maintainer contact template

Suggested Discussion title:

```text
Proposal: Windows desktop shell and installer for Vibe-Trading
```

Suggested body:

```markdown
Hi HKUDS / Vibe-Trading maintainers — @warren618,

I have been experimenting with a Windows desktop distribution for Vibe-Trading and would like to ask for direction before publishing it more broadly or preparing a large pull request.

The prototype keeps the existing React + FastAPI + Python Agent architecture. An Electron host starts an embedded Python 3.12 backend on a random loopback port, waits for `/health`, loads the existing same-origin Web UI, and owns backend shutdown. It currently produces an NSIS installer under the temporary name **Vibe-Trading Desktop Community** and is not presented as an official HKUDS application.

The prototype also includes:

- OS-backed credential encryption through Electron `safeStorage` / Windows DPAPI;
- first-run community, privacy, model-limitation, and trading-risk acknowledgement;
- real provider/model/reasoning display and reply elapsed time;
- improved model discovery and selection;
- a registry-driven Settings UI for all built-in IM adapters, including personal WeChat QR login;
- manual GitHub Releases update checks, blockmap metadata, and a draft-release Windows CI workflow;
- project icon, license/notice bundling, interface smoke checks, and process cleanup.

Current validation includes 282 frontend tests, 57 focused backend tests, an embedded-backend smoke test, 68 OpenAPI operation probes, packaged-window startup, and clean process shutdown. An earlier build was also installed and exercised in a clean Windows 11 VM. The final build still needs a clean-install rerun, a real N-to-N+1 updater test, trusted code signing, and full license/SBOM review.

Before I reconstruct the work as focused DCO-signed commits against current `main`, could you advise on the preferred direction?

1. Should this be proposed as a `desktop/` directory in the main repository, a separate HKUDS companion repository, or remain a community fork?
2. Is the temporary name `Vibe-Trading Desktop Community` and use of the existing project icon acceptable during review?
3. If an official desktop distribution is desirable, who should own Windows code signing and the GitHub Releases update feed?
4. Which optional IM channel SDKs should be bundled by default versus installed on demand?
5. Is the personal WeChat iLink integration an intended supported surface for desktop distribution?
6. Would you prefer to review the independent Electron shell first, followed by the minimal backend/frontend integration patches?

I can share a source overlay, architecture/release notes, checksums, screenshots, and an explicitly unsigned/unofficial Alpha installer for testing. I will preserve the MIT license and follow the repository's DCO and contribution requirements.

Thank you for your work on Vibe-Trading. I would be happy to adapt the implementation to the repository structure and release model you prefer.
```

For the first contact, share a public source branch or the lightweight handoff archive, screenshots, and test results. Offer the installer separately with its SHA-256 and the labels “unsigned Alpha” and “unofficial.”

## 9. Recommended next work

1. Reconstruct focused DCO-signed commits in a real fork and rebase onto current upstream `main`.
2. Obtain upstream direction on repository placement, branding, update ownership, and signing ownership.
3. Run final 0.3.0 clean-Windows installation acceptance.
4. Publish a test `0.3.0 -> 0.3.1` sequence and validate the complete updater flow.
5. Apply to SignPath or provision signing through the confirmed publisher entity.
6. Profile dependencies and split the core runtime from optional connector bundles instead of deleting Python packages blindly.
7. Measure backend cold start, first route navigation, and antivirus scanning independently before optimizing.
8. Produce a third-party license inventory and SBOM.

## 10. License and responsibility

Upstream is MIT licensed. `LICENSE` and `NOTICE` are included in this handoff. Any public distribution must retain upstream attribution and remain clearly labeled Community/Unofficial until HKUDS publicly approves official status.

This is a financial research tool. Model, data, and strategy output are not guaranteed, and the desktop layer must not bypass upstream live-trading authorization, risk, confirmation, or audit boundaries.
