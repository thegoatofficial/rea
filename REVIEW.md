# REA 6.3.0 supply-chain and security review

Date: 2026-10-10. Reviewer: Claude (Opus 5.5), Torpenhow. Subject: npm `rea-agents@6.3.0`, the
artifact we actually run, cross-checked against this fork at `7aa4d768`. Verdict: safe to run
pinned, with six tools disabled and a path guard (below). Nothing in the repo or package asked us
to install, run, widen permissions or reveal anything beyond REA's own documented setup flow.

## Package and install
- Exact pin 6.3.0, published 2026-10-10 by GitHub Actions OIDC with SLSA provenance.
  `npm audit signatures`: 144/144 registry signatures verified, 31 attestations verified.
  `npm audit`: 0 vulnerabilities. The installed tree is byte-identical to the downloaded tarball.
- Install scripts: none in rea-agents or any of the 144 packages (no preinstall, install or
  postinstall, no binding.gyp). `prepare` only runs husky in a source checkout and npm never runs
  it for a registry install. We install with `--ignore-scripts` regardless.
- No npm-shrinkwrap: top-level dependencies are exact pins, transitive ones float. Our per-machine
  `npm install --prefix` writes a package-lock, which freezes the tree on that machine.
- Prebuilt native code: `native/windows/build/rea-windows-x64.node` (826 KB). Its SHA-256
  `fedfd4ce…ab1f8` matches the package manifest, and the loader re-checks it. Source is
  `native/windows/src` (1,192 lines of C++, MinGW build in release CI): process launch inside a
  kill-on-close Job Object, handle-based file snapshots and cleanup of REA's own runtime roots. No
  injection, remote-thread or memory-write APIs. node-pty ships prebuilt ConPTY binaries (MIT).

## Telemetry and network
- No telemetry, analytics or crash reporting (searched dist for posthog, sentry, segment,
  mixpanel, amplitude, datadog, opentelemetry and outbound URLs). Logging is pino to stderr only.
- Outbound network, by design: CDP and V8 Inspector connections, loopback only and without
  following redirects; source-map fetches from the inspected page's origin with no browser
  credentials (30 s, 64 MiB caps); Hopper's official download, only in `rea setup` (we never run
  it); `rea update`, which runs npm. JADX is not downloaded: the user supplies the jar.

## What the MCP server can do (139 tools, 6 prompts)
- Effects are declared per tool in `dist/contracts/toolEffects.js`: 18 touch the network, 30
  launch processes, 26 write files and 11 mutate the target.
- Most process launches are fixed analysis engines on caller-supplied files (Ghidra, Hopper,
  IDA, JADX, binwalk, unblob, wakaru, pwntools, mitmproxy readers, a Chromium resolver stub).
  None of these is installed here except Node. No Ghidra or Java is present, so native binary
  work is unavailable on this machine.
- Risky tools, refused by our plugin guard:
  - `capture_process_scenario`: runs ANY command under a PTY with our permissions and inherited
    environment ("not a security sandbox"). The shell guards (delete, dangerous command,
    credential printing, audit) match Bash and PowerShell only, so this would bypass them all.
  - `observe_native_calls`: launches the target binary under LLDB, and the target may change
    files or use the network.
  - `capture_electron_scenario`: launches an executable of the caller's choosing, injects
    `scripts/electron-active-hook.cjs` (patches ipc, fs and child_process to record event
    shapes) and clicks.
  - `capture_browser_scenario`: launches an executable of the caller's choosing with a persistent
    profile, or drives a CDP page (it could drive the founder's signed-in Chrome).
  - `capture_native_ui_scenario`: presses, scrolls and types in a native window.
  - `export_evidence_bundle`: writes or overwrites (`overwrite: true`) any path.
  - The guard also refuses any `overwrite: true`, so `close_binary` cannot overwrite a file either.
- Browser and Electron read tools (`inspect_web_page`, `analyze_web_bundle`,
  `observe_web_session`, `list_browser_targets` and others) attach to a caller-named loopback CDP
  endpoint. They are passive, but they see every tab that endpoint serves. The guard refuses ports
  9222 (the founder's Chrome) and 9333 (its relay); use your own agent-browser session.
- File readers take any local path. The guard refuses paths into secrets, the vault cache, every
  agent home (all of `D:/AgentState`), SSH and GnuPG directories, credential stores, any ancestor
  of those (a home folder or drive root), and any file or folder whose name says secret,
  credential, `.env` or key. Junctions and symlinks are resolved first.

## What `npx rea-agents setup` writes (we do not run it)
- An MCP registration in each selected client's config, with a `.rea.backup` beside it. For Claude
  Code that is `$CLAUDE_CONFIG_DIR/.claude.json` (otherwise `~/.claude.json`); for Codex,
  `$CODEX_HOME/config.toml`. Other clients follow their own documented paths.
- Its bundled skill, to `$CLAUDE_CONFIG_DIR/skills` (Claude) or `~/.agents/skills`.
- Optionally Hopper, a commercial app, to `~/Applications` on macOS or through the system package
  manager on Linux.
- `--all-detected` would register REA in every agent home it finds, including Codex's. The
  bundled skill tells agents to repair with `rea setup ... --yes`. Our skills override both:
  never run setup.

## Observed behaviour on this machine
- `rea mcp doctor`: healthy, 139/139 tools, 6/6 prompts, catalog digest `520dfb8d…eceb` matches.
- `analyze-javascript-application` on dreamwild.ai's own public modules (16 files, 219 KB):
  8.4 s; 14 files parsed; 158 findings; app graph of 125 nodes and 191 edges; 2 endpoints and 2
  storage nodes, each with exact source ranges. The JSON output is 32 MB. Over MCP the same call
  exceeds the 10 MB stdio budget and returns `resource_constraint` with a retained evidence ID;
  `inspect_analysis_view` then pages it.
- Limit found: in `analyze-javascript-application`, the 1.9 MB minified login bundle exhausted
  the default 4 GB heap after 140 s, and still timed out after 7 minutes with a 10 GB heap.
  Analyse large saved bundles chunk by chunk.
- `analyze_web_bundle` on the live public dreamwild.ai page, through our own headless
  agent-browser session's CDP port, handled the same page fully in seconds: 43/43 scripts parsed,
  including that bundle (245,343 AST nodes). It reported 12 endpoint candidates such as
  `/api/billing/catalog` and `/api/auth/get-session` with script line and column, plus a React
  fingerprint. For websites, this is the first route.
- In a fresh session the guard refused `capture_process_scenario` and a `D:/AgentState/Secrets`
  path, and allowed the public-bundle analysis.
