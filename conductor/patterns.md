# Consolidated Patterns

This file captures reusable patterns and learnings extracted from completed tracks.

## Code Conventions

*(Patterns will be added as tracks are completed)*

## Architecture

### Multi-Platform Plugin Strategy
- Use **symlink-based wrappers** to support multiple AI assistant platforms (from: copilot_analysis_20260208, 2026-02-08)
- Keep single source of truth and create platform-specific facades
- Benefit: No duplication, automatic sync, easier maintenance

### Plugin Manifest Requirements  
- Copilot CLI requires manifest at `.github/plugin/plugin.json` (from: copilot_analysis_20260208, 2026-02-08)
- Minimal fields: name, description, version, author, repository, license

## Gotchas

### Symlink Compatibility
- Git preserves symlinks, but behavior may vary across platforms (from: copilot_analysis_20260208, 2026-02-08)
- Windows requires developer mode or special permissions for symlinks
- Test symlink following in each plugin system

### Relative Path Issues
- Don't use relative paths in command files - breaks when nested in plugin directories (from: copilot_analysis_20260208, 2026-02-08)

## Testing

*(Patterns will be added as tracks are completed)*

## Context

### Backward Compatibility Principle
- New platform support should be **additive only** (from: copilot_analysis_20260208, 2026-02-08)
- Never modify existing working integrations
- Users choose their preferred installation method

### Research Track Deliverables
- Research tracks should produce: learnings.md, spec.md for next track, plan.md for next track (from: copilot_analysis_20260208, 2026-02-08)
- Separation of analysis from implementation allows review before execution

---

**Note**: This file is populated by the knowledge flywheel:
1. During implementation: Patterns captured in track's `learnings.md`
2. At phase/track completion: Elevated to this file
3. On archive: Remaining patterns extracted
4. For new tracks: Read to prime context
