---
name: make-article
description: Launch the article writing pipeline with automatic input mode detection.
argument-hint: <repo-url | topic> [--title "Title"] [--urls "url1,url2"] [--word-count N]
---

# /tech-insider:make-article

**Usage**:
```
/tech-insider:make-article <input> --title "Article Title" [options]
```

**Input Modes** (auto-detected from first argument):
- **Repo Mode**: `https://github.com/user/repo` — analyze source code
- **URL Mode**: `https://example.com/article` — fetch reference URLs
- **Idea Mode**: `Rust async 最佳实践` — web search for research

**Options**:
- `--title "..."` — article title (required)
- `--topic "..."` — specific topic focus (optional, overrides first arg for Idea mode)
- `--urls "url1,url2,..."` — comma-separated reference URLs (optional)
- `--word-count N` — target word count 3000-10000 (default 5000)
- `--audience "..."` — target readers (optional)
- `--focus "..."` — key focus areas, comma-separated (optional)
- `--article-dir "..."` — output directory (default `<topic>-article/`)

---

## Parameter Parsing

```python
# Pseudo-code for parameter extraction
input = first non-`--` argument
title = extract --title (required, error if missing)
topic = extract --topic (optional)
urls = extract --urls (optional, comma-separated)
word_count = extract --word-count (default 5000)
audience = extract --audience (optional)
focus = extract --focus (optional)
article_dir = extract --article-dir (default `<topic>-article/`)
```

### Mode Detection

```
if input starts with "https://github.com/":
    mode = "repo"
elif input starts with "https://":
    mode = "url"
    add input to --urls list
else:
    mode = "idea"
    topic = topic or input  # use first arg as topic keyword
```

---

## Validation

1. **Title required** — if missing, error: "`--title` is required"
2. **URL mode** — if `--urls` empty in URL mode, error: "At least one URL required"
3. **Idea mode** — if topic is empty, error: "Topic keyword required"
4. **Word count** — if not 3000-10000, error: "Word count must be 3000-10000"

---

## Delegation

After parsing and validation:
1. Create output directory: `mkdir -p <article-dir>/.work`
2. Load skill: `skills/article-pipeline/SKILL.md`
3. Pass all parameters to pipeline orchestrator

**The command does NOT**:
- Clone repositories
- Fetch URLs
- Write content
- Spawn subagents

All work is delegated to the pipeline skill.
