# Spec: Copilot CLI Plugin System Analysis

## Overview

Analyze how to make Conductor-Beads installable as a Copilot CLI plugin via `/plugin install` while maintaining compatibility with existing Gemini CLI and Claude Code integrations.

## Objectives

1. **Understand Copilot CLI Plugin System**
   - Study the plugin architecture from: https://deepwiki.com/github/copilot-cli/5.4-plugin-system
   - Examine the awesome-copilot example: https://github.com/github/awesome-copilot
   - Identify required files, structure, and metadata

2. **Assess Compatibility Requirements**
   - Determine if changes would break Gemini CLI integration
   - Determine if changes would break Claude Code integration
   - Identify shared vs. platform-specific components

3. **Design Minimal Plugin Structure**
   - Propose file structure for Copilot CLI compatibility
   - Map existing commands/skills to plugin format
   - Identify any required metadata or manifest files

## Success Criteria

- [ ] Clear understanding of Copilot CLI plugin requirements documented
- [ ] Compatibility analysis complete (what stays, what changes)
- [ ] Minimal implementation plan created for track 2
- [ ] No breaking changes to existing Gemini CLI / Claude Code functionality

## Constraints

- Must maintain backward compatibility with:
  - Gemini CLI extension (TOML commands)
  - Claude Code (Markdown commands + skills)
- Changes should be additive, not destructive
- Prefer configuration over code changes

## Research Tasks

1. Study Copilot CLI plugin documentation
2. Analyze awesome-copilot repository structure
3. Compare plugin format to existing `.claude/` structure
4. Identify overlap with current commands/skills
5. Document findings in track's `learnings.md`

## Deliverables

- `learnings.md` - Research findings and patterns discovered
- `plan.md` - Detailed implementation plan for track 2
- Updated `spec.md` for implementation track (track 2)
