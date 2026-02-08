# Product Definition: Conductor-Beads Toolkit

## Vision

A unified toolkit for **Context-Driven Development** that enables AI agents to manage long-horizon development tasks with:
- **Conductor**: Structured planning methodology (specs, plans, tracks, TDD workflows)
- **Beads**: Persistent task memory that survives conversation compaction

## Mission

Provide a cross-platform framework that turns AI assistants into proactive project managers following a strict protocol: **Context → Spec & Plan → Implement**.

## Target Users

1. **Developers** using AI coding assistants (Gemini CLI, Claude Code)
2. **AI Agents** executing complex, multi-session development tasks
3. **Teams** adopting context-driven development practices

## Core Value Propositions

1. **Cross-Platform Consistency**: Same workflow across Gemini CLI and Claude Code
2. **Persistent Memory**: Task graphs that survive context compaction via Beads
3. **Structured Methodology**: Enforce spec → plan → implement discipline
4. **Knowledge Accumulation**: Learnings system (Ralph-style) captures patterns across tracks
5. **Parallel Execution**: Scale implementation with concurrent sub-agents

## Key Features

### Implemented
- 17 Conductor commands (setup, newtrack, implement, etc.)
- Skills for auto-activation (conductor, beads, skill-creator)
- Beads integration (bidirectional sync, stealth mode)
- Parallel task execution with file-locking
- Knowledge flywheel (learnings.md → patterns.md)
- Manual handoffs for multi-session work

### In Development
- Formula system improvements (wisp, distill, pour)
- Enhanced parallel execution validation

## Success Metrics

1. **Adoption**: Number of projects using Conductor-Beads
2. **Completion Rate**: % of tracks marked complete vs abandoned
3. **Cross-Session Continuity**: % of sessions successfully resumed with Beads
4. **Command Coverage**: % of workflows accessible via natural language
5. **Integration Quality**: Beads data consistency with Conductor state

## Non-Goals

- Direct code generation without planning
- Replace manual coding (it augments, not replaces)
- Version control system (we integrate with Git, not replace it)
- Team collaboration features (focus is single-developer + AI)
