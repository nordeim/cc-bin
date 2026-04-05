## Installation & Usage Guide

### Extract the archive
unzip cc-bin-linux-x64.zip

### Make executable (if not already)
mv cc-bin-linux-x64 cc-bin
chmod +x cc-bin

### Run directly
./bin/cc-bin -v

### Add the bin directory to your PATH
echo 'export PATH="/path/to/cc-bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Now 'cc-bin' is available from anywhere
cc-bin -v

### Usage

```bash
# Interactive mode
cc-bin

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

### Usage

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

*Document Version: 1.0*
*Last Updated: 2026-04-05*
*Application Version: 2.1.87 (Claude Code)*
