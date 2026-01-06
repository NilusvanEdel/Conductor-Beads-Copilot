---
description: Dispatch a track to Gastown for multi-agent execution
argument-hint: [track_id]
---

<!-- 
SYSTEM DIRECTIVE: You are an AI agent for the Conductor framework with Gastown integration.
CRITICAL: Validate every tool call. If any fails, halt and announce the failure.
-->

# Conductor Dispatch

Dispatch track to Gastown: $ARGUMENTS

---

## 1.0 PREREQUISITES CHECK

**PROTOCOL: Verify Gastown and Beads are available.**

1. **Check Gastown CLI:**
   ```bash
   which gt
   ```
   - If NOT found:
     > "❌ Gastown is not installed. Install with: `go install github.com/steveyegge/gastown/cmd/gt@latest`"
     > "Alternatively, use `/conductor-implement` for native parallel execution."
     → HALT

2. **Check Beads CLI:**
   ```bash
   which bd
   ```
   - If NOT found:
     > "❌ Beads is not installed. Gastown dispatch requires Beads for work tracking."
     → HALT

3. **Check Gastown Town exists:**
   ```bash
   gt status
   ```
   - If fails:
     > "❌ Gastown Town not initialized. Run:"
     > "  `gt install ~/gt --git`"
     > "  `cd ~/gt && gt rig add <name> <repo-url>`"
     → HALT

4. **Check Conductor Setup:**
   - Verify `conductor/product.md`, `conductor/tech-stack.md`, `conductor/workflow.md` exist
   - If missing → HALT with setup instructions

---

## 2.0 TRACK SELECTION

**PROTOCOL: Identify the track to dispatch.**

1. **If track_id provided:**
   - Verify `conductor/tracks/<track_id>/` exists
   - Read `conductor/tracks/<track_id>/metadata.json`
   - Confirm with user: "Dispatching track '<description>' to Gastown. Proceed?"

2. **If no track_id:**
   - Parse `conductor/tracks.md` for first `[ ]` or `[~]` track
   - Suggest: "Found track '<description>'. Dispatch this? (y/n)"

3. **Check for parallel phases:**
   - Read `conductor/tracks/<track_id>/plan.md`
   - Scan for `<!-- execution: parallel -->` annotations
   - If NO parallel phases found:
     > "⚠️ This track has no parallel phases. Gastown dispatch is most beneficial for parallel work."
     > "A) Dispatch anyway (uses Gastown for durability)"
     > "B) Use `/conductor-implement` instead"

---

## 3.0 CREATE CONVOY

**PROTOCOL: Set up Gastown convoy for track coordination.**

1. **Get or create Beads epic:**
   ```bash
   # Check if epic exists in metadata
   EPIC_ID=$(cat conductor/tracks/<track_id>/metadata.json | jq -r '.beads_epic // empty')
   
   if [ -z "$EPIC_ID" ]; then
     # Create new epic (bd create returns JSON with id)
     EPIC_ID=$(bd create "<track_description>" -p 1 --json | jq -r '.id')
     # Update metadata.json with epic ID
   fi
   ```

2. **Create Gastown convoy:**
   ```bash
   # Create convoy to track the work (--human notifies overseer)
   gt convoy create "<track_description>" $EPIC_ID --notify --human
   ```

3. **Update track metadata:**
   ```json
   {
     "beads_epic": "<epic_id>",
     "execution_mode": "gastown",
     "dispatched_at": "<timestamp>"
   }
   ```

---

## 4.0 DISPATCH PARALLEL TASKS

**PROTOCOL: Sling parallel tasks to Gastown polecats.**

1. **Parse plan.md for parallel phases:**
   - For each phase with `<!-- execution: parallel -->`:
     - Extract tasks with their file ownership (`<!-- files: ... -->`)
     - Extract dependencies (`<!-- depends: ... -->`)

2. **Create Beads tasks and dispatch:**
   
   For each task in parallel phase:
   
   ```bash
   # Create Beads task under epic
   TASK_ID=$(bd create "<task_description>" \
     --type task \
     --parent $EPIC_ID \
     -p 1 \
     --json | jq -r '.id')
   
   # Set dependencies if needed
   # bd dep add <child> <parent>  means child depends on parent
   
   # Sling to polecat (rig name is the project name in Gastown)
   gt sling $TASK_ID <rig_name>
   
   # Polecat will:
   # 1. gt hook - check what work is assigned
   # 2. bd ready - find unblocked tasks
   # 3. bd update <id> --status in_progress - start work
   # 4. Execute TDD workflow
   # 5. bd update <id> --notes "..." - add context for compaction survival
   # 6. bd close <id> --reason "commit: <sha>" - complete task
   # 7. gt done --exit - signal completion
   ```

3. **Handle dependencies:**
   - Tasks with `<!-- depends: taskN -->`:
     ```bash
     # child depends on parent (parent must complete first)
     bd dep add $CHILD_TASK_ID $PARENT_TASK_ID
     ```
   - Dependent tasks won't appear in `bd ready` until blockers are closed

4. **Update track status:**
   - Change `## [ ] Track:` to `## [~] Track:` in `conductor/tracks.md`
   - Create dispatch state file:
     ```json
     // conductor/tracks/<track_id>/dispatch_state.json
     {
       "dispatched_tasks": [
         {"task_id": "cd-xxx", "status": "dispatched"}
       ],
       "sequential_phases": ["phase3", "phase4"],
       "dispatched_at": "<timestamp>"
     }
     ```

---

## 5.0 SEQUENTIAL PHASE HANDLING

**PROTOCOL: Queue sequential phases for later execution.**

1. **Identify sequential phases:**
   - Phases without `<!-- execution: parallel -->` annotation
   - Phases with dependencies on parallel phases

2. **Create placeholder tasks:**
   ```bash
   for phase in sequential_phases:
     PHASE_TASK=$(bd create "Execute $phase (sequential)" \
       --type task \
       --parent $EPIC_ID \
       --blocked "Waiting for parallel phases" \
       --json | jq -r '.id')
     
     gt convoy add <convoy_id> $PHASE_TASK
   ```

3. **Inform user:**
   > "📋 Sequential phases queued: <phase_list>"
   > "These will execute after parallel phases complete."

---

## 6.0 MONITORING SETUP

**PROTOCOL: Provide monitoring commands to user.**

1. **Display convoy status:**
   ```bash
   gt convoy list
   ```

2. **Provide monitoring commands:**
   > "## 🚀 Track Dispatched to Gastown"
   > 
   > **Convoy:** `<convoy_id>`
   > **Polecats:** <count> workers dispatched
   >
   > ### Monitor Progress
   > ```bash
   > # Check convoy status
   > gt convoy list
   >
   > # View dashboard
   > gt dashboard --port 8080
   >
   > # Check agent sessions
   > gt agents
   > ```
   >
   > ### When Complete
   > Run `/conductor-status <track_id>` to sync and see results.

3. **Refinery notification:**
   > "🔀 **Merge Queue:** Refinery will auto-merge polecat branches."
   > "Check queue: `gt refinery status`"

---

## 7.0 ERROR HANDLING

**PROTOCOL: Handle dispatch failures gracefully.**

1. **Partial dispatch failure:**
   - If some tasks fail to sling:
     > "⚠️ <N> tasks failed to dispatch:"
     > - <task>: <error>
     > 
     > "A) Retry failed tasks"
     > "B) Continue with dispatched tasks only"  
     > "C) Abort and clean up convoy"

2. **Convoy creation failure:**
   - Clean up any created Beads tasks
   - Offer to fall back to native execution:
     > "❌ Convoy creation failed: <error>"
     > "Fall back to `/conductor-implement`? (y/n)"

3. **Network/RPC errors:**
   - Store partial state for resume
   - Provide manual recovery commands

---

## Status Reference

After dispatch, track statuses:

| Location | Status |
|----------|--------|
| `conductor/tracks.md` | `[~]` In Progress |
| Beads epic | `in_progress` |
| Gastown convoy | `active` |
