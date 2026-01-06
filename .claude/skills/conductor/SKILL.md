---
name: conductor
description: |
  Context-driven development methodology for organized, spec-first coding. Use when:
  - Project has a `conductor/` directory
  - User mentions specs, plans, tracks, or context-driven development
  - Files like `conductor/tracks.md`, `conductor/product.md`, `conductor/workflow.md` exist
  - User asks about project status, implementation progress, or track management
  - User wants to organize development work with TDD practices
  - User invokes `/conductor-*` commands (setup, newtrack, implement, status, revert, validate, block, skip, revise, archive, export, refresh)
  - User mentions documentation is outdated or wants to sync context with codebase changes
  
  Interoperable with Gemini CLI extension and Claude Code commands.
  Integrates with Beads for persistent task memory across sessions.
---

# Conductor: Context-Driven Development

Measure twice, code once.

## Overview

Conductor enables context-driven development by:
1. Establishing project context (product vision, tech stack, workflow)
2. Organizing work into "tracks" (features, bugs, improvements)
3. Creating specs and phased implementation plans
4. Executing with TDD practices and progress tracking
5. **Parallel execution** of independent tasks using sub-agents

## Gastown Integration (Multi-Agent Orchestration)

Conductor integrates with [Gastown](https://github.com/steveyegge/gastown) for enhanced parallel execution:

### Detection
```bash
# Check if Gastown is available
if which gt > /dev/null 2>&1; then
  GASTOWN_ENABLED=true
fi
```

### Gastown Architecture
Gastown uses a **Town** (workspace) model:
- **Town** (`~/gt/`): Central workspace managing multiple projects
- **Rig**: A project repository added to the Town
- **Polecat**: Ephemeral worker agents (spawn → work → disappear)
- **Crew**: Long-running workspaces for ongoing development
- **Mayor**: AI coordinator (primary interface via `gt mayor attach`)
- **Witness**: Monitors polecats, handles stuck workers
- **Refinery**: Automated merge queue for parallel branches

### Benefits Over Native Task()
- **Polecat workers**: Purpose-built ephemeral agents with isolated worktrees
- **Durability**: Work state persists in Beads, survives crashes/compaction
- **Refinery**: Automated merge queue for parallel branches
- **Convoy tracking**: Batch progress monitoring with `gt convoy list` and `gt dashboard`

### Gastown CLI Commands (`gt`)
| Command | Description |
|---------|-------------|
| `gt install <path> --git` | Initialize Town workspace |
| `gt rig add <name> <repo-url>` | Add project repository to Town |
| `gt status` | Check Town/rig status |
| `gt sling <task-id> <rig-name>` | Dispatch task to polecat worker |
| `gt hook` | Check what work is assigned (inside polecat) |
| `gt convoy create "<desc>" <epic-id> --human` | Create convoy for batch tracking |
| `gt convoy list` | Show active convoys |
| `gt convoy status <id>` | Detailed convoy progress |
| `gt handoff` | Cycle polecat session (preserve sandbox) |
| `gt done --exit` | Signal completion, exit polecat |
| `gt dashboard --port 8080` | Launch monitoring web UI |
| `gt agents` | Navigate between agent sessions |

### Conductor Commands
- `/conductor-dispatch [track_id]` - Dispatch track to Gastown polecats
- Use `gt convoy list` to monitor progress
- Use `gt dashboard --port 8080` for web UI
- Use `gt agents` to navigate between sessions

### Fallback
If Gastown unavailable, Conductor uses native `Task()` sub-agents with `parallel_state.json`.

See: [docs/GASTOWN_INTEGRATION.md](../../../docs/GASTOWN_INTEGRATION.md)

## Parallel Execution (New Feature)

Conductor now supports parallel task execution for eligible phases:

### How It Works
- During `/conductor-newtrack`, tasks are analyzed for parallelization potential
- Tasks with no file conflicts and no dependencies can run in parallel
- Parallel phases use `<!-- execution: parallel -->` annotation
- Each task has `<!-- files: ... -->` for exclusive file ownership
- Dependencies use `<!-- depends: ... -->` annotation

### Plan.md Format for Parallel Phases
```markdown
## Phase 1: Core Setup
<!-- execution: parallel -->

- [ ] Task 1: Create auth module
  <!-- files: src/auth/index.ts, src/auth/index.test.ts -->
  
- [ ] Task 2: Create config module
  <!-- files: src/config/index.ts -->
  
- [ ] Task 3: Create utilities
  <!-- files: src/utils/index.ts -->
  <!-- depends: task1 -->
```

### Execution Flow
1. **Coordinator** parses parallel annotations
2. Detects file conflicts (fails safe if found)
3. Spawns sub-agents via `Task()` for independent tasks
4. Monitors `parallel_state.json` for completion
5. Aggregates results and updates plan.md

### When to Use Parallel Execution
- ✅ Tasks modifying different files
- ✅ Independent components (auth, config, utils)
- ✅ Multiple test file creation
- ❌ Tasks with shared state
- ❌ Tasks with sequential dependencies

## Context Loading

When this skill activates, load these files to understand the project:
1. `conductor/product.md` - Product vision and goals
2. `conductor/tech-stack.md` - Technology constraints
3. `conductor/workflow.md` - Development methodology (TDD, commits)
4. `conductor/tracks.md` - Current work status

**Important**: Conductor commits locally but never pushes. Users decide when to push to remote.

For active tracks, also load:
- `conductor/tracks/<track_id>/spec.md`
- `conductor/tracks/<track_id>/plan.md`

## Beads Integration

Beads integration is **always attempted** for persistent task memory. If `bd` CLI is unavailable or fails, the user can choose to continue without it.

### Detection (MUST check before using bd commands)

Before using ANY `bd` command, you MUST verify:
1. `bd` CLI is installed: `which bd` returns a path
2. `conductor/beads.json` exists AND has `"enabled": true`

```bash
# Check availability - run this before any bd command
if which bd > /dev/null 2>&1 && [ -f conductor/beads.json ]; then
  BEADS_ENABLED=$(cat conductor/beads.json | grep -o '"enabled"[[:space:]]*:[[:space:]]*true' || echo "")
  if [ -n "$BEADS_ENABLED" ]; then
    # Beads is available and enabled - use bd commands
  fi
fi
```

### If Beads is NOT available:
- **DO NOT** run any `bd` commands
- Use only plan.md markers for task tracking
- All conductor commands work normally without Beads

### If Beads IS available:
- Tracks become Beads epics
- Tasks sync to Beads for persistent memory
- Use `bd ready` instead of manual task selection
- Notes survive context compaction

### Beads CLI Commands (`bd`)
| Command | Description |
|---------|-------------|
| `bd init` | Initialize Beads in project (creates `.beads/`) |
| `bd create "<desc>" -p <priority>` | Create task (use `--json \| jq -r '.id'` to capture ID) |
| `bd list` | List all open tasks |
| `bd list --parent <epic-id>` | List children of an epic |
| `bd ready` | List unblocked, ready-to-work tasks |
| `bd show <id>` | Show task details |
| `bd update <id> --status in_progress` | Mark task as in-progress |
| `bd update <id> --notes "..."` | Add notes (survives compaction!) |
| `bd close <id> --reason "..."` | Complete a task |
| `bd dep add <child> <parent>` | Set dependency (child waits for parent) |
| `bd sync` | Push local changes to remote |
| `bd compact --auto` | Archive old completed tasks |

## Proactive Behaviors

1. **On new session**: Check for in-progress tracks, offer to resume
2. **On task completion**: Suggest next task or phase verification
3. **On blocked detection**: Alert user and suggest alternatives
4. **On all tasks complete**: Congratulate and offer archive/cleanup
5. **On stale context detected**: If setup >2 days old or significant codebase changes detected, suggest `/conductor-refresh`
6. **On Beads available**: If `bd` CLI detected during setup, offer integration

## Intent Mapping

| User Intent | Command |
|-------------|---------|
| "Set up this project" | `/conductor-setup` |
| "Create a new feature" | `/conductor-newtrack [desc]` |
| "Start working" / "Implement" | `/conductor-implement [id]` |
| "What's the status?" | `/conductor-status` |
| "Undo that" / "Revert" | `/conductor-revert` |
| "Check for issues" | `/conductor-validate` |
| "This is blocked" | `/conductor-block` |
| "Skip this task" | `/conductor-skip` |
| "This needs revision" / "Spec is wrong" | `/conductor-revise` |
| "Save context" / "Handoff" / "Transfer to next section" | `/conductor-handoff` |
| "Archive completed" | `/conductor-archive` |
| "Export summary" | `/conductor-export` |
| "Docs are outdated" / "Sync with codebase" | `/conductor-refresh` |
| "Dispatch to Gastown" / "Use polecats" | `/conductor-dispatch` |

## References

- **Detailed workflows**: [references/workflows.md](references/workflows.md) - Step-by-step command execution
- **Directory structure**: [references/structure.md](references/structure.md) - File layout and status markers
- **Beads integration**: [references/beads-integration.md](references/beads-integration.md)
