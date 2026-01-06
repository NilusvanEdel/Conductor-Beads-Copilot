# Gastown Integration Design

This document outlines how Conductor-Beads integrates with [Gastown](https://github.com/steveyegge/gastown) for multi-agent orchestration.

## Prerequisites

Before using `/conductor-dispatch`, you need:

### Required

#### 1. Go 1.23+
```bash
# Check Go version
go version  # Should be 1.23 or higher

# Install Go: https://go.dev/dl/
```

#### 2. Git 2.25+
```bash
git --version  # Should be 2.25 or higher
```

#### 3. Beads CLI (`bd`)
```bash
# Install Beads
go install github.com/steveyegge/beads/cmd/bd@latest

# Verify
bd --version
```

#### 4. Gastown CLI (`gt`)
```bash
# Install Gastown
go install github.com/steveyegge/gastown/cmd/gt@latest

# Verify
gt --version
```

### Recommended (for full experience)

#### 5. tmux 3.0+
```bash
# Check tmux
tmux -V  # Should be 3.0 or higher

# Install (Ubuntu/Debian)
sudo apt install tmux

# Install (macOS)
brew install tmux
```

**Without tmux:** Gastown runs in "minimal mode" - you manually start agents with `claude --resume`. State tracking still works.

**With tmux:** Full orchestration - Mayor session, automatic polecat lifecycle, interactive monitoring.

#### 6. Claude Code CLI
```bash
# Required for agent sessions
claude --version
```

### Gastown Workspace Setup

Gastown uses a **Town** (workspace) model that manages multiple project **Rigs**:

```bash
# 1. Create Gastown workspace (one-time setup)
gt install ~/gt --git
cd ~/gt

# 2. Add your project as a Rig
gt rig add myproject https://github.com/you/repo.git

# 3. (Optional) Create a personal Crew workspace
gt crew add yourname --rig myproject

# 4. Start Gastown services
gt start

# 5. Access Mayor session (primary interface)
gt mayor attach
```

#### Per-Project Setup (in your project directory)

```bash
# Run Conductor setup
/conductor-setup

# Initialize Beads if not done
bd init
```

### Quick Verification
```bash
# Required (all must pass):
which go && which bd && which gt && ls conductor/product.md

# Recommended:
which tmux && which claude
```

## Overview

Gastown provides multi-agent coordination for Claude Code. By integrating with Gastown, Conductor-Beads gains:

- **Polecat workers**: Ephemeral agents for parallel task execution
- **Hook durability**: Work survives session crashes/compaction
- **Refinery**: Automated merge queue for parallel branches
- **Convoy tracking**: Batch work progress monitoring

## Architecture

### Gastown Town Structure

```
Town (~/gt/)                     # Gastown workspace (one per machine)
├── Mayor (hq-mayor)             # AI coordinator - primary interface
├── Deacon (hq-deacon)           # Daemon process, agent lifecycle
└── Rigs/                        # Project containers
    └── myproject/
        ├── Witness              # Monitor polecats, nudge stuck workers
        ├── Refinery             # Merge queue, PR review
        ├── Polecats             # Ephemeral workers (spawn → work → disappear)
        └── Crews/               # Long-running workspaces
            └── yourname/        # Personal dev environment
```

### Gastown Roles

| Role | Scope | Job |
|------|-------|-----|
| **Overseer** | Human | Sets strategy, reviews output, handles escalations |
| **Mayor** | Town-wide | Cross-rig coordination, work dispatch (primary interface) |
| **Deacon** | Town-wide | Daemon process, agent lifecycle, plugin execution |
| **Witness** | Per-rig | Monitor polecats, nudge stuck workers |
| **Refinery** | Per-rig | Merge queue, PR review, integration |
| **Polecat** | Per-task | Ephemeral worker: execute work, file issues, request shutdown |
| **Crew** | Per-rig | Long-running workspace for ongoing development |

### Integration Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Request                            │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Conductor-Beads                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │   Specs     │  │   Plans     │  │   Track Management      │  │
│  │ (product,   │  │ (phased,    │  │   (status, resume,      │  │
│  │  tech-stack)│  │  parallel)  │  │    archive)             │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────────────┐
│  Beads Layer  │    │   Gastown     │    │   Native Task()       │
│  (shared)     │    │   Dispatch    │    │   (fallback)          │
│               │    │               │    │                       │
│  - bd create  │    │  - gt sling   │    │  - Task() sub-agents  │
│  - bd ready   │    │  - gt convoy  │    │  - parallel_state.json│
│  - bd done    │    │  - gt mail    │    │                       │
└───────────────┘    └───────────────┘    └───────────────────────┘
        │                     │
        │                     ▼
        │            ┌───────────────────────────────────────────┐
        │            │              Gastown Town                 │
        │            │  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
        │            │  │ Polecat │  │ Polecat │  │ Polecat │   │
        │            │  │ Worker 1│  │ Worker 2│  │ Worker N│   │
        │            │  └────┬────┘  └────┬────┘  └────┬────┘   │
        │            │       │            │            │        │
        │            │       ▼            ▼            ▼        │
        │            │  ┌─────────────────────────────────────┐ │
        │            │  │           Refinery                  │ │
        │            │  │      (merge queue processor)        │ │
        │            │  └─────────────────────────────────────┘ │
        └────────────┼──────────────────────────────────────────┘
                     │
                     ▼
              ┌─────────────┐
              │    Git      │
              │  Worktrees  │
              └─────────────┘
```

## Execution Methods

Conductor offers two ways to execute parallel tracks:

| Method | Command | Requirements | Best For |
|--------|---------|--------------|----------|
| **Native** | `/conductor-implement` | None | Most projects, simple parallel |
| **Gastown** | `/conductor-dispatch` | `gt` CLI installed | Large tracks, crash recovery |

Both methods:
- Follow the same TDD workflow
- Use the same spec/plan files
- Produce identical code quality

Gastown adds operational benefits (durability, auto-merge, dashboard) for complex/long-running tracks.

## Integration Modes

### Mode 1: Gastown-Native (Recommended)

When Gastown is installed (`gt` CLI available), use Gastown for parallel execution:

```bash
# Detection
if which gt > /dev/null 2>&1; then
  GASTOWN_ENABLED=true
fi
```

**Benefits:**
- Better resource management (Polecats are purpose-built workers)
- Automated merge handling via Refinery
- Hook durability for crash recovery
- Convoy tracking for batch progress

### Mode 2: Native Task() (Fallback)

When Gastown is not available, use Conductor's built-in Task() sub-agents:

- Uses `parallel_state.json` for coordination
- Manual merge conflict resolution
- No hook durability (relies on implement_state.json)

## Unified Beads Layer

Both systems share the same Beads instance for work tracking:

```
.beads/
├── issues/           # Shared issue storage
│   ├── gt-*.json     # Gastown-created issues (polecats, convoys)
│   ├── cd-*.json     # Conductor-created issues (tracks, tasks)
│   └── hq-*.json     # Gastown town-level issues
├── formulas/         # Gastown workflow templates
│   └── conductor-track.formula.toml  # NEW: Conductor integration
└── config.json
```

### Issue Prefix Convention

| Prefix | Owner | Purpose |
|--------|-------|---------|
| `cd-` | Conductor | Tracks and tasks |
| `gt-` | Gastown | Polecat work items |
| `hq-` | Gastown | Town-level coordination |

### Beads Fields Mapping

```json
{
  "id": "cd-auth-20241226",
  "type": "epic",
  "title": "Authentication Feature",
  "metadata": {
    "conductor_track": "auth_20241226",
    "gastown_convoy": "convoy-abc123",
    "spec_path": "conductor/tracks/auth_20241226/spec.md",
    "plan_path": "conductor/tracks/auth_20241226/plan.md"
  }
}
```

## CLI Command Reference

### Beads (`bd`) - Data Layer

| Command | Description |
|---------|-------------|
| `bd init [--prefix PREFIX]` | Initialize beads in git repo |
| `bd create "Title" [-p PRIORITY] [--type TYPE] [--parent ID]` | Create issue |
| `bd ready` | List tasks with no open blockers |
| `bd list [--status STATUS] [--parent ID]` | List issues |
| `bd show <id>` | View issue details |
| `bd update <id> [--status STATUS] [--notes "..."]` | Update issue |
| `bd close <id> [--reason "..."]` | Close issue |
| `bd dep add <child> <parent>` | Add dependency |
| `bd sync` | Force immediate git sync |
| `bd prime` | AI-optimized workflow context |

### Gastown (`gt`) - Orchestration Layer

| Command | Description |
|---------|-------------|
| `gt install [path] [--git]` | Create town workspace |
| `gt rig add <name> <url>` | Add project as rig |
| `gt start` | Start Gastown services |
| `gt convoy create "name" [issues...] [--notify] [--human]` | Create convoy |
| `gt convoy list [--all]` | Dashboard of convoys |
| `gt convoy status [id]` | Detailed convoy progress |
| `gt sling <bead> <rig>` | Assign work to polecat |
| `gt hook` | Check what's on my hook |
| `gt handoff` | Request session cycle |
| `gt done [--exit]` | Signal work completion |
| `gt mail inbox` | Check messages |
| `gt dashboard [--port PORT]` | Start web dashboard |
| `gt agents` | Navigate between sessions |
| `gt mayor attach` | Enter Mayor session |

## Gastown Dispatch Protocol

### Phase 1: Track Creation → Convoy

When `/conductor-newtrack` creates a parallel-enabled track:

```bash
# 1. Create Beads epic for the track
EPIC_ID=$(bd create "Feature: Auth System" -p 1 --json | jq -r '.id')

# 2. Create Gastown convoy for tracking
gt convoy create "Auth System" $EPIC_ID --notify --human

# 3. Store in track metadata
# conductor/tracks/auth_20241226/metadata.json
{
  "beads_epic": "cd-abc123",
  "execution_mode": "gastown"
}
```

### Phase 2: Parallel Task Dispatch → Polecats

When `/conductor-dispatch` encounters a parallel phase:

```bash
# For each parallel task:

# 1. Create Beads task under epic
TASK_ID=$(bd create "Implement auth module" \
  --type task \
  --parent $EPIC_ID \
  -p 1 \
  --json | jq -r '.id')

# 2. Sling to Gastown polecat (rig name required)
gt sling $TASK_ID myproject

# Polecat automatically:
# - Gets isolated git worktree
# - Reads work from hook (gt hook)
# - Executes TDD workflow
# - Uses bd to track progress
# - Signals completion with gt done
```

### Phase 3: Completion → Refinery

Gastown's Refinery automatically handles merging polecat branches:

```bash
# Polecats automatically push to polecat/* branches
# Witness monitors polecat completion and notifies Refinery
# Refinery processes merge queue automatically

# Check refinery status
gt refinery status

# Manual merge if needed
gt refinery merge polecat-$TASK_ID
```

### Phase 4: Progress Tracking

```bash
# Query convoy status
gt convoy list

# Detailed convoy view
gt convoy status convoy-abc123

# Monitor in real-time (dashboard)
gt dashboard --port 8080
```

## Hooks and Durability

Gastown's hook system provides crash recovery. Each agent has a hook where work is persisted:

```bash
# Agents check their hook on startup
gt hook

# The "Propulsion Principle": If your hook has work, RUN IT
# Work state survives:
# - Session crashes
# - Context compaction  
# - Agent restarts

# Work state is tracked in Beads, not agent memory
```

## Polecat Work Session (Inside the Polecat)

When a polecat is spawned via `gt sling`, here's what happens inside:

```bash
# 1. Check hook - what am I working on?
gt hook

# 2. Find ready work from beads
bd ready

# 3. Start work on a task
bd update <task-id> --status in_progress

# 4. Execute TDD workflow
#    - Write failing tests
#    - Implement to pass
#    - Refactor
#    - Commit changes

# 5. Add notes (survives compaction!)
bd update <task-id> --notes "Implemented JWT auth with RS256. Key files: auth.ts, auth.test.ts"

# 6. Complete the task
bd close <task-id> --reason "commit: abc123"

# 7. Session cycling (if context getting full)
gt handoff                    # Session cycles, sandbox persists

# 8. When all work done, signal completion
gt done --exit                # Exit session, Witness handles cleanup
```

### Polecat Lifecycle Layers

| Layer | Component | Lifecycle | Persistence |
|-------|-----------|-----------|-------------|
| **Session** | Claude (tmux) | Ephemeral | Cycles per step/handoff |
| **Sandbox** | Git worktree | Persistent | Until nuke |
| **Slot** | Name from pool | Persistent | Until nuke |

**Key insight:** Session cycling is normal operation. The polecat's sandbox (worktree) and work state (in Beads) persist across all session cycles.

## Conductor-Track Formula

Create a Gastown formula for Conductor workflows:

```toml
# .beads/formulas/conductor-track.formula.toml
formula = "conductor-track"
description = "Execute a Conductor track via Gastown"
version = "1.0.0"

[metadata]
author = "Conductor-Beads"
category = "development"

[[steps]]
id = "load-context"
description = "Load track spec and plan"
instructions = """
Read and understand:
- conductor/tracks/{{track_id}}/spec.md
- conductor/tracks/{{track_id}}/plan.md
- conductor/workflow.md
"""

[[steps]]
id = "execute-phases"
description = "Execute each phase"
needs = ["load-context"]
parallel = true
instructions = """
For each phase in plan.md:
1. Check execution mode (parallel/sequential)
2. If parallel: spawn polecats via gt sling
3. If sequential: execute tasks in order
4. Mark phase complete when all tasks done
"""

[[steps]]
id = "sync-status"
description = "Sync completion to Conductor"
needs = ["execute-phases"]
instructions = """
1. Update conductor/tracks.md status to [x]
2. Delete implement_state.json
3. Run bd sync
"""
```

## Command Extensions

### `/conductor-dispatch` (New Command)

Dispatch track to Gastown for execution:

```markdown
# Usage
/conductor-dispatch [track_id]

# Behavior
1. Verify Gastown is available (gt CLI)
2. Verify project is added as a Rig in Gastown Town
3. Create convoy for track
4. Sling parallel tasks to polecats
5. Monitor via convoy list/dashboard
```

### `/conductor-status` Enhancement

Add Gastown convoy status when available:

```markdown
## Track: auth_20241226 [~]
- Phase 1: ████████░░ 80% (4/5 tasks)
- Phase 2: ░░░░░░░░░░ 0% (waiting)

### Gastown Status
- Convoy: convoy-abc123
- Polecats: 3 active, 2 completed
- Refinery: 2 PRs queued
```

## Configuration

### conductor/beads.json (Updated)

```json
{
  "enabled": true,
  "prefix": "cd",
  "gastown": {
    "enabled": true,
    "auto_dispatch": false,
    "convoy_notify": true,
    "use_refinery": true
  }
}
```

### Detection Flow

```bash
# Check both systems
BEADS_ENABLED=false
GASTOWN_ENABLED=false

if which bd > /dev/null 2>&1; then
  BEADS_ENABLED=true
fi

if which gt > /dev/null 2>&1; then
  GASTOWN_ENABLED=true
fi

# Execution mode decision
if [ "$GASTOWN_ENABLED" = "true" ]; then
  PARALLEL_MODE="gastown"
elif [ "$BEADS_ENABLED" = "true" ]; then
  PARALLEL_MODE="native-beads"
else
  PARALLEL_MODE="native-task"
fi
```

## Migration Path

### From Native Task() to Gastown

1. Install Gastown: `go install github.com/steveyegge/gastown/cmd/gt@latest`
2. Create Town: `gt install ~/gt --git`
3. Add project as Rig: `gt rig add myproject https://github.com/you/repo.git`
4. Update beads.json: `"gastown": {"enabled": true}`
5. Run `/conductor-dispatch` to use Gastown orchestration

### Preserving In-Progress Work

```bash
# Export current parallel_state.json to Gastown
gt convoy create "Migrated Track" cd-existing-track

# For each in-progress worker:
gt sling cd-task-N $(pwd) --resume
```

## Error Handling

### Gastown Unavailable Mid-Execution

```markdown
⚠️ Gastown command failed: `gt sling` returned error

Options:
A) Retry with Gastown
B) Fall back to native Task() execution
C) Stop and investigate

Note: In-progress polecats will continue independently.
Query status with: gt convoy show <convoy_id>
```

### Merge Conflicts (Refinery)

```markdown
⚠️ Refinery reported merge conflict

Conflicting files:
- src/auth/index.ts (polecat-1 vs polecat-2)

Options:
A) Open merge tool manually
B) Prefer polecat-1's version
C) Prefer polecat-2's version
D) Stop convoy and fix manually
```

## Benefits Summary

| Feature | Native Task() | + Gastown |
|---------|---------------|-----------|
| Parallel execution | ✅ | ✅ |
| Crash recovery | ⚠️ implement_state.json | ✅ Hook durability |
| Merge handling | ❌ Manual | ✅ Refinery auto-merge |
| Progress tracking | ⚠️ parallel_state.json | ✅ Convoy dashboard |
| Resource management | ⚠️ Sub-agents compete | ✅ Polecat isolation |
| Cross-session memory | ✅ Beads | ✅ Beads + Mail protocol |

## Future Enhancements

1. **Mayor escalation**: Route blocked tasks to Mayor for cross-rig coordination
2. **Crew integration**: Long-running tracks use Crew instead of Polecats
3. **Dashboard embedding**: Show Conductor status in Gastown web dashboard
4. **Formula library**: Pre-built formulas for common Conductor patterns
