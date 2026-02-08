# Technology Stack

## Platform Architecture

### Distribution Format
- **No runtime code** - Conductor-Beads is a configuration-based toolkit
- **Markup & Declarative**: Markdown commands, TOML configs, JSON metadata
- **Zero Dependencies**: Works with any AI agent that supports slash commands or extensions

### Command Formats

| Platform | Format | Location |
|----------|--------|----------|
| **Claude Code** | Markdown (`.md`) | `.claude/commands/` |
| **Gemini CLI** | TOML (`.toml`) | `commands/conductor/` |

### Skills

- **Format**: SKILL.md specification + references/
- **Location**: `.claude/skills/` (Claude), shareable to compatible CLIs
- **Languages**: Documentation only (no code execution)

## External Dependencies

### Required
- **Beads CLI** (`bd`): Persistent task memory
  - Install: `npm install -g @beads/bd` or `brew install steveyegge/beads/bd`
  - Repository: https://github.com/steveyegge/beads
  - Used for: Task graph, dependency tracking, cross-session memory

### Optional
- **Git**: Version control integration (used by all commands)
- **Python 3.6+**: For skill-creator utility scripts (init_skill.py, package_skill.py, quick_validate.py)

## File Formats

- **Context Files**: Markdown (`.md`)
- **Configuration**: JSON (`.json`)
- **Gemini Commands**: TOML (`.toml`)
- **Templates**: Markdown (`.md`)
- **State Files**: JSON (`.json`)

## Template Management

Located in `templates/`:
- `workflow.md` - TDD workflow and commit conventions
- `code_styleguides/` - Language-specific style templates
  - bash.md
  - go.md
  - java.md
  - javascript.md
  - python.md
  - rust.md
  - typescript.md

## Integration Points

1. **Git Integration**
   - Commit history tracking
   - Branch management
   - Git notes for audit trails
   - **Policy**: Local commits only (never push automatically)

2. **Beads Integration**
   - Task creation: `bd create`
   - Status sync: `bd update`, `bd done`
   - Dependency management: `bd dep add`
   - Ready queue: `bd ready`
   - Context loading: `bd show`, `bd prime`

3. **AI Agent Integration**
   - Natural language → command mapping (via skills)
   - Context injection (via CLAUDE.md, GEMINI.md)
   - Sub-agent spawning (for parallel execution)

## Design Principles

1. **Configuration over Code**: Use markup and declarative formats
2. **Platform Agnostic**: Same `conductor/` structure across all platforms
3. **Graceful Degradation**: Works without Beads (prompts user to continue)
4. **Human-Readable**: All artifacts are readable markdown/JSON
5. **Version Control Friendly**: Text-based formats for easy diffing
