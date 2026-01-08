---
description: Document all subdirectories of a root path with CLAUDE.md files
argument-hint: <root-path> [--depth=3] [--max-files=150] [--hidden]
allowed-tools: [Task, Bash, Read, Write, Glob]
---

# Document Directory Structure

Automatically document all direct subdirectories of a given root path by creating CLAUDE.md files with directory structure and file descriptions.

## Parameter Parsing

Parse the arguments provided in $ARGUMENTS:

1. **root-path** (required): First positional argument - the directory to document
2. **--depth=N** (optional): Maximum tree depth (default: 3)
3. **--max-files=N** (optional): Maximum files per folder (default: 150)
4. **--hidden** (optional): Include hidden files/folders (default: false)

Example invocations:
- `/document /Users/home/Projects`
- `/document ~/Code --depth=5`
- `/document /var/log --max-files=20 --hidden`

## Execution Workflow

Follow the workflow defined in the `directory-documentation` skill.

### Step 1: Parse and Validate

1. Extract root-path from $1 or first argument
2. Parse optional flags from remaining arguments
3. Set defaults: depth=3, max_files=150, include_hidden=false
4. Verify path exists and is readable

### Step 2: List Subdirectories

```bash
# Without hidden
find "$ROOT_PATH" -maxdepth 1 -mindepth 1 -type d -not -name '.*' | sort

# With hidden
find "$ROOT_PATH" -maxdepth 1 -mindepth 1 -type d | sort
```

### Step 3: Spawn Parallel Sub-Agents

For EACH subdirectory, spawn a Task agent using the **Two-Part Prompt**:

**Task Configuration:**
- `subagent_type`: "general-purpose"
- `model`: "haiku"
- `prompt`: Two-part prompt (see below)

**CRITICAL**: Launch ALL Task tool calls in a SINGLE message to run them in parallel.

### Two-Part Prompt Template

Each sub-agent receives this prompt structure:

```
INSTRUCTIONS:
First, read the skill file that contains your detailed task instructions:

Read file: ${CLAUDE_PLUGIN_ROOT}/skills/subagent-documentation-task/SKILL.md

Follow all instructions in that skill file to complete your task.

---

TASK PARAMETERS:
- Target Directory: [ACTUAL_SUBDIR_PATH]
- Maximum Tree Depth: [DEPTH_VALUE]
- Maximum Files Per Folder: [MAX_FILES_VALUE]
- Include Hidden Files: [true/false]

Execute the documentation task for the target directory using the parameters above.
Return a JSON summary as specified in the skill file.
```

**Replace placeholders** with actual values for each subdirectory.

### Step 4: Collect and Summarize

After all sub-agents complete, display a summary table:

```
╔══════════════════════════════════════════════════════════════════════╗
║                    DOCUMENTATION COMPLETE                            ║
╠══════════════════════════════════════════════════════════════════════╣
║ Root: /path/to/root                                                  ║
║ Parameters: depth=3, max-files=150, hidden=false                     ║
║ Subdirectories: 5 processed                                          ║
╠═══════════════════╦════════╦═══════════╦══════════╦═════════════════╣
║ Directory         ║ Status ║ Total     ║ Documented║ Skipped        ║
╠═══════════════════╬════════╬═══════════╬══════════╬═════════════════╣
║ project-a/        ║   ✓    ║    45     ║    32    ║ 13 (binary)    ║
║ project-b/        ║   ✓    ║    23     ║    20    ║ 3 (too large)  ║
║ project-c/        ║   ✗    ║     -     ║     -    ║ Permission err ║
╚═══════════════════╩════════╩═══════════╩══════════╩═════════════════╝

Created CLAUDE.md files:
  → /path/to/root/project-a/CLAUDE.md
  → /path/to/root/project-b/CLAUDE.md
```

## Error Handling

- Invalid root path: Report error and stop
- No subdirectories: Report "No subdirectories found" and stop
- Sub-agent failure: Continue with others, mark as error in summary
- All errors are isolated and don't affect other operations

## Architecture

```
/document ~/Projects
    │
    ├── Parse: depth=3, max_files=150, hidden=false
    ├── Validate: ~/Projects exists
    ├── List subdirs: [app-a, app-b, app-c]
    │
    │   ┌── Part 1: "Read skill file..."    ──┐
    │   └── Part 2: "Target: app-a, ..."    ──┼── Prompt per sub-agent
    │
    ├── Task(haiku, prompt) for app-a/ ──┐
    ├── Task(haiku, prompt) for app-b/ ──┼── All parallel
    └── Task(haiku, prompt) for app-c/ ──┘
              │
              │ Each sub-agent:
              │ 1. Reads skill file
              │ 2. Follows instructions
              │ 3. Creates CLAUDE.md
              │ 4. Returns JSON summary
              │
              ▼
       Collect results → Display summary
```
