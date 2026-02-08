# Spec: Copilot CLI Plugin Implementation

## Overview

Implement Copilot CLI plugin support for Conductor-Beads while maintaining 100% backward compatibility with existing Claude Code and Gemini CLI integrations.

## Objectives

1. **Create Plugin Structure**
   - Add `.github/plugin/plugin.json` manifest
   - Create `plugins/conductor-beads/` directory
   - Implement symlink-based structure (fallback to copy if needed)

2. **Maintain Backward Compatibility**
   - Verify existing `.claude/` commands work unchanged in Claude Code
   - Verify existing `commands/conductor/` TOML files work unchanged in Gemini CLI
   - No breaking changes to existing user workflows

3. **Document New Installation Method**
   - Update README.md with Copilot CLI installation instructions
   - Create plugin-specific README in `plugins/conductor-beads/`
   - Document namespace changes (`/conductor-beads:command`)

4. **Test Multi-Platform Support**
   - Verify installation works via `copilot plugin install`
   - Test that symlinks work correctly in Copilot CLI
   - Document any platform-specific issues (Windows, etc.)

## Success Criteria

- [ ] Plugin manifest (`plugin.json`) created with correct metadata
- [ ] Plugin directory structure created with symlinks to `.claude/`
- [ ] Plugin installs successfully via `copilot plugin install conductor-beads@conductor-beads`
- [ ] All commands accessible with `/conductor-beads:` namespace
- [ ] Existing Claude Code installation method still works
- [ ] Existing Gemini CLI installation method still works
- [ ] Documentation updated for all three platforms
- [ ] No files duplicated (symlinks working)

## Constraints

- **CRITICAL: No breaking changes** to existing `.claude/` or `commands/conductor/` structures
- Must support all three platforms: Copilot CLI, Claude Code, Gemini CLI
- Prefer symlinks over file duplication
- Plugin must be installable from GitHub repository directly
- Namespace prefix (`/conductor-beads:`) is acceptable trade-off

## Technical Requirements

### 1. Manifest File

Location: `.github/plugin/plugin.json`

```json
{
  "name": "conductor-beads",
  "description": "Context-Driven Development framework with persistent memory. Combines structured planning (Conductor) with cross-session task tracking (Beads).",
  "version": "0.1.0",
  "author": {
    "name": "Conductor-Beads Community"
  },
  "repository": "https://github.com/NguyenSiTrung/Conductor-Beads",
  "license": "Apache-2.0"
}
```

### 2. Plugin Directory Structure

```
plugins/conductor-beads/
├── .github/
│   └── plugin/
│       └── plugin.json      # Symlink to ../../.github/plugin/plugin.json
├── README.md                 # New: Plugin-specific documentation
├── commands/                 # Symlink to ../../.claude/commands
└── skills/                   # Symlink to ../../.claude/skills
```

### 3. Symlink Creation Commands

```bash
# From repo root
mkdir -p plugins/conductor-beads/.github/plugin
mkdir -p .github/plugin

# Create manifest
# (content created in .github/plugin/plugin.json)

# Create symlinks
cd plugins/conductor-beads
ln -s ../../.github/plugin .github/plugin
ln -s ../../.claude/commands commands
ln -s ../../.claude/skills skills
```

### 4. Plugin README Content

High-level overview pointing to main README:
- What is Conductor-Beads
- Quick installation: `copilot plugin install conductor-beads@conductor-beads`
- Link to full documentation in main README
- Note about namespace prefix
- Mention other installation methods

## Testing Strategy

1. **Symlink Test:**
   - Create symlinks locally
   - Verify Git preserves symlinks
   - Test on macOS/Linux
   - Test on Windows (may need special handling)

2. **Installation Test (Manual):**
   - Cannot test actual `copilot plugin install` without Copilot CLI access
   - Validate manifest JSON schema
   - Verify directory structure matches awesome-copilot pattern

3. **Backward Compatibility Test:**
   - Test existing Claude Code installation still works
   - Test existing Gemini CLI installation still works
   - Verify no changes to user workflows

## Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Symlinks don't work in Copilot CLI | High | Add build script to copy files as fallback |
| Windows symlink issues | Medium | Document dev mode requirement, provide copy alternative |
| Namespace prefix confuses users | Low | Clear documentation, update all examples |
| Plugin discovery fails | High | Verify manifest location and format match spec |

## Deliverables

1. `.github/plugin/plugin.json` - Plugin manifest
2. `plugins/conductor-beads/` - Plugin directory with symlinks
3. `plugins/conductor-beads/README.md` - Plugin documentation
4. Updated main `README.md` - Add Copilot CLI installation section
5. Testing notes - Document what was tested and results
6. Known issues doc - Any platform-specific problems discovered

## Non-Goals

- Creating new commands (use existing 17 commands)
- Modifying command behavior
- Changing Gemini CLI or Claude Code integration
- Supporting other plugin systems beyond Copilot CLI
