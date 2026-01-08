---
name: Sub-Agent Documentation Task
description: This skill provides instructions for sub-agents documenting a single directory. It should be read at the start of the documentation task to understand the output format, text file detection rules, and description guidelines.
version: 1.1.0
---

# Directory Documentation Task

Instructions for creating a CLAUDE.md file that documents a directory's structure and file contents.

---

## IMPORTANT: First Step - Check for Existing CLAUDE.md

**Before doing anything else, you MUST check if a CLAUDE.md file already exists in the target directory.**

### Step 1: Check Existence

```bash
test -f "{TARGET_DIR}/CLAUDE.md" && echo "EXISTS" || echo "NOT_EXISTS"
```

### Step 2: Handle Based on Result

**If CLAUDE.md does NOT exist:**
- Proceed directly to the documentation process (skip to "Your Task" section)

**If CLAUDE.md ALREADY exists:**
1. **Read the existing file first** using the Read tool
2. Note any useful context from the existing documentation (e.g., project purpose, special notes)
3. Proceed with the normal documentation process
4. **Delete the old file** before writing the new one:
   ```bash
   rm "{TARGET_DIR}/CLAUDE.md"
   ```
5. **Write the new CLAUDE.md** with fresh, up-to-date content

This ensures that:
- Existing documentation is acknowledged before being replaced
- The new documentation reflects the current state of the directory
- No stale information persists

---

## Your Task

Create a comprehensive `CLAUDE.md` documentation file for the assigned directory. The file must contain:
1. A header with the directory path and timestamp
2. A directory tree structure
3. Brief descriptions for each text file

## Output File Structure

Create the file at `{TARGET_DIR}/CLAUDE.md` with this format:

```markdown
# Documentation: {TARGET_DIR}

Generated: {ISO_TIMESTAMP}

## Directory Structure

{ASCII_TREE_WITH_DESCRIPTIONS}
```

## ASCII Tree Format

Use standard ASCII tree characters with descriptions after arrows:

```
├── file.txt           → Brief description of file contents
├── folder/
│   ├── nested.js      → Description of nested file
│   └── config.json    → Configuration settings for X
└── README.md          → Project documentation and setup guide
```

**Tree characters:**
- `├──` for items with siblings below
- `└──` for last item in a level
- `│   ` for vertical continuation
- Indent with 4 spaces per level

## Text File Detection

### Known Text Extensions

Read and describe files with these extensions:

**Source Code:**
txt, md, markdown, json, yaml, yml, toml, xml, html, htm, css, scss, sass, less, js, mjs, cjs, ts, mts, jsx, tsx, vue, svelte, swift, py, pyw, rb, php, go, rs, c, h, cpp, cc, cxx, hpp, java, kt, kts, scala, groovy, sh, bash, zsh, fish, ps1, bat, cmd, sql, r, lua, pl, pm, hs, ml, ex, exs, erl, clj, lisp, el, scm, nim, cr, d, v, asm, s

**Configuration:**
ini, cfg, conf, env, properties, editorconfig, prettierrc, eslintrc, babelrc, gitignore, gitattributes, gitmodules, dockerignore, npmrc, yarnrc

**Documentation:**
README (any extension), LICENSE, CHANGELOG, CONTRIBUTING, AUTHORS, MAINTAINERS, CODEOWNERS

**Build & CI:**
Makefile, GNUmakefile, CMakeLists.txt, Dockerfile, Vagrantfile, Gemfile, Rakefile, Podfile, Fastfile, Dangerfile, Brewfile

### Fallback Detection

For files without recognized extensions, use:

```bash
file --mime-type -b "$filepath"
```

If result starts with `text/` or is `application/json`, `application/xml`, `application/x-yaml` → treat as text file.

## PDF File Handling

**PDF files require special handling.** They are readable but need size limits.

### PDF Rules

| Rule | Value |
|------|-------|
| Max PDF size | 5 MB |
| Max PDFs per subagent | 10 |
| Extension | `.pdf` |

### PDF Processing

1. **Identify PDFs**: Files with `.pdf` extension
2. **Check size**: Skip PDFs larger than 5 MB
3. **Count limit**: Only process first 10 PDFs encountered (sorted alphabetically)
4. **Read PDF**: Use the Read tool (Claude Code supports PDF reading)
5. **Generate description**: Summarize document purpose/content in <20 words

### PDF Skip Notes

| Condition | Skip Note |
|-----------|-----------|
| PDF > 5 MB | `→ [PDF too large >5MB - skipped]` |
| PDF limit exceeded | `→ [PDF limit reached (max 10) - skipped]` |

### PDF in Output Summary

Track PDFs separately in the JSON summary:

```json
{
  "pdfs_processed": 5,
  "pdfs_skipped": 2,
  "pdf_skip_reasons": {
    "too_large": 1,
    "limit_exceeded": 1
  }
}
```

## Skip Rules

Do NOT read these files. Instead, add a skip note in the tree:

| Condition | Skip Note |
|-----------|-----------|
| Binary file (non-text MIME) | `→ [Binary file - skipped]` |
| File size > 1MB | `→ [Too large >1MB - skipped]` |
| No read permission | `→ [No read permission - skipped]` |
| Encrypted/unreadable | `→ [Encrypted content - skipped]` |

## Reading Text Files

### Efficient Reading

Only read what's necessary for a reliable description:
- **First 500 lines** OR **first 20KB** (whichever is less)
- For structured formats (JSON, YAML): Focus on top-level keys/metadata
- For code: Focus on imports, class/function names, comments

### Description Guidelines

Generate descriptions that are:
- **Brief**: Maximum 20 words
- **Factual**: Describe purpose or content objectively
- **Neutral**: No speculation or assumptions
- **Safe**: NEVER include sensitive content verbatim

**Good descriptions:**
- `Swift package manifest defining project dependencies and targets`
- `Application entry point with CLI argument parsing`
- `Unit tests for authentication service`
- `ESLint configuration for TypeScript projects`

**Conservative fallbacks** (when content is unclear):
- `Configuration file for [detected service/tool]`
- `Source code module for [detected functionality]`
- `Data file in [format] format`
- `Script for [detected purpose]`

## Tree Limits

Respect the configured limits:

- **Max files per folder**: Show up to `{MAX_FILES}` files, then add:
  ```
  └── ... and 23 more files
  ```

- **Max depth**: Stop at `{DEPTH}` levels, don't recurse deeper

- **Hidden files**: Only include if `{INCLUDE_HIDDEN}` is true
  - Hidden = name starts with `.`

## Error Handling

- If directory doesn't exist: Return error status
- If no read permission: Return error status
- If individual file fails: Skip with note, continue with others
- Never crash the entire task for a single file error

## Output Summary

After creating CLAUDE.md, return this JSON summary:

```json
{
  "status": "success",
  "subdir": "{TARGET_DIR}",
  "claude_md_path": "{TARGET_DIR}/CLAUDE.md",
  "had_existing_claude_md": true,
  "total_files": 45,
  "text_files_documented": 32,
  "pdfs_processed": 5,
  "pdfs_skipped": 2,
  "files_skipped": 13,
  "skip_reasons": {
    "binary": 8,
    "too_large": 3,
    "no_permission": 2,
    "pdf_too_large": 1,
    "pdf_limit_exceeded": 1
  }
}
```

**Note the `had_existing_claude_md` field** - set to `true` if an existing CLAUDE.md was found and replaced, `false` if this was a fresh creation.

On error:

```json
{
  "status": "error",
  "subdir": "{TARGET_DIR}",
  "error": "Description of what went wrong"
}
```

## Complete Workflow Summary

```
1. CHECK: Does {TARGET_DIR}/CLAUDE.md exist?
   │
   ├── YES: Read existing file → Note context → Continue
   │
   └── NO: Continue directly

2. SCAN: List directory contents respecting limits

3. ANALYZE: For each file:
   - Detect if text file (extension or MIME)
   - For PDFs: Check size (<5MB) and count (max 10)
   - Read first 500 lines if text, or full PDF if within limits
   - Generate <20 word description
   - Or mark as skipped with reason

4. BUILD: Construct ASCII tree with descriptions

5. WRITE:
   - If existing CLAUDE.md: Delete it first
   - Write new CLAUDE.md file

6. RETURN: JSON summary with statistics
```

## Example Output

A complete CLAUDE.md example:

```markdown
# Documentation: /Users/home/Projects/MyApp

Generated: 2025-12-28T15:30:00Z

## Directory Structure

├── Package.swift          → Swift package manifest with SPM dependencies
├── README.md              → Project overview and installation instructions
├── Sources/
│   ├── main.swift         → CLI entry point with argument parsing
│   ├── App/
│   │   ├── AppDelegate.swift  → Application lifecycle management
│   │   └── Config.swift       → Environment configuration loader
│   └── Utils/
│       ├── Logger.swift       → Structured logging with levels
│       └── Extensions.swift   → String and Date helper extensions
├── Tests/
│   └── AppTests/
│       └── ConfigTests.swift  → Unit tests for configuration loading
├── Resources/
│   ├── icon.png           → [Binary file - skipped]
│   └── Localizable.strings → Localization strings for UI
├── build/
│   ├── release            → [Binary file - skipped]
│   └── debug.log          → [Too large >1MB - skipped]
└── .env.local             → [No read permission - skipped]
```
