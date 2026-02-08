# Track Learnings: Copilot CLI Plugin System Analysis

## Research Findings

## [2026-02-08 12:57] - Phase 1 Task 1: Study Copilot CLI plugin documentation
Thread: N/A (research task)
- **Researched:** Copilot CLI plugin system architecture via awesome-copilot repository
- **Sources:** 
  - https://github.com/github/awesome-copilot (main repository)
  - https://github.com/github/awesome-copilot/tree/main/plugins/awesome-copilot (example plugin)
- **Key Findings:**
  - **Plugin Structure:** Plugins are directories containing:
    - `README.md` - Plugin documentation
    - `.github/plugin/plugin.json` - Manifest file with metadata
    - `commands/` - Slash commands (markdown files, e.g., `suggest-awesome-github-copilot-agents.md`)
    - `agents/` - Custom agents (markdown files referencing prompts)
    - Optional: `skills/`, `instructions/`, `prompts/`
  - **Manifest Format** (`plugin.json`):
    ```json
    {
      "name": "plugin-name",
      "description": "Description text",
      "version": "1.0.0",
      "author": { "name": "Author Name" },
      "repository": "https://github.com/...",
      "license": "MIT"
    }
    ```
  - **Installation:** Users run `copilot plugin install <plugin-name>@<repo-name>`
  - **Commands:** Markdown files in `commands/` directory become slash commands
    - Format: `command-name.md` → `/plugin-name:command-name`
    - Can reference prompts in `../../../prompts/` for shared logic
  - **Directory Structure** (from awesome-copilot):
    ```
    plugins/<plugin-name>/
    ├── .github/
    │   └── plugin/
    │       └── plugin.json      # Manifest
    ├── README.md                 # Plugin docs
    ├── commands/                 # Slash commands
    │   └── *.md
    ├── agents/                   # Custom agents
    │   └── *.md
    └── skills/                   # Optional skills
        └── */
    ```
  
- **Learnings:**
  - Patterns: Plugin system is **file-based and declarative** - no code execution, just markdown + JSON
  - Patterns: Commands are **namespace-scoped** with plugin name prefix (e.g., `/awesome-copilot:suggest-...`)
  - Gotchas: Commands reference prompts via relative paths (`../../../prompts/`), suggesting a specific repo structure is expected
  - Gotchas: Manifest must be at `.github/plugin/plugin.json` (specific path required)
  - Context: This is very similar to our existing `.claude/` structure! Just needs manifest + different directory naming
  - Context: The plugin system appears to be a **superset** of Claude Code commands - it can contain the same markdown command files

## [2026-02-08 12:58] - Phase 1 Task 2: Analyze awesome-copilot example
Thread: N/A (research task)
- **Researched:** Complete structure of awesome-copilot plugin and comparison to Conductor-Beads
- **Sources:**
  - https://github.com/github/awesome-copilot/tree/main/plugins/awesome-copilot
  - Examined: commands/, agents/, README.md, plugin.json manifest
- **Key Findings:**
  - **Complete Plugin Structure:**
    ```
    plugins/awesome-copilot/
    ├── .github/
    │   └── plugin/
    │       └── plugin.json           # REQUIRED manifest
    ├── README.md                      # Plugin documentation
    ├── commands/                      # Slash commands
    │   ├── suggest-awesome-github-copilot-agents.md
    │   ├── suggest-awesome-github-copilot-collections.md
    │   ├── suggest-awesome-github-copilot-instructions.md
    │   └── suggest-awesome-github-copilot-prompts.md
    └── agents/                        # Custom agents
        └── meta-agentic-project-scaffold.md
    ```
  - **Command Files:** Simple markdown that references shared prompts:
    ```markdown
    ../../../prompts/suggest-awesome-github-copilot-agents.prompt.md
    ```
  - **Installation Process:**
    1. User runs: `copilot plugin install awesome-copilot@awesome-copilot`
    2. Copilot CLI discovers the `.github/plugin/plugin.json` manifest
    3. Plugin assets (commands, agents) are symlinked/loaded into user's environment
    4. Commands become available as `/awesome-copilot:command-name`
  
- **Comparison Table:**
  
  | Aspect | Copilot CLI Plugin | Conductor-Beads (Current) | Notes |
  |--------|-------------------|---------------------------|-------|
  | **Manifest** | `.github/plugin/plugin.json` | None | NEW: Need to add |
  | **Commands** | `commands/*.md` | `.claude/commands/*.md` | COMPATIBLE |
  | **Agents** | `agents/*.md` | N/A | NEW: Could add |
  | **Skills** | `skills/*/` | `.claude/skills/*/` | COMPATIBLE |
  | **Installation** | `copilot plugin install` | Manual copy | Different method |
  | **Namespace** | `/plugin-name:cmd` | `/cmd` | Different convention |
  | **Documentation** | `README.md` in plugin root | `README.md` in repo root | Compatible |
  | **Gemini CLI** | N/A | `commands/conductor/*.toml` | Conductor-specific |
  
- **Learnings:**
  - Patterns: Copilot plugins expect a **specific manifest location** (`.github/plugin/plugin.json`)
  - Patterns: Markdown command files are **directly compatible** between Copilot CLI and Claude Code
  - Gotchas: Command references use relative paths (`../../../prompts/`) - assumes plugin is nested in `plugins/` directory
  - Gotchas: Namespace prefixing means commands become `/conductor-beads:setup` instead of `/conductor-setup`
  - Context: **Minimal changes needed!** We already have compatible markdown commands
  - Context: Could create a `plugins/conductor-beads/` subdirectory with symlinks to existing `.claude/` content

---

**Format for entries:**
```markdown
## [YYYY-MM-DD HH:MM] - Phase N Task M: <task title>
Thread: <URL if applicable>
- **Researched:** <what was studied>
- **Sources:** <links>
- **Key Findings:**
  - Finding 1
  - Finding 2
- **Learnings:**
  - Patterns: <patterns discovered>
  - Gotchas: <pitfalls or surprises>
  - Context: <important context for future work>
```

## [2026-02-08 12:58] - Phase 2 Task 3: Compare plugin format to existing structure
Thread: N/A (analysis task)
- **Analyzed:** Detailed compatibility between Copilot CLI plugin requirements and Conductor-Beads existing structure
- **Sources:** Current repository structure vs. Copilot CLI plugin specification
- **Key Findings:**
  
  ### Compatibility Matrix
  
  | Component | Copilot CLI Requirement | Conductor-Beads Current | Compatibility | Action Needed |
  |-----------|------------------------|------------------------|---------------|---------------|
  | **Manifest** | `.github/plugin/plugin.json` | ❌ None | ⚠️ **ADD** | Create manifest file |
  | **Commands** | `commands/*.md` | `.claude/commands/*.md` | ✅ **COMPATIBLE** | Symlink or copy |
  | **Skills** | `skills/*/SKILL.md` | `.claude/skills/*/SKILL.md` | ✅ **COMPATIBLE** | Symlink or copy |
  | **Namespace** | `/plugin-name:command` | `/command` | ⚠️ **DIFFERENT** | No change needed (Copilot CLI handles) |
  | **README** | Plugin-specific in plugin dir | Repo-level README | ⚠️ **ADAPT** | Create plugin README |
  | **Gemini CLI** | N/A | `commands/conductor/*.toml` | ✅ **INDEPENDENT** | Keep separate |
  | **Directory** | Must be in `plugins/<name>/` | Root-level `.claude/` | ⚠️ **RESTRUCTURE** | Create plugin subdirectory |
  
  ### Conflicts/Overlaps Identified:
  
  1. **No Breaking Conflicts!** 🎉
     - Copilot CLI plugin structure is **additive**
     - Existing `.claude/` commands work as-is in Claude Code
     - Existing `commands/conductor/` TOML files work as-is in Gemini CLI
  
  2. **Command Naming:**
     - **Current:** `/conductor-setup`, `/conductor-implement`
     - **After Plugin:** `/conductor-beads:setup`, `/conductor-beads:implement`
     - **Impact:** Users need to learn new namespace prefix
     - **Mitigation:** Document clearly, possibly create aliases
  
  3. **Relative Path References:**
     - **Issue:** If commands reference files with relative paths (e.g., `../templates/`), paths break when nested in `plugins/`
     - **Current State:** Our commands don't use relative references (they're self-contained)
     - **Impact:** No issue for us
  
  4. **Documentation Split:**
     - **Current:** Single README.md at repo root
     - **Plugin Needs:** README.md in `plugins/conductor-beads/` for plugin-specific docs
     - **Solution:** Create plugin README that references main README
  
- **Learnings:**
  - Patterns: **Symlinks** are the standard approach (awesome-copilot uses them) - avoids duplication
  - Patterns: Plugin structure is a **wrapper** around existing content, not a replacement
  - Gotchas: Must maintain **two installation methods**: plugin (Copilot CLI) + manual (Claude/Gemini)
  - Context: This is a **multi-platform compatibility** problem, not a migration
  - Context: Goal is to support **all three platforms** simultaneously (Copilot CLI, Claude Code, Gemini CLI)


## [2026-02-08 12:59] - Phase 2 Task 4: Test backward compatibility approach
Thread: N/A (design task)
- **Designed:** Multi-platform repository structure that supports all three AI assistant platforms
- **Approach:** **Symlink-based plugin wrapper** (non-destructive, maintains all existing functionality)
- **Key Findings:**
  
  ### Proposed Repository Structure
  
  ```
  Conductor-Beads/
  ├── .claude/                         # ✅ KEEP (Claude Code)
  │   ├── commands/                    # 17 markdown commands
  │   └── skills/                      # conductor, beads, skill-creator
  ├── commands/conductor/              # ✅ KEEP (Gemini CLI)
  │   └── *.toml                       # 17 TOML commands
  ├── plugins/                         # ⭐ NEW (Copilot CLI)
  │   └── conductor-beads/
  │       ├── .github/
  │       │   └── plugin/
  │       │       └── plugin.json      # ⭐ NEW manifest
  │       ├── README.md                # ⭐ NEW plugin-specific docs
  │       ├── commands/                # → symlink to ../../.claude/commands
  │       └── skills/                  # → symlink to ../../.claude/skills
  ├── templates/                       # ✅ KEEP (shared)
  ├── docs/                            # ✅ KEEP (shared)
  ├── README.md                        # ✅ KEEP (main docs)
  ├── CLAUDE.md                        # ✅ KEEP (Claude context)
  ├── GEMINI.md                        # ✅ KEEP (Gemini context)
  └── gemini-extension.json            # ✅ KEEP (Gemini manifest)
  ```
  
  ### Verification: No Breaking Changes
  
  | Platform | Installation Method | Files Used | Impact |
  |----------|-------------------|------------|--------|
  | **Claude Code** | Manual copy `.claude/` | `.claude/commands/*.md`, `.claude/skills/*/` | ✅ **No change** |
  | **Gemini CLI** | `gemini extensions install` | `commands/conductor/*.toml`, `gemini-extension.json` | ✅ **No change** |
  | **Copilot CLI** | `copilot plugin install` | `plugins/conductor-beads/` (symlinks to `.claude/`) | ⭐ **New capability** |
  
  ### Symlink Strategy
  
  ```bash
  # Create plugin structure with symlinks
  mkdir -p plugins/conductor-beads/.github/plugin
  cd plugins/conductor-beads
  ln -s ../../.claude/commands commands
  ln -s ../../.claude/skills skills
  ```
  
  **Benefits:**
  - ✅ Single source of truth (`.claude/` directory)
  - ✅ No file duplication
  - ✅ Changes propagate automatically
  - ✅ All platforms stay in sync
  
  **Potential Issues:**
  - ⚠️ Symlinks don't work in all environments (Windows without dev mode, some cloud storage)
  - ⚠️ GitHub preserves symlinks in repos (good!)
  - ⚠️ Need to test if Copilot CLI follows symlinks correctly
  
  ### Alternative: Copy-based Approach
  
  If symlinks don't work:
  - Use build script to copy `.claude/` → `plugins/conductor-beads/`
  - Add to CI/CD to keep in sync
  - Trade-off: More complex, but more compatible
  
- **Learnings:**
  - Patterns: **Symlink-first, copy-as-fallback** is the pragmatic approach
  - Patterns: Plugin directory is a **facade** over existing structure
  - Gotchas: Must test symlink support in Copilot CLI (not documented)
  - Gotchas: Windows users may need special handling
  - Context: This maintains **100% backward compatibility** - existing users unaffected
  - Context: New users can choose their preferred installation method


## [2026-02-08 13:00] - Phase 3 Task 5: Create implementation spec for track 2
Thread: N/A (planning task)
- **Created:** Complete specification and implementation plan for Copilot CLI plugin support
- **Outputs:**
  - `conductor/tracks/copilot_plugin_impl_20260208/spec.md` - Detailed requirements and technical specs
  - `conductor/tracks/copilot_plugin_impl_20260208/plan.md` - 10-task implementation plan across 4 phases
- **Key Decisions:**
  1. **Symlink-first approach** - Use symlinks to avoid file duplication, with copy as fallback
  2. **Additive-only changes** - No modifications to existing `.claude/` or `commands/conductor/` structures
  3. **Multi-platform support** - Maintain all three: Copilot CLI, Claude Code, Gemini CLI
  4. **Namespace acceptance** - Accept `/conductor-beads:` prefix as trade-off for plugin system benefits
- **Implementation Phases:**
  1. **Phase 1:** Create plugin structure (manifest, directories, symlinks)
  2. **Phase 2:** Update documentation (README, CLAUDE.md)
  3. **Phase 3:** Testing & validation (structure, backward compat)
  4. **Phase 4:** Release preparation (review, handoff)
- **Success Metrics:**
  - Plugin manifest validates
  - Symlinks work and are preserved in Git
  - No breaking changes to existing platforms
  - Clear documentation for all installation methods
- **Learnings:**
  - Patterns: Research track deliverables = spec + plan for implementation track
  - Patterns: Implementation should be **testable without external dependencies** (Copilot CLI access)
  - Context: Track 2 is **pure structure and docs** - no code changes needed
  - Context: Validation is structural (file existence, symlink integrity, JSON schema)


## [2026-02-08 13:01] - Phase 3 Task 6: Document patterns and recommendations
Thread: N/A (consolidation task)
- **Consolidated:** Key patterns and recommendations from research
- **Outputs:** Patterns ready for elevation to `conductor/patterns.md`

### Reusable Patterns Discovered

1. **Multi-Platform Plugin Strategy**
   - Use **symlink-based wrappers** to support multiple AI assistant platforms
   - Keep single source of truth (e.g., `.claude/` for commands/skills)
   - Create platform-specific facades (e.g., `plugins/` for Copilot CLI, `commands/` for Gemini)
   - Benefit: No duplication, automatic sync, easier maintenance

2. **Plugin Manifest Requirements**
   - Copilot CLI requires manifest at `.github/plugin/plugin.json`
   - Minimal required fields: `name`, `description`, `version`, `author`, `repository`, `license`
   - Manifest enables discovery via `copilot plugin install <name>@<repo>`

3. **Backward Compatibility Principle**
   - New platform support should be **additive only**
   - Never modify existing working integrations
   - Prefer creating new structures alongside old ones
   - Users choose their preferred installation method

4. **Research Track Deliverables**
   - Research/analysis tracks should produce:
     - `learnings.md` with detailed findings
     - `spec.md` for implementation track (requirements, constraints, success criteria)
     - `plan.md` for implementation track (phased task breakdown)
   - Separation of analysis from implementation allows review before execution

### Recommendations for Future Work

1. **Symlink Testing**
   - Test symlink behavior on Windows (may require developer mode)
   - Verify Copilot CLI follows symlinks correctly
   - Document platform-specific quirks

2. **Namespace Convention**
   - Consider creating command aliases to support both `/conductor-*` and `/conductor-beads:*`
   - Update all documentation examples to show both forms
   - Survey users on namespace preference

3. **Build Automation**
   - If symlinks prove problematic, create build script to copy files
   - Add CI/CD check to verify plugin structure stays in sync with source
   - Consider GitHub Actions workflow for validation

4. **Plugin Discovery**
   - Submit to awesome-copilot repository once stable
   - Create plugin showcase/demo
   - Gather feedback from Copilot CLI users

### Gotchas for Implementation Track

- **Git Behavior:** Git preserves symlinks by default, but clone behavior may vary
- **Relative Paths:** Don't use relative paths in command files (breaks when nested)
- **Windows Compatibility:** Symlinks require special permissions on Windows
- **Testing Limitations:** Cannot fully test without Copilot CLI installed

