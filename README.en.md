# Vibe-Trading Desktop Community — Historical Handoff

[简体中文](README.zh-CN.md) | [Short overview](README.md)

## Status

This repository preserves the original source-first evaluation handoff for an
unofficial Windows desktop prototype. It is no longer the active development
repository.

The work was rebuilt from a real fork of current upstream source, split into
bounded pull requests, and moved upstream:

| Pull request | Scope | Status |
| --- | --- | --- |
| [#923](https://github.com/HKUDS/Vibe-Trading/pull/923) | Electron shell, loopback authentication, diagnostics, lifecycle, watchdog, localization | Merged |
| [#924](https://github.com/HKUDS/Vibe-Trading/pull/924) | Provider/model discovery, model selection, runtime identity and response duration, reply-copy fix | Merged |
| [#1015](https://github.com/HKUDS/Vibe-Trading/pull/1015) | Windows NSIS packaging, isolated Python 3.12 runtime, Electron safeStorage | Draft review |

The active source of truth is now
[HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading) and the
[QCYTSN fork](https://github.com/QCYTSN/Vibe-Trading).

## Why this repository is archived

The `source-overlay/` directory came from an old 0.1.11 source snapshot without
the upstream Git history. It was useful for demonstrating the prototype, but
it cannot provide a clean merge base and is now obsolete. Building or extending
that overlay would recreate already-solved review and synchronization problems.

This repository therefore remains only as provenance for the proposal,
screenshots, and early validation notes. It is not:

- the current Vibe-Trading source;
- an official HKUDS desktop client;
- an installer distribution channel;
- a supported updater feed;
- a branch that should receive new product changes.

## Current packaging boundary

The active Windows packaging PR intentionally includes only a small base
runtime:

- current React/FastAPI application;
- Electron desktop host;
- isolated checksum-pinned CPython 3.12 runtime;
- NSIS packaging;
- Windows user-bound encrypted credential storage;
- clean-runner source and packaged lifecycle tests.

Optional IM/channel packages, personal WeChat/Weixin QR pairing, broker extras,
an updater, and a release feed remain outside the default build.

## Release gates still open

1. Complete upstream review of the packaging pull request.
2. Agree on the Authenticode certificate owner and long-term release owner.
3. Build and validate a signed installer on clean Windows.
4. Run a real signed 0.3.0 to 0.3.1 update on a clean machine before enabling
   any updater.
5. Continue independent package-size, cold-start, and long-term desktop-runtime
   comparisons.

No executable is published from this archive. Any unsigned installer created
during review is intentionally local/CI-only and must not be released to users.

## License and naming

The prototype remains **Vibe-Trading Desktop (Unofficial Community Build)**
until HKUDS explicitly decides otherwise. Vibe-Trading and its original work
belong to the upstream authors. The preserved code remains under the upstream
MIT license and third-party components retain their own licenses.
