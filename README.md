## Overview

Two distribution options are provided:

1. **Option A: Shell Wrapper** — Requires `bun` runtime on the target machine (~24MB distribution)
2. **Option C: Native Binary** — Zero dependencies, standalone executable (~117MB)

---

## Distribution Package Contents

### What to Include in the ZIP Archive

#### Option A: Shell Wrapper Distribution (Recommended for users with `bun`)

```
cc-bin/
├── cli.js                    # Bundled application (23.67 MB)
├── vendor/                   # Native binaries (ripgrep)
│   └── ripgrep/
│       └── x64-linux/
│           └── rg
├── bin/
│   └── cc-bin                # Shell wrapper script (executable)
└── package.json              # For global installation via bun/npm
```

**Total Size**: ~24 MB

#### Option C: Native Binary Distribution (Zero dependencies)

```
cc-bin/
├── cc-bin-linux-x64          # Standalone native executable (117 MB)
└── vendor/                   # Native binaries (ripgrep)
    └── ripgrep/
        └── x64-linux/
            └── rg
```

**Total Size**: ~117 MB

---

## Creating the Distribution Package

### Step 1: Build the Application

```bash
# From the cc-src project root
bun build src/entrypoints/cli.tsx --outdir=dist --target=bun
cp -r vendor dist/ 2>/dev/null || true
```

### Step 2: Create the Shell Wrapper (Option A)

```bash
mkdir -p dist/bin
cat > dist/bin/cc-bin << 'WRAPPER'
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")" && pwd)"
CLI_PATH="$SCRIPT_DIR/../cli.js"

if ! command -v bun &> /dev/null; then
  echo "Error: 'bun' runtime is required but not found in PATH."
  echo "Install: https://bun.sh/docs/installation"
  exit 1
fi

if [ ! -f "$CLI_PATH" ]; then
  echo "Error: cli.js not found at $CLI_PATH"
  exit 1
fi

exec bun "$CLI_PATH" "$@"
WRAPPER
chmod +x dist/bin/cc-bin
```

### Step 3: Create package.json (Option A)

```bash
cat > dist/package.json << 'PKG'
{
  "name": "cc-bin",
  "version": "2.1.87",
  "description": "Agentic CLI terminal interface for autonomous software engineering",
  "bin": {
    "cc": "./bin/cc-bin"
  },
  "engines": {
    "bun": ">=1.3.11"
  }
}
PKG
```

### Step 4: Create Native Binary (Option C)

```bash
# Compile to standalone native executable
bun build src/entrypoints/cli.tsx --compile --outfile dist/cc-bin-linux-x64
```

### Step 5: Create ZIP Archive

```bash
# Option A: Shell wrapper
cd dist && zip -r ../cc-bin-v2.1.87-shell.zip cli.js vendor/ bin/ package.json

# Option C: Native binary
cd dist && zip -r ../cc-bin-v2.1.87-native.zip cc-bin-linux-x64 vendor/
```

---

## Installation & Usage Guide

### Option A: Shell Wrapper (Requires `bun`)

#### Prerequisites
- **Bun** runtime v1.3.11+ installed: `curl -fsSL https://bun.sh/install | bash`

#### Installation Methods

**Method 1: Direct Execution**
```bash
# Extract the archive
unzip cc-bin-v2.1.87-shell.zip
cd cc-bin-v2.1.87

# Run directly
./bin/cc-bin -v
# Output: 2.1.87 (Claude Code)
```

**Method 2: Global Installation**
```bash
# Extract and install globally
unzip cc-bin-v2.1.87-shell.zip
cd cc-bin-v2.1.87
bun install -g

# Now 'cc' is available from anywhere
cc -v
# Output: 2.1.87 (Claude Code)
```

**Method 3: Add to PATH**
```bash
# Add the bin directory to your PATH
echo 'export PATH="/path/to/cc-bin-v2.1.87/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Now 'cc-bin' is available from anywhere
cc-bin -v
```

#### Usage

```bash
# Interactive mode
cc-bin
# or
cc

# Non-interactive mode (print mode)
cc-bin -p "Explain this code" --file src/main.ts

# With specific model
cc-bin -p "Hello" --model "nvidia/nemotron-3-super-120b-a12b"

# With OpenAI provider
LLM_PROTOCOL=openai OPENAI_PROVIDER=openrouter OPENROUTER_API_KEY=sk-or-... \
  cc-bin -p "Hello" --model "openai/gpt-4o-mini"

# With NVIDIA provider
LLM_PROTOCOL=openai OPENAI_PROVIDER=nvidia NVIDIA_API_KEY=nvapi-... \
  cc-bin -p "Hello" --model "nvidia/nemotron-3-super-120b-a12b"

# With Anthropic (default)
ANTHROPIC_API_KEY=sk-ant-... cc-bin -p "Hello"

# Resume a previous session
cc-bin --resume <session-id>

# Verbose output
cc-bin -p "Hello" --verbose
```

---

### Option C: Native Binary (Zero Dependencies)

#### Prerequisites
- **None** — The binary is fully self-contained (except for the vendor/ directory)

#### Installation

```bash
# Extract the archive
unzip cc-bin-v2.1.87-native.zip

# Make executable (if not already)
chmod +x cc-bin-linux-x64

# Run directly
./cc-bin-linux-x64 -v
# Output: 2.1.87 (Claude Code)
```

#### Optional: Add to PATH

```bash
# Move to a system directory
sudo mv cc-bin-linux-x64 /usr/local/bin/cc
sudo mv vendor/ /usr/local/lib/cc-vendor/

# Create a wrapper script
cat > /usr/local/bin/cc << 'WRAPPER'
#!/usr/bin/env bash
VENDOR_DIR="/usr/local/lib/cc-vendor"
exec /usr/local/bin/cc --vendor-dir "$VENDOR_DIR" "$@"
WRAPPER
chmod +x /usr/local/bin/cc

# Now 'cc' is available system-wide
cc -v
```

#### Usage

Same as Option A, but use the binary name:

```bash
# Interactive mode
./cc-bin-linux-x64

# Non-interactive mode
./cc-bin-linux-x64 -p "Explain this code"

# With environment variables
NVIDIA_API_KEY="..." LLM_PROTOCOL=openai OPENAI_PROVIDER=nvidia \
  ./cc-bin-linux-x64 -p "Hello" --model "nvidia/nemotron-3-super-120b-a12b"
```

---

## Environment Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `ANTHROPIC_API_KEY` | Anthropic API key | `sk-ant-...` |
| `OPENAI_API_KEY` | OpenAI API key | `sk-...` |
| `OPENROUTER_API_KEY` | OpenRouter API key | `sk-or-v1-...` |
| `NVIDIA_API_KEY` | NVIDIA API key | `nvapi-vn-...` |
| `LLM_PROTOCOL` | Protocol to use (`anthropic` or `openai`) | `openai` |
| `OPENAI_PROVIDER` | Provider when using OpenAI protocol | `nvidia`, `openrouter`, `openai` |
| `CLAUDE_CONFIG_DIR` | Custom config directory | `~/.my-cc-config` |

---

## Supported Providers

| Provider | Environment Variables | Example Command |
|----------|----------------------|-----------------|
| **Anthropic** (default) | `ANTHROPIC_API_KEY` | `cc-bin -p "Hello"` |
| **OpenAI** | `LLM_PROTOCOL=openai OPENAI_API_KEY=sk-...` | `LLM_PROTOCOL=openai OPENAI_API_KEY=sk-... cc-bin -p "Hello"` |
| **OpenRouter** | `LLM_PROTOCOL=openai OPENAI_PROVIDER=openrouter OPENROUTER_API_KEY=sk-or-...` | `LLM_PROTOCOL=openai OPENAI_PROVIDER=openrouter OPENROUTER_API_KEY=sk-or-... cc-bin -p "Hello" --model "openai/gpt-4o-mini"` |
| **NVIDIA NIM** | `LLM_PROTOCOL=openai OPENAI_PROVIDER=nvidia NVIDIA_API_KEY=nvapi-...` | `LLM_PROTOCOL=openai OPENAI_PROVIDER=nvidia NVIDIA_API_KEY=nvapi-... cc-bin -p "Hello" --model "nvidia/nemotron-3-super-120b-a12b"` |

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| `bun: command not found` | Install bun: `curl -fsSL https://bun.sh/install \| bash` |
| `cli.js not found` | Ensure `cli.js` is in the same directory as `bin/cc-bin` |
| `rg: command not found` | Ensure `vendor/ripgrep/x64-linux/rg` exists and is executable |
| `API Error: 401` | Check API key is correct for the selected provider |
| `API Error: 402` | OpenRouter credits exhausted — purchase more or use NVIDIA |
| Tool calls not executing | Model may not support tool calling — try `openai/gpt-4o-mini` |

### Debug Mode

For troubleshooting, enable debug logging:

```bash
# File-based debug logging (Ink captures stderr)
export DEBUG=1
./cc-bin-linux-x64 -p "Hello" 2>/tmp/cc-debug.log
cat /tmp/cc-debug.log
```

---

## Platform Support

| Platform | Architecture | Option A | Option C |
|----------|-------------|----------|----------|
| Linux | x86_64 | ✅ | ✅ |
| Linux | ARM64 | ✅ | Build on target |
| macOS | x86_64 | ✅ | Build on target |
| macOS | ARM64 (Apple Silicon) | ✅ | Build on target |
| Windows | x86_64 | ✅ (via WSL) | ❌ |

**Note**: For native compilation on other platforms, run `bun build --compile` on the target platform.

---

## File Sizes

| File | Size | Purpose |
|------|------|---------|
| `cli.js` | 23.67 MB | Bundled application code |
| `cc-bin-linux-x64` | 117 MB | Standalone native executable |
| `vendor/ripgrep/x64-linux/rg` | ~7 MB | Ripgrep binary for Glob tool |
| `bin/cc-bin` | ~600 B | Shell wrapper script |
| `package.json` | ~200 B | Package metadata |

---

## Security Notes

- The application requires API keys to function — never commit keys to version control
- The ripgrep binary (`vendor/ripgrep/x64-linux/rg`) is a pre-compiled binary from the official ripgrep project
- Shell wrapper uses `exec` to replace the shell process — no intermediate shell remains
- Native binary is compiled from source — no external dependencies beyond the vendor directory

---

*Document Version: 1.0*
*Last Updated: 2026-04-05*
*Application Version: 2.1.87 (Claude Code)*
