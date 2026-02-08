# Learnings: Copilot CLI Installation Fix

This file captures patterns, gotchas, and context discovered during implementation.

---

## [2026-02-08] Track Created

**Issue Identified:** Copilot plugin marketplace option (`copilot plugin install conductor-beads@conductor-beads`) doesn't work - plugin not in marketplace.

**Solution Approach:** Provide two working installation methods:
1. Local installation (clone + copy, like Claude Code section)
2. GitHub URL installation (copilot plugin install with URL)

**Patterns to Maintain:**
- Match Claude Code section detail level
- Keep installation comparison table
- Ensure all 3 platforms have clear, working instructions

---
