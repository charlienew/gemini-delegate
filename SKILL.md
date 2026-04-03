---
name: gemini-delegate
description: >
  Delegate tasks to Google Gemini CLI when they exceed Claude's efficient working
  range or require Gemini's unique capabilities. AUTO-INVOKE for: files over 500
  lines or whole-codebase analysis; images (PNG, JPG, GIF, WEBP, SVG, BMP),
  video (MP4, MOV, AVI, WEBM), or PDF files; real-time web search or current
  information; architecture analysis, code generation, code review, security
  audit, test generation, or documentation tasks where a second AI perspective
  adds value; any task requiring more context than Claude can handle efficiently.
  Also invoke when user explicitly asks to use Gemini.
allowed-tools:
  - Bash
  - Read
  - Write
  - Grep
  - Glob
---

# Gemini Delegate Skill

Delegates tasks to Gemini CLI for its 1M context window, real-time web search,
and native multimodal file reading (images, video, PDF).

## See Also
- `tools.md` — when and how to use each Gemini tool (`codebase_investigator`, `google_web_search`, `read_file`, `save_memory`)
- `examples.md` — full command gallery for every use case
- `patterns.md` — advanced integration patterns (generate-review-fix, validation pipeline, anti-patterns, sessions)
- `reference.md` — complete CLI flag, model, and configuration reference
- `config.md` — model reference table and rate limit details

---

## Step 1: Load Configuration

```bash
cat ~/.claude/skills/gemini-delegate/config.md 2>/dev/null
```

Extract `TIER`, `DEFAULT_MODEL`, `FAST_MODEL`, `PREVIEW_MODEL`. If the file is
missing or a value is empty, use these defaults:

| Variable      | Free default            | Pro default              |
|---------------|-------------------------|--------------------------|
| DEFAULT_MODEL | `gemini-2.5-flash`      | `auto-gemini-3`          |
| FAST_MODEL    | `gemini-2.5-flash-lite` | `gemini-3-flash-preview` |
| PREVIEW_MODEL | *(not available)*       | `gemini-3.1-pro-preview` |

Pro tier also supports aliases: `auto-gemini-3` (recommended), `pro`, `flash`, `flash-lite`.

---

## Step 2: Plan Mode Rule (Required)

Since v0.34.0, Gemini's Plan Mode is ON by default. Every prompt **must** end with:

> "Execute immediately, do not show a plan."

Without this, the Bash call hangs waiting for interactive input.

Use `--yolo` / `-y` for tasks that **write files**. Omit `--yolo` for read-only tasks.

---

## Step 3: Standard Command Template

```bash
gemini -m [MODEL] [--yolo] \
  "[task]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Step 4: Should I Delegate to Gemini?

```
Will this require reading more than 3-4 files?        → YES, delegate
Will this generate more than ~100 lines of code?      → YES, have Gemini write directly
Does this need current web info Claude doesn't have?  → YES, delegate
Does this involve images, video, or PDF?              → YES, delegate
Can Claude handle it in 1-2 targeted reads?           → Handle directly
```

**Output format rules — critical for token savings:**
- **Generation tasks** (`--yolo`): Ask Gemini to write files directly to disk.
  Generated code never enters Claude's context — only a short confirmation does.
- **Analysis tasks** (no `--yolo`): Ask for structured output — file paths,
  markdown checklists, tables. Never prose. Claude acts on the list without
  re-reading anything.

**Task types:**
- File > 500 lines or large file summary → **Large File Reading**
- Images, video, PDF → **Multimodal**
- Current info / latest versions / CVEs → **Web Search**
- Understand a codebase → **Architecture Analysis**
- Generate CLAUDE.md, test suites, scaffolding → **Full Delegation: Write Directly**
- Refactor impact, dead code, migration, diff review → **Full Delegation: Structured Handoff**
- Write new code → **Code Generation**
- Review code or find bugs → **Code Review**
- Write tests → **Test Generation**
- Write JSDoc, README, API docs → **Documentation**

---

## Use Cases

### Large File Reading
```bash
gemini -m $DEFAULT_MODEL \
  "Read and summarize @./path/to/file.ts. Include purpose, key exports, and main logic flows. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Multimodal (Images, Video, PDF)
All use `@path/to/file` syntax. Gemini's `read_file` handles the format automatically.
```bash
gemini -m $DEFAULT_MODEL \
  "[Describe / analyze / extract from] @./file.[png|mp4|pdf]. Execute immediately, do not show a plan." \
  -o text 2>&1
```
→ See `examples.md` for image, video, and PDF patterns separately.

### Real-Time Web Search
Use `$FAST_MODEL`. Always tell Gemini to use Google Search explicitly.
```bash
gemini -m $FAST_MODEL \
  "Use Google Search: [question requiring current info]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Architecture Analysis
Uses `codebase_investigator`. Always requires `--yolo`.
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use codebase_investigator to analyze this project. Report architecture, key files, entry points, and dependencies. Execute immediately, do not show a plan." \
  -o text 2>&1
```
→ See `tools.md` for scoped analysis and tool combination patterns.

### Code Generation
Always use `--yolo`. Add `--include-directories ./src` for codebase-aware generation.
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Create [description] with [features]. Write all files. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Code Review
Omit `--yolo` for read-only review. Add it only if also applying fixes.
```bash
gemini -m $DEFAULT_MODEL \
  "Review @./path/to/file.ts. Report bugs, security issues, and improvements with line references. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Test Generation
Always use `--yolo`.
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Generate [Jest/pytest/Vitest] tests for @./path/to/module.ts covering all exports and edge cases. Write to ./path/to/module.test.ts. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Documentation
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Add JSDoc to every undocumented function in @./path/to/file.ts. Write back to the same file. Execute immediately, do not show a plan." \
  -o text 2>&1
```
→ See `examples.md` for README and API doc patterns.

---

## After Delegation: Validate Output

For generated code, always verify before presenting to the user:
- **Syntax:** `node --check file.js` / `tsc --noEmit file.ts` / `python -m py_compile file.py`
- **Security:** check for XSS, injection, exposed secrets, missing input validation
- **Tests:** run existing test suite (`npm test`, `pytest`, etc.)

→ See `patterns.md` Pattern 6 for the full validation pipeline with automated sequencing.

---

## Output

- `-o text 2>&1` — standard output (use for most tasks)
- `-o json 2>&1` — structured output; parse `.response` for content, `.stats` for token usage

---

## Common Errors

| Symptom | Fix |
|---------|-----|
| Bash call hangs | Prompt missing "Execute immediately, do not show a plan." |
| `quota will reset after Xs` | CLI auto-retries; switch to `$FAST_MODEL` for sustained pressure |
| `Model not found` | Model in `config.md` not available on your plan |
| Auth error | Run `gemini` interactively to re-authenticate |
| Context too large | Add `.geminiignore` or scope with `--include-directories ./src` |
