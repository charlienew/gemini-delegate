# Gemini Delegate — Command Examples

Full example gallery for each use case. Commands use `$DEFAULT_MODEL` and
`$FAST_MODEL` placeholders — substitute values from `config.md` before running.

All commands include `--yolo` only when writing files. All prompts end with
`"Execute immediately, do not show a plan."` to bypass Gemini's Plan Mode.

---

## Large File Reading

**Summarize a large file:**
```bash
gemini -m $DEFAULT_MODEL \
  "Read and summarize @./path/to/large-file.ts. Include: purpose, key exports, main logic flows, and external dependencies. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Find patterns in a large file:**
```bash
gemini -m $DEFAULT_MODEL \
  "In @./path/to/file.ts, find and list all error handling patterns including any unhandled cases. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Compare two large files:**
```bash
gemini -m $DEFAULT_MODEL \
  "Compare @./old-implementation.ts and @./new-implementation.ts. List behavioral differences and any regressions introduced. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Explain a complex configuration file:**
```bash
gemini -m $DEFAULT_MODEL \
  "Read @./webpack.config.js and explain what each section does. Highlight anything unusual or potentially problematic. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Image Understanding

**Describe an image:**
```bash
gemini -m $DEFAULT_MODEL \
  "Describe in detail what is shown in @./screenshot.png. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**UI/UX analysis from screenshot:**
```bash
gemini -m $DEFAULT_MODEL \
  "Analyze the UI shown in @./ui-mockup.png. Identify: layout structure, components present, navigation patterns, and accessibility concerns. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Interpret an architecture or system diagram:**
```bash
gemini -m $DEFAULT_MODEL \
  "Interpret the architecture diagram in @./system-diagram.png. Describe each component, their responsibilities, and how they interact. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Extract text from an image (OCR):**
```bash
gemini -m $DEFAULT_MODEL \
  "Extract all text visible in @./document-scan.jpg. Preserve the original formatting and structure. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Compare two design mockups:**
```bash
gemini -m $DEFAULT_MODEL \
  "Compare @./design-v1.png and @./design-v2.png. List all visual differences and which version better follows standard UX patterns. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Video Understanding

**Summarize a demo or walkthrough:**
```bash
gemini -m $DEFAULT_MODEL \
  "Watch @./demo.mp4 and summarize what is demonstrated. Include timestamps for key moments. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Extract steps from a tutorial video:**
```bash
gemini -m $DEFAULT_MODEL \
  "Watch @./tutorial.mp4 and extract all steps demonstrated as a numbered list. Be specific enough that someone could reproduce them without watching. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Describe a bug from a screen recording:**
```bash
gemini -m $DEFAULT_MODEL \
  "Watch @./bug-recording.mp4 and describe the bug shown. Include: what user actions trigger it, what the expected behavior should be, and what the actual behavior is. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## PDF Documents

**Extract key points:**
```bash
gemini -m $DEFAULT_MODEL \
  "Read @./report.pdf and extract the key findings, decisions, and action items. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Summarize a technical spec:**
```bash
gemini -m $DEFAULT_MODEL \
  "Read @./spec.pdf and summarize: the problem being solved, the proposed solution, constraints, and open questions. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Extract structured data from a PDF:**
```bash
gemini -m $DEFAULT_MODEL \
  "Read @./invoice.pdf and extract all line items, amounts, dates, and totals as a markdown table. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Real-Time Web Search

**Latest version and changelog:**
```bash
gemini -m $FAST_MODEL \
  "Use Google Search: what is the latest stable version of [library] and what changed in the last two releases? Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Security vulnerabilities:**
```bash
gemini -m $FAST_MODEL \
  "Use Google Search: are there known CVEs or security vulnerabilities in [package]@[version]? List each with severity and recommended fix. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Current best practices:**
```bash
gemini -m $FAST_MODEL \
  "Use Google Search: what are current community best practices for [topic] in [year]? Summarize the top recommendations. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**API or documentation lookup:**
```bash
gemini -m $FAST_MODEL \
  "Use Google Search: find the current documentation for [endpoint/method] from [service]. Summarize parameters, response format, and any recent changes. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Dependency compatibility:**
```bash
gemini -m $FAST_MODEL \
  "Use Google Search: is [package-a]@[version] compatible with [package-b]@[version]? Are there known conflicts? Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Architecture Analysis

**Full project analysis:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use codebase_investigator to fully analyze this project. Report: overall architecture, key file purposes, entry points, component relationships, dependency chains, data flow, and any potential issues. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Scoped to a directory:**
```bash
gemini -m $DEFAULT_MODEL --yolo --include-directories ./src/auth \
  "Use codebase_investigator to map the authentication system. Report: components, session handling, token flow, and security patterns. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Dependency analysis:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use codebase_investigator to map all external dependencies in this project and how they are used. Identify any that appear unused or risky. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Onboarding summary:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use codebase_investigator to generate an onboarding guide for a new developer joining this project. Cover: project purpose, key concepts, how to run it locally, and the most important files to understand first. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Code Generation

**Single file:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Create [description] with [features]. Follow [language/framework] conventions. Write the complete file to [path]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Multi-file feature:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Create a [feature] using [stack]. Files needed: [list or 'determine what's needed']. Make it immediately runnable. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Codebase-aware (reads existing patterns first):**
```bash
gemini -m $DEFAULT_MODEL --yolo --include-directories ./src \
  "Add a [feature] to this codebase. Read the relevant existing files first to match patterns and conventions, then implement. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Refactor:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Refactor @./path/to/file.ts to [goal — e.g., use async/await, split into smaller functions, remove duplication]. Preserve all existing behavior. Write back to the same file. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Code Review and Security Audit

**Comprehensive file review (read-only — no `--yolo`):**
```bash
gemini -m $DEFAULT_MODEL \
  "Review @./path/to/file.ts and report: 1) what it does, 2) bugs found, 3) security issues (XSS, injection, auth flaws, secrets), 4) performance concerns, 5) improvement suggestions. Reference specific line numbers. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Security audit of a directory:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use codebase_investigator and read_file to security-audit ./src/api. Check for: SQL injection, XSS, insecure JWT handling, plaintext secrets, missing input validation, IDOR vulnerabilities. Report each finding with file and approximate line. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Review and auto-fix:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Review @./path/to/file.ts for bugs and security issues. Fix each issue found in place. Summarize what was changed. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Review a git diff:**
```bash
git diff HEAD > /tmp/changes.diff && gemini -m $DEFAULT_MODEL \
  "Review the git diff at @/tmp/changes.diff for correctness, logic errors, missing edge cases, and security issues. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Test Generation

**Unit tests:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Generate [Jest/pytest/Vitest] unit tests for @./path/to/module.ts. Cover: all exported functions, edge cases (null/undefined/empty), error paths, and boundary conditions. Write to ./path/to/module.test.ts. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Integration tests:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Generate integration tests for the API routes in @./src/routes/. Use [framework]. Test happy paths and error scenarios for each endpoint. Write to ./tests/integration/. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Full feature test suite (uses codebase context):**
```bash
gemini -m $DEFAULT_MODEL --yolo --include-directories ./src \
  "Read the [feature name] feature in this codebase, then generate a comprehensive test suite covering unit and integration tests. Write to ./tests/. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Tests for a specific bug fix:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Read @./path/to/module.ts. Write regression tests that would have caught the bug described as: [bug description]. Write to ./path/to/module.test.ts. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Documentation

**JSDoc / TSDoc:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Add JSDoc comments to every undocumented function and class in @./path/to/file.ts. Preserve all existing comments. Write back to the same file. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**README generation:**
```bash
gemini -m $DEFAULT_MODEL --yolo --include-directories . \
  "Use codebase_investigator and read_file to understand this project, then write a comprehensive README.md. Include: description, prerequisites, installation, configuration, usage examples, and contributing guide. Write to ./README.md. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**API documentation:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Document all REST endpoints in @./src/routes/. For each: method, path, request parameters, request body schema, response schema, example request and response. Output as Markdown. Write to ./docs/api.md. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Inline code explanation:**
```bash
gemini -m $DEFAULT_MODEL \
  "Read @./path/to/complex-file.ts and add inline comments explaining the non-obvious parts — algorithms, business logic, and side effects. Do not comment self-evident lines. Write back to the same file. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Advanced Patterns

**Generate, then self-review:**
```bash
# Step 1: Generate
gemini -m $DEFAULT_MODEL --yolo \
  "Create [description]. Write to [path]. Execute immediately, do not show a plan." \
  -o text 2>&1

# Step 2: Review what was generated
gemini -m $DEFAULT_MODEL \
  "Review the file just written at [path] for bugs, security issues, and correctness. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Background execution for long tasks:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "[long-running task]. Execute immediately, do not show a plan." \
  -o text 2>&1 &
echo "Gemini task running in background (PID: $!)"
```

**JSON output for structured results:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "[task]. Return results as JSON with fields: [list fields]. Execute immediately, do not show a plan." \
  -o json 2>&1
# Parse .response for content, .stats for token usage
```

**Session continuity for multi-turn tasks:**
```bash
# Start a session and save context
gemini -m $DEFAULT_MODEL --yolo \
  "Analyze [part 1 of task]. Execute immediately, do not show a plan." \
  -o text 2>&1

# Continue in the same session
echo "Now do [part 2]" | gemini -r latest -o text 2>&1
```

**Validation pipeline (generate → check → test):**
```bash
# Step 1: Generate
gemini -m $DEFAULT_MODEL --yolo \
  "Create [module]. Write to ./module.ts. Execute immediately, do not show a plan." \
  -o text 2>&1

# Step 2: Syntax check
tsc --noEmit ./module.ts

# Step 3: Run tests
npm test

# Automated (stops on first failure):
gemini -m $DEFAULT_MODEL --yolo "Create [module] in ./module.ts. Execute immediately, do not show a plan." -o text 2>&1 \
  && tsc --noEmit ./module.ts \
  && npm test
```

**Anti-pattern: forgetting Plan Mode override:**
```bash
# ❌ Wrong — Bash call hangs waiting for interactive input
gemini -m $DEFAULT_MODEL --yolo \
  "Create a user service." \
  -o text 2>&1

# ✅ Correct — includes Plan Mode override
gemini -m $DEFAULT_MODEL --yolo \
  "Create a user service. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Anti-pattern: using --yolo on a review task:**
```bash
# ❌ Wrong — --yolo gives Gemini permission to write files during "review"
gemini -m $DEFAULT_MODEL --yolo \
  "Review @./auth.ts for security issues. Execute immediately, do not show a plan." \
  -o text 2>&1

# ✅ Correct — omit --yolo for read-only tasks
gemini -m $DEFAULT_MODEL \
  "Review @./auth.ts for security issues. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Full Delegation Workflows

These patterns are designed to save Claude Code tokens. Gemini does all the
reading/writing — Claude only sees a short confirmation or structured summary.

### CLAUDE.md Generation

Gemini analyzes the entire codebase and writes the file directly. Claude reads nothing.

```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use codebase_investigator to fully analyze this project. Write a CLAUDE.md
  covering: project purpose, architecture overview, key files and their roles,
  development commands (build, test, lint), and coding conventions. Write
  directly to ./CLAUDE.md. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Refactor Impact Map

Returns a targeted checklist. Claude reads only the listed files.

```bash
gemini -m $DEFAULT_MODEL \
  "Use codebase_investigator to find every file affected by [describe change].
  Return a markdown checklist: each item is a file path + one sentence on what
  needs to change. Nothing else. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Large Diff / PR Review

Pipes the diff directly to Gemini. The full diff never enters Claude's context.

```bash
git diff main...HEAD | gemini -m $DEFAULT_MODEL \
  "Review this diff. Return a markdown report: (1) change summary, (2) bugs
  found with file:line references, (3) security concerns, (4) suggestions.
  Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Test Suite Generation

Gemini reads source files and writes test files directly. Claude sees only a summary.

```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Read all source files in @./src/[module]. Generate comprehensive tests
  covering all exports and edge cases. Write test files alongside source files.
  Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Dependency Migration Analysis

Returns a file-by-file change checklist for upgrading a library or API.

```bash
gemini -m $DEFAULT_MODEL \
  "Use codebase_investigator to find every usage of [old API/library]. Return a
  markdown checklist: file path + exactly what needs to change for [new version].
  Order by dependency. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Data / Log File Exploration

Scans many files and returns a structured summary Claude can act on directly.

```bash
gemini -m $DEFAULT_MODEL \
  "Read all files matching @./data/*.json. Identify the schema, value ranges,
  and any anomalies. Return a structured summary: field names, types, and sample
  values. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Dead Code Detection

Returns a list. Claude removes the listed symbols without reading the codebase.

```bash
gemini -m $DEFAULT_MODEL \
  "Use codebase_investigator to find all unused exports, functions, and
  variables across this project. Return a list: file path + symbol name + why
  it appears unused. Execute immediately, do not show a plan." \
  -o text 2>&1
```

### Codebase Map Saved to Memory

Front-load once. Gemini stores the map in `~/.gemini/` for reuse across sessions.

```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use codebase_investigator to map this project. Then use save_memory with key
  'codebase_map' = a structured list of every meaningful file: path, one-line
  purpose, and key exports. Execute immediately, do not show a plan." \
  -o text 2>&1
```
