# gemini-delegate — Claude Code Skill

A Claude Code skill that automatically delegates tasks to Google Gemini CLI when
they benefit from Gemini's unique strengths: 1M token context, real-time Google
Search, multimodal file reading (images, video, PDF), and deep codebase analysis.

---

## What This Skill Does

Claude Code auto-invokes this skill when it detects:

| Situation | Why Gemini Handles It Better |
|-----------|------------------------------|
| Files > ~500 lines | Gemini's 1M token context vs Claude's working range |
| Whole-codebase analysis | `codebase_investigator` tool for deep cross-file mapping |
| Image files (PNG, JPG, GIF, WEBP, SVG, BMP) | Gemini's native multimodal file reading |
| Video files (MP4, MOV, AVI, WEBM) | Gemini's native video understanding |
| PDF documents | Gemini's `read_file` tool handles PDFs natively |
| Real-time web search | `google_web_search` tool (Claude Code has no live internet access) |
| Latest versions, CVEs, current docs | Google Search grounding for up-to-date information |
| Code generation, review, test generation, docs | Second AI perspective for higher confidence output |

You can also explicitly invoke it: *"Use Gemini to analyze this codebase"* or
*"Have Gemini search for the latest Next.js release notes"*.

---

## How It Differs from `gemini-cli`

The `gemini-cli` skill is a broad reference covering Gemini usage with detailed
templates and patterns. `gemini-delegate` is focused on a single workflow:

1. Claude recognizes a task is better handled by Gemini
2. Reads your `config.md` to select the right model for your tier
3. Builds and runs the Gemini CLI command
4. Reviews the output and returns the result

Key additions in `gemini-delegate`:
- **Tier-aware** — reads `config.md` to pick models and respect rate limits
- **Plan Mode safe** — every command includes the v0.34.0+ Plan Mode override
- **Auto-triggers** — description is tuned to fire on the right task signals
- **Concrete patterns** — opinionated commands for every use case, not templates

---

## Prerequisites

### 1. Install Gemini CLI

```bash
npm install -g @google/gemini-cli
```

### 2. Authenticate

```bash
# Option A: OAuth (recommended)
gemini
# Follow the interactive prompt on first run

# Option B: API key
export GEMINI_API_KEY=your_key_here
```

### 3. Verify

```bash
gemini --version
# gemini x.x.x
```

> **Tested on:** Gemini CLI v0.36.0

---

## Installation

```bash
# Clone this repo
git clone https://github.com/charlienew/gemini-delegate.git

# Copy the skill folder to your Claude skills directory
cp -r gemini-delegate ~/.claude/skills/gemini-delegate
```

That's it. Claude Code will discover the skill automatically on next launch.

---

## Configuration (Required)

Open `~/.claude/skills/gemini-delegate/config.md` and set your tier.
This is the only file you need to edit — the skill reads it before every task.

**Free tier (default):** No changes needed. The defaults work out of the box.

**Google AI Pro:** Change these three lines:
```
TIER=pro
DEFAULT_MODEL=auto-gemini-3
FAST_MODEL=gemini-3-flash-preview
```

Optionally pin to the latest preview model:
```
PREVIEW_MODEL=gemini-3.1-pro-preview
```

---

## Usage Examples

Once installed, Claude invokes this skill automatically. You can also ask directly:

```
"What's the latest stable version of Prisma?"
→ Gemini runs a Google Search and returns current release info

"Explain the architecture of this project"
→ Gemini runs codebase_investigator and maps the full structure

"What does this screenshot show?"
→ Gemini reads the image file and describes it

"Summarize this 3000-line config file"
→ Gemini reads the large file with its 1M context window

"Generate Jest tests for ./src/auth/middleware.ts"
→ Gemini reads the file and writes a test suite

"Security audit the ./src/api directory"
→ Gemini uses codebase_investigator to find vulnerabilities

"Watch this screen recording and describe the bug"
→ Gemini analyzes the video file

"Extract the key points from this PDF report"
→ Gemini reads the PDF with its read_file tool
```

---

## How It Works

```
1. Skill triggers (auto or explicit)
       ↓
2. Claude reads config.md → extracts TIER + model settings
       ↓
3. Claude selects model:
   - Complex/large → DEFAULT_MODEL
   - Quick/simple  → FAST_MODEL
   - Cutting-edge  → PREVIEW_MODEL (pro only)
       ↓
4. Claude builds command:
   gemini -m [model] [--yolo] "[task]. Execute immediately, do not show a plan." -o text 2>&1
       ↓
5. Claude runs command, reviews output, presents result
```

**Why "Execute immediately, do not show a plan."?**
Since Gemini CLI v0.34.0, Plan Mode is on by default. Without this phrase,
Gemini pauses and waits for interactive confirmation — which hangs Claude's
Bash call. This override is included in every command the skill generates.

---

## Example Commands

**Codebase architecture (pro):**
```bash
gemini -m auto-gemini-3 --yolo \
  "Use codebase_investigator to analyze this project. Report architecture, key files, and dependencies. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Large file summary (pro):**
```bash
gemini -m auto-gemini-3 \
  "Read and summarize @./src/legacy/main.ts. Include key exports and logic flows. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Image analysis:**
```bash
gemini -m auto-gemini-3 \
  "Describe the UI shown in @./mockup.png. Identify components and layout. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Real-time web search:**
```bash
gemini -m gemini-3-flash-preview \
  "Use Google Search: what are the known CVEs in lodash@4.17.20? List severity and fixes. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**PDF analysis:**
```bash
gemini -m auto-gemini-3 \
  "Read @./report.pdf and extract the key findings and recommendations. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Test generation (free tier):**
```bash
gemini -m gemini-2.5-flash --yolo \
  "Generate Jest unit tests for @./src/utils/parser.ts covering all exports and edge cases. Write to ./src/utils/parser.test.ts. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Troubleshooting

**Gemini hangs without responding**
Plan Mode is waiting for interactive input. The skill always appends
"Execute immediately, do not show a plan." — if this happens, check that the
prompt is being constructed correctly.

**Rate limit errors (free tier)**
The CLI auto-retries with exponential backoff. You'll see:
```
quota will reset after Xs
```
Wait for auto-retry. For sustained pressure, use `FAST_MODEL` by setting
`DEFAULT_MODEL=gemini-2.5-flash-lite` in `config.md` temporarily.

**"Model not found" error**
Your `DEFAULT_MODEL` doesn't match your actual plan. If you set `auto-gemini-3`
but are on the free tier, update `config.md` to `gemini-2.5-flash`.

**Authentication errors**
Re-authenticate: run `gemini` interactively to complete OAuth again.
Or check that `GEMINI_API_KEY` is set in your shell environment.

**Context too large**
Create a `.geminiignore` file in your project root to exclude large directories:
```
node_modules/
dist/
.next/
*.log
.env
```
Or scope the task: `--include-directories ./src` instead of the full project.

---

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill — Claude reads this for all instructions |
| `config.md` | User configuration — set your TIER here |
| `README.md` | This file — installation, usage, troubleshooting |

---

## Credits

This project is inspired by and based on the original
[gemini_cli_skill](https://github.com/forayconsulting/gemini_cli_skill) by
[@forayconsulting](https://github.com/forayconsulting). Key additions in this
fork: Gemini 3 model support, tier-aware config, Plan Mode safety, and an
expanded multi-file skill structure.

---

## Related

- [Gemini CLI GitHub](https://github.com/google-gemini/gemini-cli)
- [Google AI Studio](https://aistudio.google.com/) — get an API key or upgrade to Pro
- [Original skill](https://github.com/forayconsulting/gemini_cli_skill) — forayconsulting/gemini_cli_skill

---

## License

MIT
