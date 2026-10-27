---
upstream: https://github.com/morluto/rea
kind: github
pinned: rea-agents-6.3.0
commit: 7aa4d768eb15317a63a476431ed75bafec086033
npm: rea-agents@6.3.0, sha512-1l3L22PYqlzW70I2Z8fVjWufdBp8lN5dwXSiBERLKPP7pybiFSHeCD8LnXfAoilYkmSQe1NfRgOrbIr9ujkOfQ==
licence: MIT; root LICENSE and the npm package LICENSE read 2026-10-10 (Copyright (c) 2026 morluto)
fork: https://github.com/thegoatofficial/rea, unmodified mirror; this file and REVIEW.md are intake notes only
---

Taken: the published npm package `rea-agents@6.3.0` (not this checkout's source build), installed
per machine with install scripts disabled. On this machine: `D:/AgentTools/rea` (package.json pins
`rea-agents: 6.3.0`, package-lock.json pins the 144-package tree).

Upstream state at intake (2026-10-10): main `7aa4d768` ("chore(release): sync corrected 6.3
checkpoint"); latest release tag `rea-agents-6.3.0`; npm `latest` 6.3.0 published
2026-10-10T00:02Z by GitHub Actions OIDC with SLSA provenance, gitHead `74cf9e14` (the release
merge, one commit behind main).

Licences of what we run: REA MIT. Bundled third-party notices in the package: evmole MIT,
pwndbg MIT (both READMEs/licences only; the engines are bring-your-own). Dependency tree: 111 MIT,
11 ISC, 7 Apache-2.0, 7 BSD-3-Clause, 1 BSD-2-Clause, 5 BlueOak-1.0.0, pako MIT AND Zlib, sha.js
MIT AND BSD-3-Clause. No GPL or AGPL. Git submodules (binwalk, unblob, wakaru, jadx-headless-mcp,
ghidra-nativeaot) are not in the npm package and were not taken.

Re-sync: read the new release's CHANGELOG and tool catalog, repeat REVIEW.md's checks against the
new tarball, then bump the pin in three places together: `D:/AgentTools/rea/package.json`,
`daedalus-skills/claude/plugins/rea/start-rea.mjs` (`PINNED`) and the plugin version.

Source mirror, 2026-10-28: this branch (`intake/rea-6.8.0`) merges upstream tag `rea-agents-6.8.0`
(`160d821ac9a0310afcb8d1fc427862dd6508add5`, 2026-10-25; MIT, root LICENSE unchanged from 6.3.0) and keeps
the two intake commits above. The runtime pin is NOT moved by this merge: the 6.8.0 npm package
(`sha512-9QDeRnGq...Zww==`) fails the maintainer's audit gate (zod 4.4.3, GHSA-mrhx-c7xc-m3w5, fixed on
upstream main but unreleased) and changes 37 reviewed tool contracts; see claude/plugins/rea/SOURCE.md in
daedalus-skills for the live runtime pin.
