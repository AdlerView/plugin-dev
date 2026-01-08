---
name: Directory Documentation
description: This skill should be used when the user asks to "document directories", "create CLAUDE.md files", "document folder structure", "generate directory documentation", or wants to automatically document subdirectories of a root path. Provides the complete workflow for orchestrating parallel sub-agents to create documentation files.
version: 1.2.0
---

# Directory Documentation Workflow

Orchestrate the automatic documentation of all direct subdirectories of a given root path by spawning parallel sub-agents that create CLAUDE.md files.

## Overview

This skill provides:
1. Complete workflow steps for the orchestrator (Claude Code main session)
2. Two-part prompt template for sub-agents (skill reference + variables)
3. **Verification step** to confirm CLAUDE.md files were actually created
4. Summary generation format

## Orchestrator Workflow

Execute these steps when the user invokes `/document`:

### Step 1: Parse Parameters

Extract from user input:
- `root_path` (required): The directory to document
- `depth` (optional, default: 3): Maximum tree depth
- `max_files` (optional, default: 150): Maximum files shown per folder
- `include_hidden` (optional, default: false): Whether to include hidden files/folders

### Step 2: Validate Root Path

```bash
test -d "$ROOT_PATH" && test -r "$ROOT_PATH" && echo "VALID" || echo "INVALID"
```

If invalid, report error and stop.

### Step 3: List Direct Subdirectories

```bash
# Without hidden
find "$ROOT_PATH" -maxdepth 1 -mindepth 1 -type d -not -name '.*' | sort

# With hidden
find "$ROOT_PATH" -maxdepth 1 -mindepth 1 -type d | sort
```

Store the list for tracking purposes.

### Step 4: Spawn Sub-Agents in Parallel

For each subdirectory, use the Task tool with:
- `subagent_type`: "general-purpose"
- `model`: "haiku"
- `prompt`: Use the **Two-Part Prompt Template** below

**IMPORTANT**: Launch ALL Task calls in a single message to run them in parallel.

### Step 5: Collect Results from Sub-Agents

Wait for all sub-agents to complete. Each sub-agent returns a JSON summary with:
- `status`: "success" or "error"
- `subdir`: The target directory path
- `claude_md_path`: Path to created CLAUDE.md
- `had_existing_claude_md`: Whether an existing file was replaced
- Statistics (total_files, text_files_documented, files_skipped)

### Step 6: ⚠️ MANDATORY VERIFICATION - READ ALL CREATED CLAUDE.md FILES ⚠️

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║  ⚠️  CRITICAL: YOU MUST READ EACH CLAUDE.md FILE TO VERIFY IT EXISTS  ⚠️     ║
║                                                                               ║
║  DO NOT trust sub-agent reports. Sub-agents may claim success without         ║
║  actually creating the file. YOU MUST verify by READING the files.            ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

**THIS STEP IS NOT OPTIONAL. YOU MUST:**

1. **USE THE READ TOOL** to read each expected CLAUDE.md file
2. **DO NOT** just check if the file exists with bash `test -f`
3. **ACTUALLY READ** the file content to confirm it was written correctly

### Required Verification Process

For EACH subdirectory that a sub-agent processed:

```
┌─────────────────────────────────────────────────────────────────────────┐
│  FOR EACH SUBDIR:                                                       │
│                                                                         │
│  1. Use Read tool: Read("{SUBDIR}/CLAUDE.md")                          │
│                                                                         │
│  2. Check result:                                                       │
│     ├── File readable + has content → ✓ VERIFIED                       │
│     ├── File not found              → ✗ MISSING (sub-agent failed)     │
│     └── File empty/malformed        → ⚠ INCOMPLETE                     │
│                                                                         │
│  3. Record verification status for summary                              │
└─────────────────────────────────────────────────────────────────────────┘
```

### Verification Code Pattern

```
For each subdirectory in [processed_subdirs]:
    Use Read tool to read: {subdir}/CLAUDE.md

    If file exists and has content:
        → Mark as VERIFIED
        → Note file size/line count

    If file not found or empty:
        → Mark as FAILED
        → Flag for user attention
```

### Why Reading is Required (Not Just Existence Check)

| Method | Problem |
|--------|---------|
| `test -f` | File might exist but be empty |
| `ls` | File might exist but be corrupted |
| **Read tool** | ✓ Confirms file exists AND has valid content |

### Verification Outcomes

| Sub-Agent Report | Read Result | Final Status |
|------------------|-------------|--------------|
| success | Content readable | ✓ **VERIFIED** |
| success | File not found | ✗ **FAILED** (sub-agent lied) |
| success | File empty | ⚠ **INCOMPLETE** |
| error | N/A | ✗ **ERROR** (expected) |

### Step 7: Generate Summary with Verification Status

**Only after reading all CLAUDE.md files**, generate the summary table.

Include verification column showing READ confirmation status.

---

## Two-Part Prompt Template

The sub-agent prompt consists of two parts:

### Part 1: Skill Reference (Static)

This part is identical for all sub-agents:

```
INSTRUCTIONS:
First, read the skill file that contains your detailed task instructions:

Read file: ${CLAUDE_PLUGIN_ROOT}/skills/subagent-documentation-task/SKILL.md

Follow all instructions in that skill file to complete your task.
```

### Part 2: Task Variables (Dynamic)

This part is specific to each sub-agent:

```
TASK PARAMETERS:
- Target Directory: {SUBDIR_PATH}
- Maximum Tree Depth: {DEPTH}
- Maximum Files Per Folder: {MAX_FILES}
- Include Hidden Files: {INCLUDE_HIDDEN}

Execute the documentation task for the target directory using the parameters above.
Return a JSON summary as specified in the skill file.
```

### Complete Prompt Template

Combine both parts when spawning each sub-agent:

```
INSTRUCTIONS:
First, read the skill file that contains your detailed task instructions:

Read file: ${CLAUDE_PLUGIN_ROOT}/skills/subagent-documentation-task/SKILL.md

Follow all instructions in that skill file to complete your task.

---

TASK PARAMETERS:
- Target Directory: {SUBDIR_PATH}
- Maximum Tree Depth: {DEPTH}
- Maximum Files Per Folder: {MAX_FILES}
- Include Hidden Files: {INCLUDE_HIDDEN}

Execute the documentation task for the target directory using the parameters above.
Return a JSON summary as specified in the skill file.
```

---

## Console Summary Format

After verification, display:

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                         DOCUMENTATION COMPLETE                                ║
╠══════════════════════════════════════════════════════════════════════════════╣
║ Root: /path/to/root                                                          ║
║ Parameters: depth=3, max-files=150, hidden=false                             ║
║ Subdirectories: 5 processed                                                  ║
╠════════════════════╦════════╦══════════╦═══════════╦══════════╦═════════════╣
║ Directory          ║ Status ║ Verified ║ Total     ║ Documented║ Skipped    ║
╠════════════════════╬════════╬══════════╬═══════════╬══════════╬═════════════╣
║ project-a/         ║   ✓    ║    ✓     ║    45     ║    32    ║ 13 (binary)║
║ project-b/         ║   ✓    ║    ✓     ║    23     ║    20    ║ 3 (large)  ║
║ project-c/         ║   ✗    ║    -     ║     -     ║     -    ║ Perm error ║
║ project-d/         ║   ✓    ║    ✗     ║    10     ║     8    ║ FILE MISSING║
╚════════════════════╩════════╩══════════╩═══════════╩══════════╩═════════════╝

Verified CLAUDE.md files:
  ✓ /path/to/root/project-a/CLAUDE.md
  ✓ /path/to/root/project-b/CLAUDE.md
  ✗ /path/to/root/project-d/CLAUDE.md (MISSING - sub-agent reported success but file not found)

Summary:
  - Total subdirectories: 5
  - Successfully documented: 2
  - Failed: 1
  - Verification failed: 1
```

---

## Architecture Diagram

```
/document ~/Projects --depth=2
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Claude Code (Orchestrator)                                         │
│  ═══════════════════════════                                        │
│  1. Parse parameters: depth=2, max_files=150, hidden=false          │
│  2. Validate root path                                              │
│  3. List subdirectories: [proj-a, proj-b, proj-c]                   │
│  4. For each subdir, spawn Task with two-part prompt                │
│  5. Wait for all sub-agents                                         │
│  6. ⚠️ MANDATORY: READ each CLAUDE.md file to verify existence      │
│  7. Display summary with READ verification status                   │
└─────────────────────────────────────────────────────────────────────┘
         │
         ├── Task(haiku) → Reads skill → Documents proj-a/ → CLAUDE.md
         ├── Task(haiku) → Reads skill → Documents proj-b/ → CLAUDE.md
         └── Task(haiku) → Reads skill → Documents proj-c/ → CLAUDE.md
                                    │
                                    ▼
                         All run in parallel
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  ⚠️ MANDATORY VERIFICATION PHASE ⚠️                                 │
│  ════════════════════════════════                                   │
│                                                                     │
│  For EACH subdir that reported success:                             │
│                                                                     │
│    ❌ DO NOT: test -f "$subdir/CLAUDE.md"                           │
│    ✅ MUST DO: Use Read tool → Read("$subdir/CLAUDE.md")            │
│                                                                     │
│  Only READ confirms file exists AND has valid content!              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Verification Implementation

**Use the Read tool, NOT bash commands:**

```
For each processed subdirectory:
    → Read("{subdir}/CLAUDE.md")
    → If readable with content: VERIFIED ✓
    → If not found or empty: FAILED ✗
```

**Remember: The Read tool is the ONLY reliable way to verify file creation.**

---

## Benefits of Two-Part Prompt

1. **Maintainability**: Update skill file once, all sub-agents get new instructions
2. **Consistency**: All sub-agents follow identical detailed instructions
3. **Efficiency**: Shorter prompts, skill loaded only when needed
4. **Clarity**: Clear separation between "how to do it" and "what to do"

## Error Handling

- Sub-agent errors are isolated and don't affect others
- Failed sub-agents report error status in summary
- **Verification catches silent failures** where sub-agent reports success but file is missing
- The orchestrator continues with remaining sub-agents
- No temporary files or processes left behind

## Why Verification Matters

Sub-agents may report success without actually completing the task due to:
- Context limits causing early termination
- Write permission issues not properly caught
- Path resolution errors
- Disk space issues
- Race conditions with file deletion/creation

**Always verify. Never trust blindly.**
