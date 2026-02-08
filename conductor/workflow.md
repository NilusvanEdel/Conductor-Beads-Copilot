# Conductor-Beads Development Workflow

## Guiding Principles

1. **Documentation is Code**: All commands, skills, and templates are text-based artifacts
2. **Cross-Platform Consistency**: Maintain parity between Gemini CLI and Claude Code
3. **Interoperability**: Conductor and Beads must work together seamlessly
4. **Graceful Degradation**: Commands work without Beads (prompt user to continue)
5. **Human-Readable**: All artifacts must be readable markdown/JSON
6. **Test Before Release**: Validate commands in both Gemini CLI and Claude Code

## Status Markers

- `[ ]` - Pending/New
- `[~]` - In Progress  
- `[x]` - Completed
- `[!]` - Blocked (with reason)
- `[-]` - Skipped (with justification)

## Development Workflow

### For New Commands

1. **Spec the Command**: Create track with spec.md defining:
   - Command syntax
   - Expected behavior
   - Input validation
   - Error handling
   - Beads integration (if applicable)

2. **Implement Gemini Version**: Create `.toml` in `commands/conductor/`
   - Define command metadata
   - Write prompt with all validations
   - Reference templates as needed

3. **Implement Claude Version**: Create `.md` in `.claude/commands/`
   - Match Gemini functionality
   - Use markdown format
   - Maintain consistent behavior

4. **Update Documentation**: 
   - Add to README.md command table
   - Update CLAUDE.md and GEMINI.md context files
   - Add to relevant skill references

5. **Test Both Platforms**:
   - Verify in Gemini CLI
   - Verify in Claude Code
   - Test with/without Beads
   - Test error conditions

6. **Commit**: Use format `feat(commands): add <command-name>`

### For Skills

1. **Define Skill**: Create SKILL.md with:
   - Activation conditions
   - Context requirements
   - Proactive behaviors
   - Intent mappings

2. **Add References**: Create references/ subdirectory with relevant docs

3. **Test Auto-Activation**: Verify skill loads when conditions met

4. **Document**: Update README.md skills table

### For Templates

1. **Create/Update**: Edit files in `templates/`
2. **Validate Format**: Ensure markdown is well-formed
3. **Test Usage**: Use template in a test command
4. **Document Variables**: Explain placeholders clearly

## Commit Convention

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types
- `feat`: New command, skill, or feature
- `fix`: Bug fix in existing command/skill
- `docs`: Documentation changes (README, context files)
- `refactor`: Code restructuring without behavior change
- `test`: Add or update tests
- `chore`: Maintenance (dependencies, tooling)

### Scopes
- `commands`: Conductor commands (both Gemini and Claude)
- `skills`: Skills (conductor, beads, skill-creator)
- `templates`: Template files
- `docs`: Documentation files
- `integration`: Beads or platform integration

### Examples
```
feat(commands): add conductor-formula for template management
fix(skills): conductor skill now handles missing beads.json
docs(readme): update parallel execution section
refactor(commands): consolidate validation logic
```

## Git Policy

**Critical**: All commits stay local. Never push automatically. Users decide when to push.

## Testing Strategy

### Manual Testing Required
- Run command in Gemini CLI
- Run command in Claude Code
- Test with Beads enabled
- Test without Beads (graceful degradation)
- Test error paths (missing files, invalid state)

### Validation Checklist
- [ ] Command works in Gemini CLI
- [ ] Command works in Claude Code
- [ ] Beads integration (if applicable) functions correctly
- [ ] Error messages are clear and actionable
- [ ] Documentation updated (README, context files)
- [ ] Cross-references updated (skills, command lists)

## Release Process

1. **Version Bump**: Update version in README.md
2. **Test Suite**: Run full manual test suite
3. **Documentation Review**: Ensure all docs are current
4. **Commit**: `chore(release): bump version to X.Y.Z`
5. **Tag**: `git tag vX.Y.Z`
6. **Push**: User decides when to push to GitHub

## Continuous Improvement

### Learnings Capture
After implementing features:
- Record patterns in track's `learnings.md`
- Elevate to `patterns.md` at track completion
- Use patterns to inform future work

### Command Evolution
- Monitor user feedback
- Identify repetitive workflows → create new commands
- Consolidate redundant logic → refactor
- Improve error messages based on common mistakes
