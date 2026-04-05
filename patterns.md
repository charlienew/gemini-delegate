# Gemini Delegate — Integration Patterns

Advanced patterns for orchestrating Gemini CLI effectively from Claude Code.
All commands use `$DEFAULT_MODEL`/`$FAST_MODEL` from `config.md` and include
`"Execute immediately, do not show a plan."` to bypass Plan Mode.

---

## Pattern 1: Generate-Review-Fix Cycle

The most reliable pattern for quality code. Use `--yolo` only on the generate
and fix steps; omit it on the review step to prevent unintended writes.

```bash
# Step 1: Generate (--yolo required — writes files)
gemini -m $DEFAULT_MODEL --yolo \
  "Create [description]. Write to [path]. Execute immediately, do not show a plan." \
  -o text 2>&1

# Step 2: Review (no --yolo — read only)
gemini -m $DEFAULT_MODEL \
  "Review [path] for bugs, security issues, and correctness. Reference specific lines. Execute immediately, do not show a plan." \
  -o text 2>&1

# Step 3: Fix (--yolo required — writes files)
gemini -m $DEFAULT_MODEL --yolo \
  "Fix the issues found in [path]: [list from review]. Apply fixes now. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Why it works:** Generation and review use different "mindsets." Self-review
reliably catches XSS risks, missing validation, and logic errors that slipped
through generation.

---

## Pattern 2: Background Execution

For long-running tasks, run in background and continue other work.

```bash
# Single background task
gemini -m $DEFAULT_MODEL --yolo \
  "[long task]. Execute immediately, do not show a plan." \
  -o text 2>&1 &
echo "Gemini running (PID: $!)"
```

**Parallel independent tasks:**
```bash
gemini -m $DEFAULT_MODEL --yolo "Create the frontend. Execute immediately, do not show a plan." -o text 2>&1 &
gemini -m $DEFAULT_MODEL --yolo "Create the backend. Execute immediately, do not show a plan."  -o text 2>&1 &
gemini -m $DEFAULT_MODEL --yolo "Generate the tests. Execute immediately, do not show a plan."  -o text 2>&1 &
wait  # Wait for all to complete
```

Use parallel execution only for truly independent tasks — tasks with shared
file writes can conflict.

---

## Pattern 3: Model Selection Strategy

```
Is the task complex (multi-file, deep analysis, 1M+ context)?
├── Yes → Is it codebase_investigator on a large project (> 30 source files)?
│   ├── Yes → Use FAST_MODEL + --include-directories (avoids RPM limits)
│   └── No → Use DEFAULT_MODEL
└── No → Is speed the priority (quick lookup, simple review)?
    ├── Yes → Use FAST_MODEL (gemini-3-flash-preview on pro, gemini-2.5-flash-lite on free)
    └── No → Is cutting-edge reasoning needed? (pro only)
        ├── Yes → Use PREVIEW_MODEL (gemini-3.1-pro-preview)
        └── No → Use DEFAULT_MODEL
```

**Why `codebase_investigator` is different:** it's tool-call-intensive, not
reasoning-intensive. The bottleneck is how fast the model can invoke file-reading
tools — FAST_MODEL has better per-minute throughput, preventing RPM limit hits
during the rapid scan. Model reasoning quality doesn't matter here.

**Examples:**
```bash
# Large codebase investigation → FAST_MODEL (RPM-safe)
gemini -m $FAST_MODEL --yolo --include-directories ./src "Use codebase_investigator... Execute immediately, do not show a plan." -o text 2>&1

# Complex reasoning task → DEFAULT_MODEL
gemini -m $DEFAULT_MODEL --yolo "Generate auth module with JWT... Execute immediately, do not show a plan." -o text 2>&1

# Quick: version lookup → FAST_MODEL
gemini -m $FAST_MODEL "Use Google Search: latest version of [lib]. Execute immediately, do not show a plan." -o text 2>&1

# Cutting-edge (pro only): experimental reasoning
gemini -m $PREVIEW_MODEL "Analyze [complex problem]. Execute immediately, do not show a plan." -o text 2>&1
```

---

## Pattern 4: Rate Limit Handling

**Default (let CLI handle it):** The CLI auto-retries with exponential backoff.
You'll see `quota will reset after Xs` — just wait.

**Switch to fast model for sustained pressure:**
```bash
# If hitting limits on DEFAULT_MODEL tasks
gemini -m $FAST_MODEL --yolo "[task]. Execute immediately, do not show a plan." -o text 2>&1
```

**Batch related operations into one call:**
```bash
# Instead of 3 separate calls:
gemini -m $DEFAULT_MODEL --yolo \
  "Create files A, B, and C: [specs for each]. Create all now. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Sequential with delay (for automated scripts):**
```bash
gemini -m $DEFAULT_MODEL --yolo "[task 1]. Execute immediately, do not show a plan." -o text 2>&1
sleep 2
gemini -m $DEFAULT_MODEL --yolo "[task 2]. Execute immediately, do not show a plan." -o text 2>&1
```

---

## Pattern 5: Context Enrichment

**Explicit file references:**
```bash
gemini -m $DEFAULT_MODEL \
  "Based on @./package.json and @./src/index.ts, suggest architectural improvements. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Directory-level inclusion:**
```bash
gemini -m $DEFAULT_MODEL --yolo --include-directories ./src \
  "[task that needs full src context]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Persistent project context via GEMINI.md:**

Create `.gemini/GEMINI.md` in the project root — Gemini loads it automatically
in every session for that project:
```markdown
# .gemini/GEMINI.md

## Project Overview
[Project purpose and stack]

## Coding Standards
- [Standards to follow]

## When Making Changes
- [Guidelines for modifications]
```

**Explicit inline context:**
```bash
gemini -m $DEFAULT_MODEL --yolo \
  "Project uses React 18 + TypeScript. State: Zustand. Styling: Tailwind.
  Create a [component]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

---

## Pattern 6: Validation Pipeline

Always validate generated code before presenting to the user.

```bash
# Step 1: Generate
gemini -m $DEFAULT_MODEL --yolo \
  "Create [module]. Write to ./module.ts. Execute immediately, do not show a plan." \
  -o text 2>&1

# Step 2: Syntax check
node --check ./module.js        # JavaScript
tsc --noEmit ./module.ts        # TypeScript
python -m py_compile module.py  # Python

# Step 3: Security scan (manual or automated)
# - Check for innerHTML with user input (XSS)
# - Look for eval() / Function() calls
# - Verify input validation on external data
# - Check for hardcoded secrets or API keys

# Step 4: Functional test
npm test          # Run existing test suite
pytest            # Python equivalent

# Step 5: Style check
eslint ./module.ts
prettier --check ./module.ts
```

**Automated sequence:**
```bash
gemini -m $DEFAULT_MODEL --yolo "Create [module] in ./module.ts. Execute immediately, do not show a plan." -o text 2>&1 \
  && tsc --noEmit ./module.ts \
  && eslint ./module.ts \
  && npm test
```

---

## Pattern 7: Incremental Refinement

Build complex outputs in stages. Each stage validates before the next begins.

```bash
# Stage 1: Core structure
gemini -m $DEFAULT_MODEL --yolo \
  "Create a basic Express server with routes for /api/users and /api/auth. Write to ./server.ts. Execute immediately, do not show a plan." \
  -o text 2>&1

# Stage 2: Add authentication
gemini -m $DEFAULT_MODEL --yolo \
  "Add JWT authentication middleware to @./server.ts. Preserve existing routes. Execute immediately, do not show a plan." \
  -o text 2>&1

# Stage 3: Add another feature
gemini -m $DEFAULT_MODEL --yolo \
  "Add rate limiting to @./server.ts. Execute immediately, do not show a plan." \
  -o text 2>&1

# Stage 4: Final review
gemini -m $DEFAULT_MODEL \
  "Review @./server.ts for bugs, security issues, and improvements. Execute immediately, do not show a plan." \
  -o text 2>&1
```

Benefits: easier to debug, clearer audit trail, each stage validates continuity
with what came before.

---

## Pattern 8: Cross-Validation (Claude ↔ Gemini)

Use both AIs for highest confidence output.

**Claude generates, Gemini reviews:**
```bash
# 1. Claude writes the code (using normal Claude Code tools)
# 2. Gemini reviews from a different perspective
gemini -m $DEFAULT_MODEL \
  "Review @./path/to/claude-generated.ts for bugs, security issues, and logic errors.
  Be critical. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Gemini generates, Claude reviews:**
```bash
# 1. Gemini generates
gemini -m $DEFAULT_MODEL --yolo \
  "Create [code]. Write to [path]. Execute immediately, do not show a plan." \
  -o text 2>&1
# 2. Claude reviews in the normal conversation
# "Review the file Gemini just created at [path]..."
```

**Different strengths:**
- Claude: strong reasoning, following complex multi-step instructions, deep context
- Gemini: strong codebase investigation, current web knowledge, 1M context window

---

## Pattern 9: Session Continuity

For multi-turn workflows, use Gemini sessions to preserve context.

```bash
# Initial analysis
gemini -m $DEFAULT_MODEL --yolo \
  "Analyze the architecture of this project. Execute immediately, do not show a plan." \
  -o text 2>&1
# Session is saved automatically

# List saved sessions
gemini --list-sessions

# Continue in the same session (by index)
echo "Now focus on the authentication flow in detail." | gemini -r 1 -o text 2>&1

# Continue with latest session
echo "What security issues did you find?" | gemini -r latest -o text 2>&1
```

**Use cases for sessions:**
- Iterative architecture analysis (zoom out → zoom in)
- Multi-step code generation building on earlier context
- Debugging sessions where each step reveals the next

---

## Pattern 10: JSON Output Pipeline

Use JSON output for automation, monitoring, and structured result handling.

```bash
OUTPUT=$(gemini -m $DEFAULT_MODEL --yolo \
  "[task]. Execute immediately, do not show a plan." \
  -o json 2>&1)

# Extract response content
echo "$OUTPUT" | python3 -c "import json,sys; print(json.load(sys.stdin)['response'])"

# Check token usage
echo "$OUTPUT" | python3 -c "
import json, sys
data = json.load(sys.stdin)
stats = data.get('stats', {})
models = stats.get('models', {})
for model, info in models.items():
    tokens = info.get('tokens', {})
    print(f'{model}: {tokens.get(\"total\", 0)} tokens')
"

# Check which tools were called
echo "$OUTPUT" | python3 -c "
import json, sys
data = json.load(sys.stdin)
tools = data.get('stats', {}).get('tools', {}).get('byName', {})
for tool, info in tools.items():
    print(f'{tool}: {info[\"count\"]} calls, {info[\"success\"]} success')
"
```

---

## Anti-Patterns

### Plan Mode Hang
**Problem:** Bash call hangs indefinitely waiting for interactive input.
**Cause:** Prompt doesn't end with "Execute immediately, do not show a plan."
```bash
# ❌ Hangs
gemini -m $DEFAULT_MODEL --yolo "Create a user service." -o text 2>&1

# ✅ Works
gemini -m $DEFAULT_MODEL --yolo "Create a user service. Execute immediately, do not show a plan." -o text 2>&1
```

### Blind Trust
**Problem:** Presenting Gemini's output to the user without review.
**Risk:** Security vulnerabilities (XSS, injection), incorrect logic, outdated patterns.
**Fix:** Always review and validate before presenting. Use the Validation Pipeline (Pattern 6).

### Over-Specifying in One Prompt
**Problem:** Extremely long prompts with complex multi-step requirements confuse the model.
**Fix:** Use Incremental Refinement (Pattern 7) — break into stages.

### Ignoring Rate Limits
**Problem:** Hammering the API causes cascading retries and slow responses.
**Fix:** Batch related operations (Pattern 4). Use FAST_MODEL for lower-priority tasks.

### Context Overflow
**Problem:** Running codebase analysis on a large project without scoping — Gemini hits context limits.
**Fix:**
1. Add `.geminiignore` to exclude `node_modules/`, `dist/`, `*.log`, `.env`
2. Use `--include-directories ./src` instead of the whole project
3. Split analysis into focused scoped calls

---

## Pattern 11: Full Delegation

The highest token-saving pattern. Gemini does all the reading and writing —
Claude only receives a short confirmation or structured summary.

**When to use:**
- Task requires reading more than 3-4 files
- Task generates more than ~100 lines of code
- Task involves scanning many files for patterns (data, logs, dead code)

**Two variants:**

**Write directly** — for generation tasks. Use `--yolo`. Gemini writes files to
disk; Claude never sees the code, only a confirmation.
```bash
# Gemini generates AND writes — Claude context cost: ~50 tokens
gemini -m $DEFAULT_MODEL --yolo \
  "Generate [code]. Write to [path]. Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Structured handoff** — for analysis tasks. No `--yolo`. Gemini returns a
targeted list (file paths, checklists, tables) Claude can act on without
re-reading anything.
```bash
# Gemini explores entire codebase — Claude context cost: size of the list only
gemini -m $DEFAULT_MODEL \
  "Use codebase_investigator to find [X]. Return a markdown checklist:
  file path + one sentence on what to change. Nothing else.
  Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Rules:**
- Never ask for prose descriptions — always request structured lists
- After Gemini returns, Claude does NOT re-read files to verify
- If re-reading feels necessary, the prompt was not specific enough
- For recurring analysis, use `save_memory` to avoid re-running next session

→ See `examples.md` "Full Delegation Workflows" for 8 ready-to-use commands.

---

## Pattern 12: Parallel Gemini via Claude Subagents

Use Claude's **Agent tool** (not bash `&`) when parallel Gemini tasks need
per-task Claude work: reading config, constructing commands, validating output,
or post-processing before returning results.

**Bash `&` (Pattern 2) vs Agent subagents:**

| | Bash `&` | Agent subagents |
|---|---|---|
| Best for | Simple independent commands | Tasks needing per-task logic or validation |
| Overhead | Zero | Subagent spawn cost |
| Output handling | `wait` then read file/stdout | Each subagent returns structured result |
| Context isolation | Shared bash session | Each subagent has its own context |
| Error handling | Manual PID tracking | Managed by Claude infrastructure |

**When to use Agent subagents:**
- 3+ modules to analyze or generate independently
- Each task needs config reading, output parsing, or validation
- You want results returned as structured data, not raw Gemini stdout

**How to dispatch:**

Tell Claude to spawn subagents in a single message (parallel dispatch). Each
subagent should:
1. Read `config.md` to get model variables
2. Run its specific Gemini command
3. Return a short structured result (not raw Gemini output)

**Example prompt to Claude:**
```
Analyze the auth, payments, and notification modules in parallel.
Use the Agent tool to dispatch three subagents simultaneously — one per module.
Each subagent should run codebase_investigator scoped to its directory and
return: module name, key files, and a 3-bullet architecture summary.
```

**What each subagent runs:**
```bash
# Subagent 1 (auth)
gemini -m $FAST_MODEL --yolo --include-directories ./src/auth \
  "Use codebase_investigator to analyze the auth module.
   Return: key files, entry point, dependencies, 3-bullet summary.
   Execute immediately, do not show a plan." \
  -o text 2>&1

# Subagent 2 (payments) — runs simultaneously
gemini -m $FAST_MODEL --yolo --include-directories ./src/payments \
  "Use codebase_investigator to analyze the payments module.
   Return: key files, entry point, dependencies, 3-bullet summary.
   Execute immediately, do not show a plan." \
  -o text 2>&1
```

**Rate limit note:** parallel Gemini subagents multiply RPM pressure. Use
`$FAST_MODEL` for subagent tasks (better RPM) and limit to 3 concurrent
subagents. If you need more, batch in waves of 3.
