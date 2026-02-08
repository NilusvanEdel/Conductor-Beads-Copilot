# Specification: Fix Copilot CLI Installation Instructions

## Overview

Update README.md to provide correct Copilot CLI installation methods since the plugin marketplace option is unavailable. Add two practical installation paths: local clone/copy and GitHub URL installation, matching the detail level of Claude Code section.

## Problem

Current README.md (line 60) references:
```bash
copilot plugin install conductor-beads@conductor-beads
```

This fails with: "failed to install plugin: marketplace 'conductor-beads' not found"

The plugin is not published to any official marketplace. Users need alternative, working installation methods.

## Functional Requirements

- [ ] Add **Local Installation** method for Copilot CLI (similar to Claude Code section)
  - Clone repository locally
  - Copy plugin directory to Copilot config location
  - Clear instructions with step-by-step commands
  
- [ ] Add **GitHub URL Installation** method for Copilot CLI
  - Use `copilot plugin install` with GitHub repository URL
  - Include syntax: `copilot plugin install <github-url>`
  - Clarify any authentication requirements
  
- [ ] Maintain **Installation Comparison Table** showing all 3 platforms
  - Update Copilot CLI row with accurate installation method
  - Keep side-by-side comparison format
  
- [ ] Match **Claude Code Section Detail Level**
  - Similar formatting and information depth
  - Include both full and minimal installation options
  - Show project-local installation if applicable

## Non-Functional Requirements

- [ ] Documentation should be clear for new users
- [ ] Keep README organized and easy to navigate
- [ ] Maintain parity with other platform sections

## Acceptance Criteria

- README.md Copilot CLI section provides two working installation methods
- Installation instructions are tested and verified as working
- Detail level matches Claude Code section
- Comparison table is updated and accurate
- All command examples are syntax-correct

## Out of Scope

- Publishing plugin to official marketplace (requires Copilot team approval)
- Creating automated installation scripts
- Platform-specific customizations beyond standard instructions
