# Vibe-Trading Desktop Community

An early Electron desktop host for the existing Vibe-Trading React + FastAPI app.

## Prototype behavior

- Starts `vibe-trading serve` on a dynamically selected loopback port.
- Generates a random per-launch `API_AUTH_KEY` and injects it into the trusted local UI.
- Waits for `GET /health` before loading the application.
- Uses an isolated, sandboxed renderer with Node integration disabled.
- Keeps logs under Electron's per-user logs directory.
- Requests graceful shutdown, then terminates the owned process tree if necessary.
- Prevents duplicate application instances.

Development mode can use an existing Vibe-Trading Python environment. Packaged builds
ship a relocatable Python 3.12 runtime under `resources/backend`; set
`VIBE_TRADING_EXECUTABLE` only when intentionally overriding it.

## Run from source

> **Prerequisite:** this directory is published as part of a source overlay. Before running the commands below, restore the entire overlay into a complete Vibe-Trading checkout matching the upstream `0.1.11` baseline. These commands will not work correctly from the standalone overlay repository.

```powershell
cd desktop\electron
npm install
npm start
```

Press `Alt` to show the native application menu. It contains restart, logs, refresh,
zoom, fullscreen, and developer-tool actions. External OAuth and documentation links
open in the system browser, but the authenticated local application itself stays in the
desktop window.

## Packaging architecture

Do not use PyInstaller `--onefile` for the backend. The distributable milestone should
assemble a relocatable directory runtime under Electron resources:

```text
resources/
  backend/
    python.exe
    Lib/site-packages/...
    frontend/dist/...
    gtk/bin/...              # minimal WeasyPrint/Pango DLL closure
```

`electron-builder` produces an NSIS installer from this directory. Preserve the upstream
MIT license and use community/unofficial branding unless HKUDS approves an official
desktop distribution.

## Verified Windows prototype

Release builds use the upstream `assets/icon.png`, maximum NSIS compression,
and prune third-party `test`/`tests` directories from the embedded Python
runtime. Runtime code, package data, native libraries, and import caches remain
intact. `npm run smoke:interfaces` launches the embedded backend and verifies
the SPA, assets, OpenAPI contract, safe parameterless GET routes, and SSE ticket
authentication.

The current build has been verified end to end on Windows:

- Desktop fast-start reaches `/health` in about 2.4–2.6 seconds in direct smoke tests.
- The assembled backend runtime is about 803.8 MiB, including portable PDF support.
- `release/win-unpacked` is about 1,151.3 MiB including Electron and the backend.
- The packaged executable launched `resources/backend/python.exe`, served the full UI,
  and returned HTTP 200 from both `/health` and `/`.
- The maximum-compression NSIS installer is about 247.7 MiB, 79.6% smaller than the
  original store-mode prototype.

After the overlay has been restored into the complete upstream source tree, run the build commands from `desktop\electron`:

```powershell
npm run runtime:win
npm run smoke:backend
npm run smoke:interfaces
npm run pack:win
npm run installer:win
```

Before publishing a public release, test the installer in a clean Windows VM, configure
code signing, audit bundled third-party licenses, and decide how model credentials
migrate from the upstream `.env` file to OS credential storage.
