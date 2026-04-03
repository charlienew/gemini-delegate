# Gemini Tools Reference

Gemini CLI provides tools beyond what Claude Code has natively. This reference
covers when to use each tool and how to invoke it effectively.

---

## Unique Gemini Tools

### `codebase_investigator`

Deep cross-file analysis tool. Unlike simple file reading, it builds a
structural understanding of the codebase — mapping components, tracing
dependencies, identifying entry points, and surfacing patterns across files.

**Use when:**
- Understanding an unfamiliar codebase for the first time
- Mapping how a feature or system is implemented across multiple files
- Generating architecture documentation
- Tracing a bug or data flow across file boundaries
- Security auditing a directory (combines well with `read_file`)

**Do NOT use when:**
- You only need to read a single specific file (use `@file` syntax instead)
- The task is quick and scoped — `codebase_investigator` is thorough but slow

**Invocation:**

Always combine with `--yolo` (the tool requires approval to run):
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use the codebase_investigator tool to [goal]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

Scope it when the full project is too large:
```bash
gemini -m $DEFAULT_MODEL --yolo --include-directories ./src/auth \
  "Use codebase_investigator to map the authentication system. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

### `google_web_search`

Real-time Google Search. Claude Code has no live internet access — this is
how you get current information.

**Use when:**
- Checking the latest version of a library or framework
- Looking up current CVEs or security advisories
- Fetching recent API changes or deprecation notices
- Finding current best practices that may have evolved since Claude's training
- Researching a library that Claude may not know well

**Do NOT use when:**
- The information is stable and Claude already knows it (adds latency for no gain)
- The task is entirely local (code review, file analysis, etc.)

**Invocation:**

Always explicitly instruct Gemini to use Google Search — otherwise it may
answer from its training data instead:
```bash
gemini -m $FAST_MODEL \
  "Use Google Search: [question]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

`$FAST_MODEL` is usually sufficient for web lookups — no need for Pro model capacity.

**Useful patterns:**
```
"Use Google Search: what is the latest stable version of [package] and what changed?"
"Use Google Search: are there known CVEs in [package]@[version]? List severity and fixes."
"Use Google Search: what does the [API endpoint] return according to current [Service] docs?"
"Use Google Search: what are current community recommendations for [topic] in [year]?"
```

---

### `read_file`

Reads files from disk, including binary formats Claude Code cannot handle natively.
This is what makes `@file` syntax work in prompts.

**Supported formats:**
- Text: any plain text, source code, config files, logs
- Images: PNG, JPG, GIF, WEBP, SVG, BMP
- Video: MP4, MOV, AVI, WEBM
- Documents: PDF

**Use when:**
- The file is too large for Claude to read efficiently (>500 lines)
- The file is an image, video, or PDF
- Multiple large files need to be read in one pass

**Invocation via `@` syntax:**

Reference files inline in the prompt:
```bash
gemini -m $DEFAULT_MODEL \
  "Read @./path/to/file.ts and [task]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

Multiple files in one prompt:
```bash
gemini -m $DEFAULT_MODEL \
  "Compare @./old.ts and @./new.ts. List differences. Execute immediately, do not show a plan." \
  -o text 2>&1
```

Directory-level inclusion (Gemini reads what it needs):
```bash
gemini -m $DEFAULT_MODEL --yolo --include-directories ./src \
  "[task involving files in ./src]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Notes:**
- Video files are token-intensive. Recommend `$DEFAULT_MODEL` (pro: `auto-gemini-3`)
  for videos longer than a few minutes
- PDFs and images use the same `@path` syntax as text files — no special flags needed

---

### `save_memory`

Persists key-value data across Gemini CLI sessions. Stored in `~/.gemini/` and
reloaded automatically in future sessions.

**Use when:**
- You want Gemini to remember project-specific context across multiple delegations
- Storing recurring configuration (team conventions, preferred libraries, style rules)
- Caching expensive analysis results (e.g., architecture summary) for reuse

**Invocation:**
```bash
gemini -m $FAST_MODEL --yolo \
  "Save to memory: key='[name]', value='[content]'. Execute immediately, do not show a plan." \
  -o text 2>&1
```

Retrieve stored memory:
```bash
gemini -m $FAST_MODEL \
  "Show all saved memory entries. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Standard Tools (Also Available in Claude Code)

These tools exist in Gemini but have Claude Code equivalents. Prefer Claude's
native tools for these unless you're already running a Gemini task.

| Gemini Tool | Claude Equivalent | Notes |
|-------------|------------------|-------|
| `list_directory` | `Glob` | Use Gemini's when already in a delegation |
| `read_file` (text) | `Read` | Use Gemini's for large files or binary formats |
| `search_file_content` | `Grep` | Ripgrep-powered, same capability |
| `glob` | `Glob` | Pattern-based file finding |
| `web_fetch` | `WebFetch` | URL fetching — both work the same |
| `write_todos` | Internal task tracking | Gemini-internal, not surfaced to Claude |

---

## Tool Combinations

Some tasks work best by combining tools in a single Gemini invocation:

**Security audit:**
`codebase_investigator` (map the surface) + `read_file` (read flagged files in depth)
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use codebase_investigator to identify security-sensitive files, then use read_file to audit each one for SQL injection, XSS, and auth issues. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Architecture + current docs:**
`codebase_investigator` (analyze local code) + `google_web_search` (check against latest API docs)
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Use codebase_investigator to map how we use the [library] API, then use Google Search to check if any of those usages are deprecated. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**README generation:**
`codebase_investigator` (understand the project) + `read_file` (read key files) + write output
```bash
gemini -m $DEFAULT_MODEL --yolo --include-directories . \
  "Use codebase_investigator and read_file to understand this project thoroughly, then write a comprehensive README.md. Execute immediately, do not show a plan." \
  -o text 2>&1
```
