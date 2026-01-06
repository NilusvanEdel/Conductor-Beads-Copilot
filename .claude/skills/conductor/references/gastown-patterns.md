# Gastown Integration Patterns

Reference for Gastown concepts, commands, and best practices when using Conductor with Gastown multi-agent orchestration.

## Core Concepts

### Town Architecture

```
Town (~/gt/)                     # Gastown workspace (one per machine)
├── Mayor (hq-mayor)             # AI coordinator - primary interface
├── Deacon (hq-deacon)           # Daemon process, agent lifecycle
└── Rigs/                        # Project containers
    └── myproject/
        ├── Witness              # Monitor polecats, nudge stuck workers
        ├── Refinery             # Merge queue, PR review
        ├── Polecats/            # Ephemeral workers (spawn → work → disappear)
        └── Crews/               # Long-running workspaces
            └── yourname/        # Personal dev environment
```

### Roles

| Role | Scope | Job |
|------|-------|-----|
| **Overseer** | Human | Sets strategy, reviews output, handles escalations |
| **Mayor** | Town-wide | Cross-rig coordination, work dispatch (primary interface) |
| **Deacon** | Town-wide | Daemon process, agent lifecycle, plugin execution |
| **Witness** | Per-rig | Monitor polecats, nudge stuck workers |
| **Refinery** | Per-rig | Merge queue, PR review, integration |
| **Polecat** | Per-task | Ephemeral worker: execute work, file issues, request shutdown |
| **Crew** | Per-rig | Long-running workspace for ongoing development |

## Polecat Lifecycle

### Three Lifecycle Layers

| Layer | Component | Lifecycle | Persistence |
|-------|-----------|-----------|-------------|
| **Session** | Claude (tmux) | Ephemeral | Cycles per step/handoff |
| **Sandbox** | Git worktree | Persistent | Until nuke |
| **Slot** | Name from pool | Persistent | Until nuke |

**Key insight:** Session cycling is normal operation. The polecat's sandbox (worktree) and work state (in Beads) persist across all session cycles.

### Polecat Work Protocol

```bash
# 1. Get AI-optimized context (CRITICAL: run first)
bd prime

# 2. Check hook - what am I working on?
gt hook

# 3. Find ready work from beads
bd ready

# 4. Start work with context notes
bd update <task_id> --status in_progress \
  --notes "Started: <task_description>
APPROACH: <planned_approach>"

# 5. Execute TDD workflow
#    - RED: Write failing tests
#    - GREEN: Implement to pass
#    - REFACTOR: Clean up
#    - Verify coverage

# 6. Add progress notes (CRITICAL for compaction survival)
bd update <task_id> --notes "COMPLETED: <what_was_done>
KEY DECISION: <important_choice_with_rationale>
FILES CHANGED: <list_of_files>
COMMIT: <sha>"

# 7. Complete with auto-advance
bd close <task_id> --continue --reason "commit: <sha>"

# 8. Session cycling if context filling
gt handoff  # Session cycles, sandbox persists

# 9. When all work done
gt done --exit  # Exit session, Witness handles cleanup
```

## The Propulsion Principle

> **If your hook has work, RUN IT.**

Agents check their hook on wake and execute molecule steps. No waiting for commands. Molecules survive crashes - any agent can continue where another left off.

## Convoy vs Swarm

| Concept | Persistent? | ID | Description |
|---------|-------------|-----|-------------|
| **Convoy** | Yes | hq-cv-* | Tracking unit. What you create, track, get notified about. |
| **Swarm** | No | None | Ephemeral. "The workers currently on this convoy's issues." |

When you "kick off a swarm", you're really:
1. Creating a convoy (the tracking unit)
2. Assigning polecats to the tracked issues
3. The "swarm" is just those polecats while they're working

When issues close, the convoy lands and notifies you. The swarm dissolves.

### Convoy Lifecycle

```
OPEN ──(all issues close)──► LANDED/CLOSED
  ↑                              │
  └──(add more issues)───────────┘
       (auto-reopens)
```

## Molecule Chemistry

### Phase States

| Phase | Name | Storage | Behavior |
|-------|------|---------|----------|
| Ice-9 | Formula | `.beads/formulas/` | Source template, composable |
| Solid | Protomolecule | `.beads/` | Frozen template, reusable |
| Liquid | Mol | `.beads/` | Flowing work, persistent |
| Vapor | Wisp | `.beads/` (ephemeral) | Transient, for patrols |

### Operators

| Operator | From → To | Effect |
|----------|-----------|--------|
| `cook` | Formula → Protomolecule | Expand macros, flatten |
| `pour` | Proto → Mol | Instantiate as persistent |
| `wisp` | Proto → Wisp | Instantiate as ephemeral |
| `squash` | Mol/Wisp → Digest | Condense to permanent record |
| `burn` | Wisp → ∅ | Discard without record |

## Escalation

```bash
gt escalate -s CRITICAL "msg"  # Immediate attention required
gt escalate -s HIGH "msg"      # Important blocker
gt escalate -s MEDIUM "msg"    # Default severity
```

## Session Discovery (Seance)

Access previous sessions for context recovery:

```bash
gt seance                      # List discoverable predecessor sessions
gt seance --talk <id>          # Talk to predecessor (full context)
gt seance --talk <id> -p "?"   # One-shot question
```

## Communication

### Mail Protocol

```bash
gt mail inbox                  # Check messages
gt mail read <id>              # Read specific message
gt mail send <addr> -s "Subject" -m "Body"
gt mail send --human -s "..."  # To overseer
```

### Agent Nudging

**IMPORTANT**: Always use `gt nudge` to send messages to Claude sessions. Never use raw `tmux send-keys`.

```bash
gt nudge <agent> "message"     # Send message to agent session
gt peek <agent>                # Check agent health
```

## Beads Routing

Gastown routes beads commands based on issue ID prefix:

```bash
bd show gp-xyz    # Routes to greenplace rig's beads
bd show hq-abc    # Routes to town-level beads
bd show wyv-123   # Routes to wyvern rig's beads
```

| Prefix | Routes To | Purpose |
|--------|-----------|---------|
| `hq-*` | `~/gt/.beads/` | Mayor mail, cross-rig coordination |
| `gp-*` | `~/gt/greenplace/mayor/rig/.beads/` | Greenplace project issues |
| `cd-*` | Conductor prefix | Conductor tracks and tasks |

## Notes Format for Compaction Survival

Write notes as if explaining to a future Claude with zero context:

```bash
bd update <task_id> --notes "COMPLETED: JWT auth with RS256
KEY DECISION: RS256 over HS256 for key rotation
IN PROGRESS: Password reset flow
NEXT: Implement rate limiting
BLOCKER: None
DISCOVERED: Found race condition in token refresh (created bd-xyz)"
```

**Fields to include:**
- `COMPLETED:` - What was finished (past tense, specific)
- `KEY DECISION:` - Important choices made (with rationale)
- `IN PROGRESS:` - Current work
- `NEXT:` - Immediate next step (concrete action)
- `BLOCKER:` - What's blocking (if any)
- `DISCOVERED:` - New issues found (with beads ID if created)

## Quick Reference

### Starting Gastown

```bash
gt install ~/gt --git          # Create town (one-time)
gt rig add myproject <url>     # Add project
gt start                       # Start services
gt mayor attach                # Enter Mayor session
```

### Dispatching Work

```bash
gt convoy create "Feature X" gt-abc --notify --human
gt sling gt-abc myproject      # Spawns polecat automatically
gt convoy list                 # Check progress
gt dashboard --port 8080       # Web UI
```

### Monitoring

```bash
gt convoy list                 # Active convoys
gt convoy status <id>          # Detailed view
gt agents                      # Navigate sessions
gt refinery status             # Merge queue
gt doctor [--fix]              # Health check
```
