# Vibe-Trading Desktop Community — Archived Handoff

[English](README.en.md) | [简体中文](README.zh-CN.md)

This repository is the historical, source-first handoff that was used to
present an unofficial Windows desktop prototype for
[HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading).

Active development has moved to the real fork and upstream pull requests:

- [#923](https://github.com/HKUDS/Vibe-Trading/pull/923) — desktop shell and
  backend lifecycle, merged;
- [#924](https://github.com/HKUDS/Vibe-Trading/pull/924) — provider/model
  discovery and runtime UX, merged;
- [#1015](https://github.com/HKUDS/Vibe-Trading/pull/1015) — Windows packaging
  and secure credential storage, under review.

> This repository is archived for provenance only. Its `source-overlay/` was
> based on an old 0.1.11 source snapshot and must not be treated as the current
> product source, an upstream-ready patch, or a release channel.

![Vibe-Trading Desktop home](preview/desktop-home.png)

## Current source of truth

Use the current upstream repository and the active fork branch instead:

- upstream: [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading)
- contributor fork: [QCYTSN/Vibe-Trading](https://github.com/QCYTSN/Vibe-Trading)
- packaging branch: `agent/windows-packaging`

No installer is published from this handoff repository. The unsigned installer
produced during packaging review is a local/CI review artifact and is not safe
to distribute as a user release.

## Remaining release gates

- upstream review and merge of the Windows packaging PR;
- an agreed Authenticode signing identity and release owner;
- a signed installer test on a clean Windows machine;
- a separate, real signed 0.3.0 to 0.3.1 updater test before any updater is
  enabled;
- continued work on package size and cold-start performance.

This community work remains unofficial unless HKUDS explicitly changes its
release ownership or branding. Vibe-Trading and its original work belong to
the upstream authors and remain covered by the upstream MIT license.
