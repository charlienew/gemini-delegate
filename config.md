# Gemini Delegate — User Configuration

Edit this file once after installing the skill. Claude reads it before every
delegated task to select the right model and respect your plan's rate limits.

---

## Tier

Set your Google AI tier. Options: `free` or `pro`

- `free` — Uses gemini-2.5-flash. 60 req/min, 1,000 req/day. No cost.
- `pro`  — Uses auto-gemini-3 (auto-selects Pro/Flash). Requires Google AI Pro subscription.

```
TIER=free
```

---

## Model Overrides (optional)

These are set automatically based on `TIER`. Only change if you want to
override the defaults for a specific model.

**Free tier models:** `gemini-2.5-flash`, `gemini-2.5-flash-lite`
**Pro tier aliases:** `auto-gemini-3` (recommended), `pro`, `flash`, `flash-lite`
**Pro tier concrete:** `gemini-3-pro-preview`, `gemini-3-flash-preview`, `gemini-3.1-pro-preview`

```
# Primary model — used for complex tasks (large files, architecture, generation)
# Free default:  gemini-2.5-flash
# Pro default:   auto-gemini-3
DEFAULT_MODEL=gemini-2.5-flash

# Fast model — used for quick tasks (lookups, simple reviews, short questions)
# Free default:  gemini-2.5-flash-lite
# Pro default:   gemini-3-flash-preview
FAST_MODEL=gemini-2.5-flash-lite

# Preview model — pro only, for cutting-edge reasoning (leave blank if unused)
# Pro option:  gemini-3.1-pro-preview
PREVIEW_MODEL=
```

---

## Model Reference

### Aliases (pro tier — recommended)

| Alias           | Resolves To                          | Best For                              |
|-----------------|--------------------------------------|---------------------------------------|
| `auto-gemini-3` | Pro or Flash based on task           | General use — avoids capacity issues  |
| `pro`           | Most capable available model         | Complex reasoning, large context      |
| `flash`         | Fast balanced model                  | Most tasks                            |
| `flash-lite`    | Fastest, most efficient model        | Simple lookups, quick reviews         |

### Concrete Model Identifiers

| Model                    | Tier | Context   | Best For                              |
|--------------------------|------|-----------|---------------------------------------|
| `gemini-2.5-flash`       | free | large     | All tasks on free tier                |
| `gemini-2.5-flash-lite`  | free | medium    | Fastest/simplest tasks on free tier   |
| `gemini-3-pro-preview`   | pro  | 1M tokens | Large context, complex analysis       |
| `gemini-3-flash-preview` | pro  | 1M tokens | Speed-critical tasks on pro tier      |
| `gemini-3.1-pro-preview` | pro  | 1M tokens | Latest capabilities (preview)         |

---

## Rate Limits Reference

| Tier | Requests/Min | Requests/Day | Notes                           |
|------|--------------|--------------|---------------------------------|
| free | 60           | 1,000        | CLI auto-retries with backoff   |
| pro  | higher       | higher       | See Google AI Pro plan details  |

---

## Pro Tier Quick Setup

If you are on Google AI Pro, change these three lines:

```
TIER=pro
DEFAULT_MODEL=auto-gemini-3
FAST_MODEL=gemini-3-flash-preview
```

And optionally enable the preview model:

```
PREVIEW_MODEL=gemini-3.1-pro-preview
```

---

## Notes

- This file is read-only from the skill's perspective. Claude will not modify it.
- Gemini CLI must be installed and authenticated before this skill works.
  Install: `npm install -g @google/gemini-cli`
  Auth: run `gemini` once interactively, or set `GEMINI_API_KEY` environment variable.
