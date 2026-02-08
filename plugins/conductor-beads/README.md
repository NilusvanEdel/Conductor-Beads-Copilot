# Conductor-Beads Plugin

**Context-Driven Development** framework with persistent memory for AI coding assistants.

Combines:
- **Conductor**: Structured planning methodology (specs, plans, tracks, TDD workflows)
- **Beads**: Persistent task tracking that survives conversation compaction

## Quick Installation

```bash
copilot plugin install conductor-beads@conductor-beads
```

After installation, commands are available with the `/conductor-beads:` namespace:

```bash
/conductor-beads:setup      # Initialize project
/conductor-beads:newtrack   # Create feature/bug track
/conductor-beads:implement  # Execute tasks from plan
/conductor-beads:status     # Show progress overview
```

## What's Included

### Commands (17 slash commands)

| Command | Description |
|---------|-------------|
| `/conductor-beads:setup` | Initialize project with context files |
| `/conductor-beads:newtrack` | Create feature/bug track with spec and plan |
| `/conductor-beads:implement` | Execute tasks from track's plan (TDD workflow) |
| `/conductor-beads:status` | Display progress overview |
| `/conductor-beads:revert` | Git-aware revert of tracks, phases, or tasks |
| `/conductor-beads:validate` | Validate project integrity |
| `/conductor-beads:block` | Mark task as blocked with reason |
| `/conductor-beads:skip` | Skip current task with justification |
| `/conductor-beads:revise` | Update spec/plan when implementation reveals issues |
| `/conductor-beads:archive` | Archive completed tracks |
| `/conductor-beads:export` | Generate project summary |
| `/conductor-beads:handoff` | Create context handoff for section transfer |
| `/conductor-beads:refresh` | Sync context docs with current codebase state |
| `/conductor-beads:formula` | List and manage track templates (Beads formulas) |
| `/conductor-beads:wisp` | Create ephemeral exploration track (no audit trail) |
| `/conductor-beads:distill` | Extract reusable template from completed track |

### Skills (3 auto-activating skills)

- **conductor**: Context-driven development methodology (activates when `conductor/` exists)
- **beads**: Persistent task memory (activates when `.beads/` exists)
- **skill-creator**: Guide for creating new AI agent skills

## Documentation

For complete documentation, see the [main README](../../README.md).

- [Manual Workflow Guide](../../docs/manual-workflow-guide.md)
- [Beads Integration](../../docs/BEADS_INTEGRATION.md)
- [Parallel Execution](../../docs/PARALLEL_EXECUTION.md)

## Alternative Installation Methods

### Claude Code (Manual)

```bash
cp -r .claude/commands ~/.claude/commands/
cp -r .claude/skills ~/.claude/skills/
```

Commands available as `/conductor-*` (no namespace prefix)

### Gemini CLI (Extension)

```bash
gemini extensions install https://github.com/NguyenSiTrung/Conductor-Beads
```

Commands available as `/conductor:*` format

## Key Differences

| Platform | Command Format | Example |
|----------|---------------|---------|
| **Copilot CLI** | `/conductor-beads:command` | `/conductor-beads:setup` |
| **Claude Code** | `/conductor-command` | `/conductor-setup` |
| **Gemini CLI** | `/conductor:command` | `/conductor:setup` |

All three platforms access the same underlying commands - only the namespace differs.

## Prerequisites

**Optional but Recommended:** Install [Beads CLI](https://github.com/steveyegge/beads) for persistent task memory:

```bash
npm install -g @beads/bd
# or
brew install steveyegge/beads/bd
```

## License

Apache-2.0

## Source

This plugin is part of [Conductor-Beads](https://github.com/NguyenSiTrung/Conductor-Beads), an open-source Context-Driven Development toolkit.
