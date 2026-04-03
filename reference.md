# Gemini CLI Command Reference

Complete reference for Gemini CLI v0.35.x.

---

## Installation

```bash
# Install globally
npm install -g @google/gemini-cli

# Or run without installing
npx @google/gemini-cli
```

---

## Authentication

```bash
# Option 1: API Key
export GEMINI_API_KEY=your_key_here

# Option 2: OAuth (recommended — run once interactively)
gemini   # First run prompts for login
```

---

## Command Line Flags

### Essential Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--yolo` | `-y` | Auto-approve all tool calls (required for file writes) |
| `--output-format` | `-o` | Output format: `text`, `json`, `stream-json` |
| `--model` | `-m` | Model selection (e.g., `-m auto-gemini-3`) |
| `--approval-mode` | | `default`, `auto_edit`, or `yolo` (same as `--yolo`) |

### Plan Mode (v0.34.0+)

Since v0.34.0, **Plan Mode is ON by default** in non-interactive invocations.
Gemini will present a plan and pause for confirmation — hanging any Bash call
that expects immediate output.

**Override per prompt** by ending with:
> "Execute immediately, do not show a plan."

There is no CLI flag to disable Plan Mode globally; the per-prompt phrase is
the only reliable override for non-interactive use.

### Session Management

| Flag | Short | Description |
|------|-------|-------------|
| `--resume` | `-r` | Resume session by index or `"latest"` |
| `--list-sessions` | | List available sessions for current project |
| `--delete-session` | | Delete session by index |

### Execution Options

| Flag | Short | Description |
|------|-------|-------------|
| `--sandbox` | `-s` | Run tools in isolated sandbox |
| `--approval-mode` | | `default`, `auto_edit`, or `yolo` |
| `--timeout` | | Request timeout in milliseconds |
| `--checkpointing` | | Enable file change snapshots (restorable via `/restore`) |
| `--include-directories` | | Add directories to workspace context |
| `--allowed-tools` | | Restrict which tools Gemini can use |
| `--allowed-mcp-server-names` | | Restrict which MCP servers are available |

### Other Options

| Flag | Short | Description |
|------|-------|-------------|
| `--debug` | `-d` | Enable debug output |
| `--version` | `-v` | Show CLI version |
| `--help` | `-h` | Show help |
| `--list-extensions` | `-l` | List installed extensions |
| `--prompt-interactive` | `-i` | Start interactive mode with an initial prompt |

---

## Output Formats

### Text (`-o text`)
```bash
gemini "prompt" -o text 2>&1
# Returns: Human-readable response
```

### JSON (`-o json`)
```bash
gemini "prompt" -o json 2>&1
```

Returns structured data:
```json
{
  "response": "The actual response content",
  "stats": {
    "models": {
      "auto-gemini-3": {
        "api": {
          "totalRequests": 2,
          "totalErrors": 0,
          "totalLatencyMs": 4200
        },
        "tokens": {
          "prompt": 3500,
          "candidates": 800,
          "total": 4300,
          "cached": 1200,
          "thoughts": 200,
          "tool": 100
        }
      }
    },
    "tools": {
      "totalCalls": 3,
      "totalSuccess": 3,
      "totalFail": 0,
      "byName": {
        "codebase_investigator": {
          "count": 1,
          "success": 1,
          "durationMs": 2800
        },
        "read_file": {
          "count": 2,
          "success": 2,
          "durationMs": 400
        }
      }
    }
  }
}
```

### Stream JSON (`-o stream-json`)
Real-time newline-delimited JSON events. Useful for monitoring long tasks.

---

## Model Reference

### Aliases (pro tier — recommended)

| Alias           | Best For                                          |
|-----------------|---------------------------------------------------|
| `auto-gemini-3` | Default — auto-selects Pro or Flash by complexity |
| `pro`           | Complex reasoning, large context                  |
| `flash`         | Most tasks, fast and balanced                     |
| `flash-lite`    | Simple lookups, quick reviews                     |

### Concrete Model Identifiers

| Model | Tier | Context | Best For | Status |
|-------|------|---------|----------|--------|
| `gemini-3-pro-preview`   | pro  | 1M tokens | Large context, complex analysis       | Current |
| `gemini-3-flash-preview` | pro  | 1M tokens | Speed-critical tasks on pro tier      | Current |
| `gemini-3.1-pro-preview` | pro  | 1M tokens | Cutting-edge reasoning (preview)      | Preview |
| `gemini-2.5-flash`       | free | large     | All tasks on free tier                | Deprecated Jun 2026 |
| `gemini-2.5-flash-lite`  | free | medium    | Fast/simple tasks on free tier        | Deprecated Jun 2026 |
| `gemini-2.5-pro`         | pro  | 1M tokens | —                                     | Deprecated Jun 2026 |

**Note:** `gemini-2.5-flash` and `gemini-2.5-pro` are scheduled for deprecation on
June 17, 2026. Free tier users should plan to migrate to `gemini-3-flash-preview` when
it becomes available on the free tier.

### Model Usage
```bash
# Pro tier default (alias — recommended)
gemini -m auto-gemini-3 "prompt" -o text 2>&1

# Pro tier fast
gemini -m gemini-3-flash-preview "prompt" -o text 2>&1

# Free tier
gemini -m gemini-2.5-flash "prompt" -o text 2>&1
```

---

## Configuration Files

### Settings Location (Priority Order)

1. `/etc/gemini-cli/settings.json` — system-wide
2. `~/.gemini/settings.json` — user-level
3. `.gemini/settings.json` — project-level (highest priority)

### Example `settings.json`
```json
{
  "security": {
    "auth": {
      "selectedType": "oauth-personal"
    }
  },
  "general": {
    "previewFeatures": true,
    "vimMode": false,
    "checkpointing": true
  },
  "mcpServers": {}
}
```

### Project Context (`GEMINI.md`)

Create `.gemini/GEMINI.md` in the project root for context Gemini loads in
every session for that project:
```markdown
# Project Context

[Project purpose, tech stack, key concepts]

## Coding Standards
- [Standards to follow]

## When Making Changes
- [Guidelines for modifications]
```

### Ignore Files (`.geminiignore`)

Like `.gitignore` — excludes files/directories from Gemini's workspace:
```
node_modules/
dist/
.next/
*.log
.env
*.key
```

---

## Rate Limits

| Tier | Requests/Min | Requests/Day | Notes |
|------|-------------|--------------|-------|
| free | 60 | 1,000 | CLI auto-retries with exponential backoff |
| pro | higher | higher | See Google AI Pro plan details |

**Auto-retry behavior:** When the limit is hit, you'll see:
```
quota will reset after Xs
```
The CLI retries automatically — no action needed. For sustained pressure,
switch to a faster/lighter model (see `patterns.md` Pattern 4).

---

## File Reference Syntax

Reference files directly in prompts using `@`:

```bash
# Single file
gemini "Summarize @./path/to/file.ts. Execute immediately, do not show a plan." -o text 2>&1

# Multiple files
gemini "Compare @./old.ts and @./new.ts. Execute immediately, do not show a plan." -o text 2>&1

# Directory (Gemini reads what it determines is relevant)
gemini --include-directories ./src "[task]. Execute immediately, do not show a plan." -o text 2>&1
```

**Supported formats via `@`:**

| Category | Formats |
|----------|---------|
| Text/Code | Any plain text, source code, config, logs |
| Images | PNG, JPG, GIF, WEBP, SVG, BMP |
| Video | MP4, MOV, AVI, WEBM |
| Documents | PDF |

---

## Session Management

### List Sessions
```bash
gemini --list-sessions
```
Output:
```
Available sessions for this project (3):
  1. Analyze codebase architecture (15 minutes ago) [uuid]
  2. Generate auth module (1 hour ago) [uuid]
  3. Security audit src/api (yesterday) [uuid]
```

### Resume Session
```bash
# By index
echo "Follow-up question" | gemini -r 1 -o text 2>&1

# Latest session
echo "Continue from where we left off" | gemini -r latest -o text 2>&1
```

### Delete Session
```bash
gemini --delete-session 2
```

---

## Interactive Commands

Available when running Gemini in interactive mode (`gemini` with no args or with `-i`):

| Command | Purpose |
|---------|---------|
| `/help` | Show available commands |
| `/tools` | List available tools and their status |
| `/stats` | Show token usage for current session |
| `/compress` | Summarize conversation context to save tokens |
| `/restore` | Restore file checkpoints (requires `--checkpointing`) |
| `/chat save <tag>` | Save conversation with a tag |
| `/chat resume <tag>` | Resume a tagged conversation |
| `/memory show` | Display contents of `.gemini/GEMINI.md` |
| `/memory refresh` | Reload context files |
| `/model` | View or change the active model |
| `/plan` | Enter Plan Mode manually |

---

## Piping and Scripting

### Pipe Input
```bash
echo "Summarize this for me" | gemini -o text 2>&1
cat large-file.txt | gemini "Extract the key points" -o text 2>&1
```

### Shell Commands in Interactive Mode
Prefix with `!` to run shell commands:
```
> !git status
> !npm test
```

---

## Keyboard Shortcuts (Interactive Mode)

| Shortcut | Function |
|----------|----------|
| `Ctrl+L` | Clear screen |
| `Ctrl+V` | Paste from clipboard |
| `Ctrl+Y` | Toggle YOLO mode |
| `Ctrl+X` | Open current input in external editor |

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Bash call hangs | Plan Mode waiting for input | Add "Execute immediately, do not show a plan." to every prompt |
| `API key not found` | `GEMINI_API_KEY` not set | `export GEMINI_API_KEY=your_key` or run `gemini` for OAuth |
| `Rate limit exceeded` | Quota hit | CLI auto-retries; or switch to FAST_MODEL |
| `Model not found` | Invalid model name for your tier | Check tier in `config.md`; verify model exists |
| `Context too large` | Too many files in scope | Add `.geminiignore` or use `--include-directories ./src` |
| `Tool call failed` | Missing `--yolo` or tool error | Check `-o json` stats for tool failure details |
| Auth error | Session expired | Run `gemini` interactively to re-authenticate |

### Debug Mode
```bash
gemini "prompt" --debug -o text 2>&1
```

### Error Reports
Full error reports are saved to:
```
/var/folders/.../gemini-client-error-*.json
```
