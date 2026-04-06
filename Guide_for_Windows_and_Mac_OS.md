# Guide for Running cc-src on Windows and macOS

## Executive Summary

`dist/cli.js` is pure JavaScript bundled for the Bun runtime — it contains no platform-specific compilation. However, **native dependencies** in `node_modules/` are platform-bound. You cannot copy `node_modules/` from Linux to macOS or Windows. Each platform requires a fresh `bun install` and `bun build`.

---

## Cross-Platform Compatibility

| Component | Linux (x86_64) | macOS (ARM/x86) | Windows (x86_64) |
|-----------|---------------|-----------------|------------------|
| `dist/cli.js` | ✅ Pure JS | ✅ Pure JS | ✅ Pure JS |
| `src/` source | ✅ | ✅ | ✅ |
| `node_modules/` | ⚠️ Linux binaries only | ❌ Must reinstall | ❌ Must reinstall |
| `vendor/rg` (ripgrep) | ✅ Linux binary | ❌ Need Darwin binary | ❌ Need `rg.exe` |
| Stubbed packages | ✅ Pure JS | ✅ Pure JS | ✅ Pure JS |

---

## Setup Instructions

### macOS

```bash
# 1. Install Bun
brew install bun

# 2. Install dependencies (resolves macOS-native bindings)
bun install --no-optional

# 3. Build for production
bun build src/entrypoints/cli.tsx --outdir=dist --target=bun

# 4. Copy ripgrep binary (optional — download macOS version)
# Download from: https://github.com/BurntSushi/ripgrep/releases
cp rg dist/vendor/rg 2>/dev/null || true

# 5. Verify
bun dist/cli.js -v
# Expected: 2.1.87 (Claude Code)
```

### Windows

```powershell
# 1. Install Bun (PowerShell as Administrator)
powershell -c "irm bun.sh/install.ps1 | iex"

# 2. Install dependencies (resolves Windows-native bindings)
bun install --no-optional

# 3. Build for production
bun build src/entrypoints/cli.tsx --outdir=dist --target=bun

# 4. Copy ripgrep binary (optional — download Windows version)
# Download from: https://github.com/BurntSushi/ripgrep/releases
# Place rg.exe in dist/vendor/

# 5. Verify
bun dist/cli.js -v
# Expected: 2.1.87 (Claude Code)
```

---

## Known Platform-Specific Risks

### 1. Native Module Bindings

The following packages have platform-specific prebuilds:

| Package | Purpose | Risk if Cross-Copied |
|---------|---------|---------------------|
| `sharp` | Image processing/resizing | `Error: Cannot find module sharp-linux-x64` |
| `tree-sitter` | Bash AST parsing | Parser fails to load |
| `node-pty` | Terminal pseudo-TTY | TTY creation fails |
| `@rollup/rollup-*` | Build-time bundler | Build fails |

**Mitigation**: Always run `bun install` on the target platform. Bun automatically downloads the correct prebuild for the current OS/architecture.

### 2. Ripgrep Binary

`vendor/rg` is a Linux x86_64 binary. Without the correct platform binary:

- **macOS**: Download `ripgrep-*-x86_64-apple-darwin.tar.gz` or `ripgrep-*-aarch64-apple-darwin.tar.gz` (Apple Silicon)
- **Windows**: Download `ripgrep-*-x86_64-pc-windows-msvc.zip` and extract `rg.exe`

**Fallback**: If ripgrep is missing, `GrepTool` falls back to a slower JavaScript-based search. Functionality is preserved but performance degrades on large codebases.

### 3. Windows Path Separators

The codebase uses `path.join()` and `path.resolve()` throughout, which correctly handle `\` (Windows) vs `/` (Unix). No known path-related breakage has been identified.

### 4. TTY and Terminal Behavior

| Feature | Linux | macOS | Windows |
|---------|-------|-------|---------|
| ANSI colors | ✅ Full | ✅ Full | ✅ Windows Terminal / ConHost VT |
| Mouse events | ✅ | ✅ | ⚠️ Limited |
| Kitty keyboard protocol | ✅ | ✅ | ❌ Not supported |
| Text selection | ✅ | ✅ | ⚠️ Terminal-dependent |
| Interactive REPL | ✅ | ✅ | ✅ Windows Terminal recommended |
| Print mode (`-p`) | ✅ | ✅ | ✅ |

**Recommendation**: Use **Windows Terminal** (not legacy `cmd.exe` or PowerShell console) for the best TUI experience on Windows.

### 5. Shell Execution

- **macOS**: Default shell is `zsh` — fully supported by BashTool's security pipeline
- **Windows**: Default shell is `PowerShell` — `PowerShellTool` handles execution, but the BashTool security pipeline (AST analysis, tree-sitter parsing) only applies to bash commands. PowerShell commands go through a separate permission flow.

---

## What You Can and Cannot Copy

| Action | Works? | Notes |
|--------|--------|-------|
| Copy `dist/cli.js` only | ❌ | Needs `node_modules/` and `vendor/` |
| Copy `dist/` + `node_modules/` from Linux | ❌ | Native bindings will fail immediately |
| Copy `src/` + `package.json` + fresh `bun install` + `bun build` | ✅ | **Recommended approach** |
| Copy built `dist/` to same-OS machine | ✅ | Works if OS/arch match exactly |
| Print mode (`-p`) on any platform | ✅ | No TTY dependency |
| Interactive REPL on Windows | ⚠️ | Works in Windows Terminal, some keybindings differ |

---

## Troubleshooting

### Native Module Load Error

```
Error: Cannot find module 'sharp-darwin-arm64'
```

**Fix**: Run `bun install --no-optional` on the target machine. Do not copy `node_modules/` from another platform.

### Ripgrep Not Found

```
Warning: ripgrep binary not found. GrepTool will use fallback search.
```

**Fix**: Download the platform-specific ripgrep binary from [BurntSushi/ripgrep releases](https://github.com/BurntSushi/ripgrep/releases) and place it in `dist/vendor/rg` (macOS/Linux) or `dist/vendor/rg.exe` (Windows).

### TTY Not Detected (Windows)

```
Error: TTY not available. Falling back to non-interactive mode.
```

**Fix**: Run in **Windows Terminal** (not legacy console). Ensure VT processing is enabled:
```powershell
Set-ItemProperty -Path 'HKCU:\Console' -Name 'VirtualTerminalLevel' -Value 1
```

### Build Fails on Windows

```
error: unresolved import
```

**Fix**: Ensure file paths use forward slashes in imports. The codebase uses `.js` extensions (ESM resolution). Windows is case-insensitive, but Bun's resolver may still fail on case mismatches. Verify import paths match actual file casing.

---

## Quick Reference

```bash
# Universal setup (run on each target platform)
bun install --no-optional
bun build src/entrypoints/cli.tsx --outdir=dist --target=bun
bun dist/cli.js -v
```

**Expected output**: `2.1.87 (Claude Code)`

---

*Guide validated against cc-src v2.1.87 source dump. Last updated: 2026-04-06*
