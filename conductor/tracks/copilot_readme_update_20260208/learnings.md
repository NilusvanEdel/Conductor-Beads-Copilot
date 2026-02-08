# Learnings: Copilot README Update

This file captures patterns, gotchas, and context discovered during implementation.

---

## [2026-02-08] Track Created

**Files Changed:** None yet

**Learnings:**
- Track created for README documentation update
- Focus on dual-platform (Gemini CLI + Copilot CLI) support
- Need to balance detail in README vs context files (CLAUDE.md/GEMINI.md)

---

## [2026-02-08 12:18] - Phases 1-3: README Documentation Review & Updates
**Files Changed:** README.md, CLAUDE.md
**Commits:** 
- 7047ee3: docs(readme): fix command count and verify Copilot CLI documentation
- 13e0bf7: feat(track): complete README documentation update track

**Learnings:**
- Patterns: Documentation accuracy requires constant cross-checking against actual codebase structure (plugin manifest, symlinks, command files)
- Patterns: Multi-platform support documentation needs parity checks - Copilot CLI namespace differs but access is same
- Gotchas: Command count changed from 17 to 16 after refactoring - update count everywhere (README headers, architecture diagram, context files)
- Gotchas: Symlink symlinks are preserved by Git but need validation on each platform (Windows requires Developer Mode)
- Context: README already had good Copilot CLI coverage from previous track's implementation
- Context: Installation comparison table is effective for helping users choose the right platform
- Context: Troubleshooting section with platform-specific guidance increases adoption

---
