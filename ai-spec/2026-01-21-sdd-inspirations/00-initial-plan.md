# User request

Improve the `sdd-plan` skill (plugins/kg/skills/sdd-plan/SKILL.md) by analyzing reference SDD-like implementations:
- Ralph-loop plugin (iterative development with self-reference loops)
- Feature-dev plugin (7-phase structured feature development)
- Obra Superpowers workflow (comprehensive TDD-driven development system)

Goals:
- Identify strengths and weaknesses of each approach
- Extract best practices applicable to `sdd-plan`
- Propose improvements to the current `sdd-plan` skill
- Focus on LLM engineering and developer experience perspectives

## Summary of a plan

**Core approach**: Enhance `sdd-plan` with battle-tested patterns from three complementary sources while preserving its unique two-pass subagent consensus-building approach.

The plan will:
1. **Adopt structured phase gates** from feature-dev (explicit approval points, clarifying questions phase)
2. **Integrate TDD-first planning** from Obra Superpowers (executable verification, bite-sized tasks)
3. **Add decision documentation** framework (capture rationale for disagreement resolution)
4. **Strengthen feedback structure** with templates and severity levels
5. **Create implementation bridge** between spec and execution (task mapping, verification commands)
6. **Maintain SDD core value**: Multi-perspective review with positive-sum consensus building

## Alternatives and rationale

### Alternative 1: Minimal Enhancement (Selected)
**Approach**: Add missing best practices without restructuring the core 8-stage workflow
- Add clarifying questions phase between exploration and initial plan
- Standardize feedback templates with required sections
- Add decision log and risk register to spec.md
- Include executable verification steps in plan details
- Create implementation task mapping section

**Pros**:
- Preserves proven two-pass subagent pattern
- Low disruption to existing users
- Incremental improvement path
- Maintains unique positive-sum thinking approach

**Cons**:
- Doesn't address all identified gaps
- May require follow-up iterations
- Won't match feature-dev's phase clarity

### Alternative 2: Complete Workflow Redesign
**Approach**: Restructure to match feature-dev's 7-phase model with explicit gates
- Separate discovery, exploration, clarification, architecture, implementation planning phases
- Replace two-pass feedback with parallel multi-focus agents
- Add explicit quality gates between phases
- Complete rewrite of templates

**Pros**:
- Industry-standard phase structure
- Clear gate definitions
- Better alignment with other tools

**Cons**:
- Breaks existing usage patterns
- Loses unique two-pass consensus building
- High migration cost for users
- Over-engineering for planning-only workflow

### Alternative 3: Modular Skill Decomposition
**Approach**: Break sdd-plan into multiple composable skills
- sdd-explore (codebase exploration)
- sdd-plan (planning only)
- sdd-review (feedback gathering)
- sdd-synthesize (final spec generation)

**Pros**:
- Maximum flexibility
- Users can skip phases
- Better testing isolation

**Cons**:
- Complexity explosion
- State management challenges
- Doesn't address core quality issues
- May fragment user experience

### Rationale for Alternative 1

The current `sdd-plan` skill has a **proven, sophisticated two-pass subagent approach** that distinguishes it from competitors. The issue isn't the core workflow—it's **missing formalization of best practices** that exist implicitly.

Feature-dev and Obra Superpowers show that:
- Explicit phase gates prevent downstream rework
- Structured feedback templates ensure consistency
- Decision documentation aids long-term maintainability
- Executable verification enables automation

These can be **added without disrupting the core workflow**.

## Relevant current code

### Current sdd-plan Structure (plugins/kg/skills/sdd-plan/SKILL.md)

**8-stage workflow**:
1. Understand user request (clarification loop)
2. Explore codebase (parallel Haiku agents)
3. Propose initial design (architect ultrathinking)
4. Create initial plan (00-initial-plan.md with template)
5. Initial plan user refinement (approval gate)
6. Subagents first reading (01-feedback-{role}.md)
7. Subagents second reading (02-feedback-{role}.md with cross-agent awareness)
8. Final review (spec.md synthesis)

**Key patterns**:
- **Two-pass feedback**: First pass independent, second pass with full context and consensus-seeking
- **Positive-sum thinking**: Explicit instruction for agents to find compatible solutions
- **User approval gate**: Step 5 blocks until user approves
- **Read-only constraint**: No implementation during planning
- **Flexible subagent selection**: 8 available roles, 4-6 selected per project

**Existing strengths to preserve**:
- Multi-perspective review
- Consensus-building approach
- Explicit alternatives analysis
- Operational impact consideration
- Clear directory structure

### Reference Patterns to Incorporate

**From feature-dev** (ai-spec/2026-01-21-sdd-inspirations/references/anthropics-claude-plugins-official/plugins/feature-dev/):
- Clarifying questions phase (Phase 3) BEFORE architecture
- Parallel agent execution with different focuses
- Confidence-based filtering (80+ threshold)
- Explicit decision points with user control
- Severity-based issue categorization

**From Obra Superpowers** (ai-spec/2026-01-21-sdd-inspirations/references/obra-superpowers/):
- Bite-sized tasks (2-5 minutes each)
- Every task has: Files, Do, Verify sections
- Verification commands are executable
- TDD-first structure (failing test → code → passing test)
- Complete specification with exact file paths
- DRY/YAGNI/TDD emphasis throughout

**From ralph-loop** (ai-spec/2026-01-21-sdd-inspirations/references/anthropics-claude-plugins-official/plugins/ralph-loop/):
- Explicit completion criteria
- Iteration awareness
- State persistence patterns
- Safety mechanisms (max iterations, validation)

### Architecture & Design Patterns

Current `sdd-plan` uses:
- **Markdown-based state**: All artifacts in `ai-spec/YYYY-MM-DD-{description}/`
- **File naming convention**: Prefixes (00, 01, 02) + role suffixes
- **Template-driven planning**: 00-initial-plan.md has structured sections
- **Agent orchestration**: Uses Task tool with parallel execution

## Functional requirements

### Core Enhancements

1. **Add Clarifying Questions Phase (Step 2.5)**
   - Insert between "Explore codebase" and "Propose initial design"
   - Generate questions about: edge cases, integration points, scope, performance, compatibility
   - Block progression until user answers
   - Format: organized list with context

2. **Standardize Feedback Templates**
   - Create explicit template for 01-feedback-{role}.md
   - Create explicit template for 02-feedback-{role}.md
   - Required sections:
     - **Summary**: 2-3 sentence perspective
     - **Risks Identified**: What could go wrong
     - **Concrete Improvements**: Specific, actionable suggestions
     - **Questions for Other Roles**: Cross-role concerns
     - **Confidence Level**: High/Medium/Low for each point

3. **Add Decision Log Section to spec.md**
   - Document subagent disagreements
   - Explain resolution rationale
   - Capture alternatives considered
   - Format: "Issue → Options → Decision → Rationale"

4. **Add Risk Register to spec.md**
   - Structured list of identified risks
   - Mitigation strategies
   - Severity levels (Critical/High/Medium/Low)
   - Owner/timeline if applicable

5. **Enhance Plan Details with Executable Verification**
   - Each task includes:
     - **Files**: Exact paths to create/modify
     - **Do**: Step-by-step actions
     - **Verify**: Executable commands with expected output
   - Make verification machine-executable (no human judgment needed)

6. **Add Implementation Task Mapping**
   - Link spec sections to concrete tasks
   - Group tasks by dependency order
   - Estimate complexity (Small/Medium/Large)
   - Identify critical path

7. **Require Rollback Strategy Section**
   - Make rollback strategy mandatory (not optional)
   - Include: detection, rollback procedure, validation
   - Address: deployment risk, data migration, config changes

8. **Add Testing Strategy Section**
   - Unit test coverage expectations
   - Integration test requirements
   - E2E test scenarios
   - Performance test criteria

### Quality Improvements

9. **Add Contradiction Validation Phase**
   - After second-pass feedback, check for conflicting requirements
   - Document contradictions explicitly
   - Require resolution before proceeding

10. **Create Decision Framework Documentation**
    - Provide principle hierarchy for conflict resolution
    - Example: "Security > Performance > Optimization"
    - Project-specific values documented in CLAUDE.md

11. **Add Structured Alternatives Comparison**
    - Replace prose with pros/cons matrix
    - Include: complexity, maintainability, performance, risk
    - Make visual/scannable

## Non-functional requirements

### Maintainability
- All templates remain markdown-based
- File naming convention preserved
- No external dependencies added
- Changes should be backward-compatible with existing SPEC directories

### Usability (DX)
- Enhanced templates should reduce cognitive load
- Clarifying questions phase prevents downstream confusion
- Executable verification enables automation
- Decision documentation aids future maintenance

### Performance
- Parallel agent execution maintained
- No additional sequential phases (unless explicitly beneficial)
- Clarifying questions phase adds latency but prevents rework

### Extensibility
- New subagent roles can be added without template changes
- Feedback template sections can be customized per project
- Decision framework can be project-specific

## Maintainability & Operational impact

### Impact on Existing Users
**Low disruption expected**:
- Core 8-stage workflow unchanged
- Existing SPEC directories remain valid
- New features are additions, not replacements
- Templates are enhanced, not restructured

### Changes to Procedures
1. **Clarifying questions phase adds one approval gate** (acceptable tradeoff for preventing rework)
2. **Feedback templates add structure** (reduces variance, improves consistency)
3. **Decision log adds documentation burden** (small cost for long-term value)

### Code Patterns Intentionally Broken
**None**. This is an enhancement, not a breaking change.

### Deployment Risk
**Minimal**:
- Changes are to a markdown skill file
- No runtime dependencies
- Users can test with new SPEC directory without affecting existing ones
- Rollback is simple (revert SKILL.md)

### Rollback Strategy
If enhanced sdd-plan causes issues:
1. Revert plugins/kg/skills/sdd-plan/SKILL.md to previous version
2. Document specific issue encountered
3. Create new SPEC directory with reverted version for testing
4. Existing SPEC directories unaffected

## Open questions, future considerations

### Questions to Resolve
1. **Should clarifying questions be mandatory or optional?** (Recommendation: mandatory for complex features, optional for simple improvements)
2. **How many questions is too many?** (Recommendation: 5-10 max, grouped by category)
3. **Should feedback templates be strictly enforced?** (Recommendation: yes for consistency)
4. **Should we add a third-pass feedback for major disagreements?** (Recommendation: no, adds complexity without clear value)

### Future Considerations
1. **Integration with implementation workflows**: Could sdd-plan output feed into subagent-driven-development?
2. **Automated validation**: Could we validate spec.md against implementation automatically?
3. **Specification versioning**: If specs need amendments, how to track changes?
4. **Traceability matrix**: Link requirements to code (heavy lift, may not justify cost)
5. **Template customization**: Should projects define their own feedback templates?
6. **AI-assisted decision resolution**: Could we use an additional "mediator" agent to propose resolutions for disagreements?

## Plan details

### Phase 1: Template & Structure Enhancements
**Goal**: Improve planning artifacts without changing workflow

- [ ] **Update 00-initial-plan.md template** (plugins/kg/skills/sdd-plan/SKILL.md:104-145)
  - Add "Testing Strategy" section after "Non-functional requirements"
  - Make "Rollback Strategy" mandatory within "Maintainability & Operational impact"
  - Add "Risk Register" section after "Open questions"
  - Enhance "Plan details" with Files/Do/Verify subsections for each task
  - Add "Implementation Task Mapping" section before "Plan details"

- [ ] **Create 01-feedback-{role}.md template** (new section in SKILL.md)
  - Add explicit template with required sections:
    - Summary (2-3 sentences)
    - Risks Identified (what could go wrong from this role's perspective)
    - Concrete Improvements (specific, actionable suggestions with file:line references)
    - Questions for Other Roles (cross-role concerns)
    - Confidence Level (High/Medium/Low for each point)

- [ ] **Create 02-feedback-{role}.md template** (new section in SKILL.md)
  - Extend 01-feedback template with additional sections:
    - Changes from First Pass (what changed and why)
    - Consensus Opportunities (where agreement with other agents exists)
    - Unresolved Disagreements (where consensus isn't possible, with rationale)
    - Positive-Sum Integrations (how other feedback improved this perspective)

- [ ] **Update spec.md template** (add to step 8 in SKILL.md:196-204)
  - Add "Decision Log" section (Issue → Options → Decision → Rationale format)
  - Add "Risk Register" section (Risk, Severity, Mitigation, Owner)
  - Add "Structured Alternatives Comparison" (table format with pros/cons/tradeoffs)
  - Add "Implementation Task Mapping" (spec sections → tasks with dependencies)

### Phase 2: Workflow Integration
**Goal**: Add clarifying questions phase without disrupting core flow

- [ ] **Insert Step 2.5: Clarifying Questions Phase** (between current steps 2 and 3)
  - Add to SKILL.md after "Explore the codebase" section
  - Instruction: Generate 5-10 questions organized by category:
    - Edge cases and error handling
    - Integration points and scope boundaries
    - Performance and scalability
    - Backward compatibility
    - Design preferences
  - Require user answers before proceeding to "Propose initial design"
  - Use AskUserQuestion tool with organized presentation

- [ ] **Update Step 6: Subagents First Reading** (SKILL.md:156-170)
  - Reference new 01-feedback template
  - Add instruction: "Follow the 01-feedback-{role}.md template structure"
  - Ensure each subagent knows about required sections

- [ ] **Update Step 7: Subagents Second Reading** (SKILL.md:173-193)
  - Reference new 02-feedback template
  - Add instruction: "Follow the 02-feedback-{role}.md template structure"
  - Emphasize cross-agent awareness and consensus-building
  - Add explicit validation: check for contradictions between subagents

- [ ] **Update Step 8: Final Review** (SKILL.md:195-204)
  - Add contradiction validation sub-step
  - Add decision framework guidance
  - Reference new spec.md sections (Decision Log, Risk Register)
  - Require documentation of all disagreement resolutions

### Phase 3: Documentation & Guidance
**Goal**: Help users understand and leverage enhancements

- [ ] **Add "Decision Framework" section to SKILL.md** (after Subagents section)
  - Provide default principle hierarchy (Security > Maintainability > Performance > Optimization)
  - Note: can be overridden by project-specific CLAUDE.md
  - Explain when to prioritize different concerns

- [ ] **Add "Best Practices" section to SKILL.md** (near end)
  - Executable verification commands
  - Bite-sized task design (2-5 minutes each)
  - TDD-first thinking (failing test → code → passing test)
  - DRY/YAGNI enforcement
  - Clear completion criteria

- [ ] **Update SPEC directory anatomy documentation** (SKILL.md:50-76)
  - Show example with new sections in 00-initial-plan.md
  - Show example 01-feedback and 02-feedback with template structure
  - Show example spec.md with Decision Log and Risk Register

- [ ] **Add examples section** (new, after main workflow)
  - Example clarifying questions for different project types
  - Example decision log entries
  - Example risk register entries
  - Example executable verification commands

### Phase 4: Validation & Testing
**Goal**: Ensure enhancements work as intended

- [ ] **Create test SPEC directory** (ai-spec/2026-01-21-test-sdd-enhancements/)
  - Use updated SKILL.md to create a test plan
  - Task: "Add a simple authentication feature to a mock project"
  - Validate all new sections are generated correctly
  - Check for template compliance

- [ ] **Verify backward compatibility**
  - Review existing SPEC directories
  - Confirm they remain valid
  - Ensure no breaking changes introduced

- [ ] **Document migration guidance**
  - Create section: "Upgrading from previous sdd-plan versions"
  - Explain new features
  - Provide optional migration steps for existing SPECs

- [ ] **User testing with LLM and DX engineers**
  - Run through workflow with updated skill
  - Gather feedback on usability
  - Identify any remaining gaps or friction points

### Testing Strategy

**Unit-level verification**:
- Each template section is complete and well-structured
- All referenced file paths exist and are correct
- Markdown syntax is valid throughout

**Integration verification**:
- Run complete sdd-plan workflow with test project
- Verify all phases execute correctly
- Confirm subagent feedback follows templates
- Validate final spec.md contains all required sections

**User acceptance**:
- DX engineer reviews for usability
- LLM engineer reviews for AI-friendliness
- Feedback incorporated before finalization

**Success criteria**:
- All template sections render correctly
- Clarifying questions phase blocks progression until answered
- Subagent feedback follows consistent structure
- Decision Log captures all disagreement resolutions
- Risk Register identifies project-specific concerns
- Implementation Task Mapping links spec to execution
- Existing SPEC directories remain valid
