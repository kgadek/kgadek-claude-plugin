# User request

Thorough review of the current `kg` plugin - identify issues, improvements, and create a comprehensive improvement plan.

## Summary of a plan

Perform a comprehensive review of the `kg` plugin to identify:
1. Bugs and issues (typos, broken references)
2. Missing functionality and infrastructure
3. Clarity and usability improvements
4. Structural/architectural improvements
5. Documentation gaps

The main improvement areas identified are:
- Fix immediate issues (typos, unclear references)
- Add missing repository infrastructure (CLAUDE.md, README)
- Improve skill clarity and robustness
- Add validation and error handling guidance
- Remove references to non-existent features

## Alternatives and rationale

**Alternative 1: Minimal fixes only**
- Fix only typos and broken references
- Pros: Quick, low risk
- Cons: Doesn't address structural issues

**Alternative 2: Comprehensive improvement (SELECTED)**
- Fix all identified issues
- Add missing infrastructure
- Improve documentation
- Pros: Results in a robust, production-ready plugin
- Cons: More work

**Alternative 3: Complete rewrite**
- Start fresh with lessons learned
- Pros: Clean slate
- Cons: Loses existing good patterns, more time

**Rationale:** Alternative 2 was selected because the existing structure is sound, but has gaps that need filling. The two skills are well-designed but need polish.

## Relevant current code

### Plugin Structure
```
kgadek-claude-plugin/
├── .claude-plugin/
│   └── marketplace.json       # Plugin registry config
├── LICENSE                    # AGPL-3.0-only
└── plugins/
    └── kg/
        └── skills/
            ├── sdd-plan/
            │   └── SKILL.md   # 221 lines, SDD planning workflow
            └── improve-skills/
                └── SKILL.md   # 180 lines, skill improvement analysis
```

### Current Patterns
- Skills defined in `SKILL.md` files with YAML frontmatter
- Frontmatter includes: `name`, `description`, optional `disable-model-invocation`
- Skills use markdown with structured sections
- Skills reference subagent patterns and dynamic context

### Key Issues Identified

1. **sdd-plan/SKILL.md:209** - Typo: "secrity specialist" should be "security specialist"

2. **improve-skills/SKILL.md:21** - References `${CLAUDE_SESSION_ID}` - unclear if this variable is available in the Claude Code skill system

3. **improve-skills/SKILL.md** - References non-existent features:
   - `user-invocable: false` (line 133, 148)
   - `context: fork` (line 149)
   - `argument-hint` (line 114)
   - Dynamic context injection with bang+backtick syntax (lines 106-113)

4. **Missing infrastructure:**
   - No repository-level `CLAUDE.md`
   - No `README.md` for the plugin
   - No usage examples

5. **sdd-plan skill gaps:**
   - No explicit instruction on how to spawn subagents (Task tool usage)
   - No guidance on subagent model selection (haiku vs sonnet)
   - No error handling guidance
   - Unclear how to handle argument validation

## Functional requirements

### FR1: Fix typos and errors
- Fix "secrity" typo in sdd-plan skill

### FR2: Clarify or remove non-existent feature references
- Either implement or remove references to `${CLAUDE_SESSION_ID}`
- Either implement or document that features like `user-invocable`, `context: fork` are planned but not yet available

### FR3: Add missing documentation
- Create repository README.md
- Create CLAUDE.md with repository-level guidance
- Add usage examples for both skills

### FR4: Improve skill robustness
- Add guidance on subagent invocation patterns
- Add error handling guidance
- Clarify argument handling

### FR5: Add plugin documentation
- Document the skill frontmatter schema
- Document available metadata options

## Non-functional requirements

### NFR1: Clarity
- Skills should be understandable by both humans and Claude
- No ambiguous instructions

### NFR2: Consistency
- Consistent formatting across all skills
- Consistent terminology

### NFR3: Maintainability
- Easy to update and extend skills
- Clear separation of concerns

### NFR4: Completeness
- All referenced features should exist or be clearly marked as planned

## Maintainability & Operational impact

**Impact:**
- Minimal operational impact - plugin is personal and not widely deployed
- Changes improve maintainability by:
  - Removing confusing non-existent feature references
  - Adding documentation for future reference
  - Fixing typos that could cause confusion

**Rollback:**
- Git-based rollback available
- No production dependencies

**Deployment:**
- Update version to 0.0.2 after changes
- No special deployment procedures needed

## Open questions, future considerations

1. **Claude Code skill system features:** What frontmatter options are actually supported?
   - `name` and `description` confirmed
   - `disable-model-invocation` appears to be used but needs verification
   - Other options like `user-invocable`, `context`, `argument-hint` - are these planned?

2. **Variable interpolation:** Is `${CLAUDE_SESSION_ID}` actually available? Are there other variables?

3. **Dynamic context injection:** Is the "bang+backtick" syntax (`!backtick`) actually supported for dynamic context?

4. **Subagent best practices:** What's the recommended way to invoke subagents from skills?

5. **Future skills:** What other skills might be useful to add to this plugin?

## Plan details

### Phase 1: Immediate fixes
- [ ] Fix typo "secrity" -> "security" in `sdd-plan/SKILL.md:209`
- [ ] Review and clarify non-existent feature references in `improve-skills/SKILL.md`
- [ ] Test: Verify skills load correctly after changes

### Phase 2: Documentation
- [ ] Create `README.md` for the repository with:
  - Plugin overview
  - Installation instructions
  - Available skills and their usage
  - License information
- [ ] Create `plugins/kg/README.md` with skill-specific documentation
- [ ] Test: Verify documentation is accurate and complete

### Phase 3: Skill improvements
- [ ] Update `sdd-plan/SKILL.md`:
  - Add explicit subagent invocation guidance (use Task tool)
  - Add model selection guidance (haiku for exploration, sonnet for planning)
  - Add error handling guidance
  - Clarify argument handling/validation
- [ ] Update `improve-skills/SKILL.md`:
  - Remove or mark non-existent features as "planned"
  - Clarify the purpose of `disable-model-invocation`
  - Add more generic examples (not just GitLab)
- [ ] Test: Run both skills and verify they work as expected

### Phase 4: Repository infrastructure
- [ ] Create `CLAUDE.md` at repository root with:
  - Repository overview
  - Contribution guidelines
  - References to skill documentation
- [ ] Consider adding `.clauderc` for repository defaults
- [ ] Test: Verify Claude Code recognizes and uses the infrastructure

### Phase 5: Version update
- [ ] Update version in `marketplace.json` from 0.0.1 to 0.0.2
- [ ] Update any version references
- [ ] Test: Verify marketplace.json is valid JSON
- [ ] Commit changes with descriptive message
