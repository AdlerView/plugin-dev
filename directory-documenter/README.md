# Directory Documenter

Automatically document all subdirectories of a root path by creating `CLAUDE.md` files with directory structure and file descriptions.

## Features

- **Parallel Processing**: Documents multiple subdirectories simultaneously using sub-agents
- **Two-Part Prompt Architecture**: Sub-agents read instructions from a skill file, receive only variable parameters
- **Smart File Detection**: Identifies text files by extension and MIME type
- **Configurable Limits**: Control tree depth, files per folder, and hidden file inclusion
- **Safe Handling**: Skips binary files, large files (>1MB), and unreadable content
- **PDF Support**: Reads and describes PDF files (<5MB, max 10 per directory)
- **Concise Descriptions**: Generates brief (<20 words) descriptions for each text file

## Installation

The plugin is installed at `~/.claude/plugins/directory-documenter/`.

To use it, start Claude Code with the plugin directory:

```bash
claude --plugin-dir ~/.claude/plugins/directory-documenter
```

## Usage

```
/document <root-path> [options]
```

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `root-path` | *required* | Directory to document |
| `--depth=N` | 3 | Maximum tree depth |
| `--max-files=N` | 150 | Maximum files shown per folder |
| `--hidden` | false | Include hidden files/folders |

### Examples

```bash
# Document all projects in ~/Code
/document ~/Code

# Shallow documentation with limited files
/document ~/Projects --depth=2 --max-files=20

# Include hidden configuration directories
/document ~/.config --hidden --depth=4
```

## Architecture

### Two-Part Prompt Design

The plugin uses a unique two-part prompt architecture for sub-agents:

```
┌─────────────────────────────────────────────────────────────────┐
│  Sub-Agent Prompt                                               │
├─────────────────────────────────────────────────────────────────┤
│  PART 1 (Static): "Read skill file at .../SKILL.md"            │
│  ────────────────                                               │
│  Same for all sub-agents. Contains detailed instructions:       │
│  - Text file detection rules                                    │
│  - Output format specifications                                 │
│  - Skip rules and reasons                                       │
│  - Description guidelines                                       │
├─────────────────────────────────────────────────────────────────┤
│  PART 2 (Dynamic): Task parameters                              │
│  ────────────────                                               │
│  Unique for each sub-agent:                                     │
│  - Target directory path                                        │
│  - Depth, max-files, hidden settings                            │
└─────────────────────────────────────────────────────────────────┘
```

**Benefits:**
- **Maintainability**: Update skill file once, all sub-agents get new instructions
- **Consistency**: All sub-agents follow identical detailed instructions
- **Efficiency**: Shorter prompts, detailed instructions loaded only when needed
- **Clarity**: Clear separation between "how to do it" and "what to do"

### Workflow

```
/document ~/Projects --depth=2
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│  Claude Code (Orchestrator)                                     │
│  1. Parse parameters                                            │
│  2. Validate root path                                          │
│  3. List subdirectories                                         │
│  4. Spawn sub-agents in parallel with two-part prompts          │
│  5. Collect results and display summary                         │
└─────────────────────────────────────────────────────────────────┘
         │
         ├── Task(haiku) → Reads skill → Documents proj-a/ → CLAUDE.md
         ├── Task(haiku) → Reads skill → Documents proj-b/ → CLAUDE.md
         └── Task(haiku) → Reads skill → Documents proj-c/ → CLAUDE.md
                                    │
                                    ▼
                         All run in parallel
```

## Output Format

For each direct subdirectory, a `CLAUDE.md` file is created:

```markdown
# Documentation: /path/to/subdirectory

Generated: 2025-12-28T14:30:00Z

## Directory Structure

├── Package.swift          → Swift package manifest with dependencies
├── README.md              → Project documentation and setup guide
├── Sources/
│   ├── main.swift         → Application entry point
│   └── Config.swift       → Configuration loading utilities
├── build/
│   └── app                → [Binary file - skipped]
└── debug.log              → [Too large >1MB - skipped]
```

## Plugin Structure

```
~/.claude/plugins/directory-documenter/
├── .claude-plugin/
│   └── plugin.json                    # Plugin manifest
├── commands/
│   └── document.md                    # /document command
├── skills/
│   ├── directory-documentation/       # Orchestrator skill
│   │   └── SKILL.md                   # Workflow instructions
│   └── subagent-documentation-task/   # Sub-agent skill
│       ├── SKILL.md                   # Detailed task instructions
│       └── references/
│           └── text-extensions.md     # Known text file extensions
└── README.md
```

### Components

| Component | Purpose |
|-----------|---------|
| `commands/document.md` | Slash command with parameter parsing |
| `skills/directory-documentation/` | Orchestrator workflow, two-part prompt template |
| `skills/subagent-documentation-task/` | Sub-agent task instructions (read by each sub-agent) |
| `references/text-extensions.md` | Comprehensive list of text file extensions |

## File Detection

### Text Files (documented with description)

Files with these extensions are read and described:
- Source code: `.swift`, `.py`, `.js`, `.ts`, `.go`, `.rs`, `.c`, `.cpp`, etc.
- Markup: `.md`, `.html`, `.xml`, `.json`, `.yaml`, `.toml`
- Config: `.ini`, `.conf`, `.env`, `.gitignore`
- Documentation: `README`, `LICENSE`, `CHANGELOG`

For files without clear extensions, `file --mime-type` determines if readable.

### Skipped Files

- `[Binary file - skipped]` - Non-text files
- `[Too large >1MB - skipped]` - Files exceeding size limit
- `[No read permission - skipped]` - Inaccessible files
- `[Encrypted content - skipped]` - Protected files

## Error Handling

- Invalid root path: Reports error and stops
- No subdirectories: Reports "No subdirectories found"
- Sub-agent failure: Continues with others, marks as error in summary
- All errors isolated: One failure doesn't affect other subdirectories

## Customization

### Modifying Sub-Agent Behavior

Edit `skills/subagent-documentation-task/SKILL.md` to change:
- Output format
- Description guidelines
- Skip rules
- File size limits

All sub-agents will automatically use the updated instructions.

### Adding File Extensions

Edit `skills/subagent-documentation-task/references/text-extensions.md` to add or remove recognized text file extensions.

## License

MIT
