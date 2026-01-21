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

**Core approach**: Restructure `sdd-plan` as a hybrid 7-phase workflow that combines feature-dev's clarity with sdd-plan's depth, enhanced by Obra Superpowers' executable verification practices.

The plan will:
1. **Adopt feature-dev's 7-phase structure** for workflow clarity and explicit gates
2. **Enhance Phase 4 (Architecture)** with three-pass design: two-pass consensus (4a/4b) + concrete alternatives exploration (4c)
3. **Integrate TDD-first planning** from Obra Superpowers (executable verification, bite-sized tasks, Files/Do/Verify structure)
4. **Add decision documentation** framework (decision log, risk register, structured alternatives comparison)
5. **Strengthen feedback structure** with templates and severity levels for all subagent passes
6. **Create implementation bridge** between spec and execution (task mapping, verification commands)
7. **Maintain SDD core value**: Multi-perspective review with positive-sum consensus building in Phase 4a/4b

## Alternatives and rationale

### Alternative 1: Minimal Enhancement
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
- Keeps workflow structure that users may find less intuitive

### Alternative 2: Hybrid 7-Phase Workflow with Enhanced Architecture Design (Selected)
**Approach**: Adopt feature-dev's clear 7-phase structure, but enhance Phase 4 with sdd-plan's two-pass consensus model

**Phase Structure**:
1. **Discovery** - Clarify requirements and context
2. **Codebase Exploration** - Parallel agents explore different aspects
3. **Clarifying Questions** - Block until ambiguities resolved
4. **Architecture Design** - THREE-PASS approach:
   - **Phase 4a (First Consensus)**: 4-6 specialized subagents provide independent architectural feedback
   - **Phase 4b (Second Consensus)**: Same subagents read all first-pass feedback, apply positive-sum thinking to find common ground
   - **Phase 4c (Tradeoff Exploration)**: 2-3 code-architect agents propose concrete designs with different tradeoff focuses (minimal changes, clean architecture, pragmatic balance)
5. **Implementation Planning** - User selects approach, create detailed implementation plan with executable verification
6. **Plan Quality Review** - Parallel review of the plan for completeness, risks, testability
7. **Final Spec** - Synthesize all feedback into spec.md with decision log

**Pros**:
- Clear, intuitive phase structure from feature-dev
- Preserves two-pass consensus building from sdd-plan
- Adds concrete architectural alternatives exploration
- Explicit gates prevent premature decisions
- Best of both worlds: structure + depth
- Three-pass architecture (4a/4b/4c) separates brainstorming from concrete design

**Cons**:
- More phases means more latency
- Higher complexity than current approach
- Requires relearning workflow for existing users
- Three-pass Phase 4 might feel heavy for simple features

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

### Rationale for Alternative 2 (Hybrid Approach)

**Key insight**: Feature-dev and sdd-plan solve different problems optimally:
- **Feature-dev**: Clear phase structure with explicit gates (great for workflow clarity)
- **Current sdd-plan**: Two-pass consensus building (great for thorough architectural thinking)

**Why not just enhance the current 8-stage workflow?** (Alternative 1)
- Users find the current structure less intuitive
- Missing explicit gates between conceptual phases
- Hard to know "where am I in the process?"
- Phases blend together (exploration → design → planning)

**Why not just adopt feature-dev as-is?** (Original Alternative 2)
- Would lose the sophisticated two-pass consensus approach
- Single-pass architectural feedback misses nuance
- No mechanism for positive-sum thinking across perspectives

**The hybrid approach synthesizes both**:
1. **Adopt feature-dev's 7-phase structure** for workflow clarity
2. **Enhance Phase 4 (Architecture Design)** with three passes:
   - **4a + 4b**: Two-pass consensus from sdd-plan (brainstorming depth)
   - **4c**: Concrete alternatives from feature-dev (tradeoff exploration)
3. **Best of both worlds**: Clear structure + sophisticated architectural thinking

**Separation of concerns in Phase 4**:
- **Phases 4a/4b**: Broad architectural thinking from domain experts (backend, frontend, QA, DevOps, security)
- **Phase 4c**: Concrete design proposals from architects with different optimization goals

This prevents mixing "what should we consider?" (4a/4b) with "here's a concrete design" (4c).

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

### Phase 1: Define New 7-Phase Workflow Structure
**Goal**: Replace current 8-stage workflow with hybrid 7-phase approach

- [ ] **Rewrite main workflow section in SKILL.md** (currently "The SDD planning process" at lines 79-204)
  Files: plugins/kg/skills/sdd-plan/SKILL.md
  Do:
    - Replace current 8-stage workflow with new 7-phase structure
    - Keep "PLANNING-ONLY MODE" constraint and SPEC directory anatomy sections unchanged
    - Maintain core philosophy and objectives
  Verify:
    - SKILL.md contains exactly 7 phases (not 8 stages)
    - Each phase has clear description and purpose
    - Phase 4 explicitly shows three sub-phases (4a, 4b, 4c)

- [ ] **Write Phase 1: Discovery**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Phase 1 section)
  Do:
    - Clarify user request through direct engagement
    - Ask questions to resolve ambiguities
    - Confirm understanding before proceeding
    - Output: Clear, validated requirements
  Verify:
    - Phase includes explicit "ultrathink" architect perspective
    - Uses AskUserQuestion tool for clarification
    - Has approval gate before Phase 2

- [ ] **Write Phase 2: Codebase Exploration**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Phase 2 section)
  Do:
    - Launch 2-3 Haiku agents in parallel, each exploring different aspects:
      - Existing SPEC files relevant to request
      - Similar features in current codebase
      - Architecture patterns and design decisions
    - Use WebSearch for external API/schema verification
    - Agents return 5-10 key files to read
    - Human reads identified files for deep context
  Verify:
    - Uses Task tool with model=haiku for parallel exploration
    - Each agent has distinct focus area
    - Outputs specific file paths with line numbers

- [ ] **Write Phase 3: Clarifying Questions**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Phase 3 section)
  Do:
    - Based on codebase exploration, generate 5-10 questions organized by category:
      - Edge cases and error handling
      - Integration points and scope boundaries
      - Performance and scalability requirements
      - Backward compatibility concerns
      - Design preferences and tradeoffs
    - Use AskUserQuestion tool with organized presentation
    - CRITICAL: Block progression until all questions answered
  Verify:
    - Marked as "CRITICAL: DO NOT SKIP"
    - Uses AskUserQuestion tool
    - Clear categories for questions
    - Explicit blocking gate

- [ ] **Write Phase 4a: Architecture First Consensus**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Phase 4a section)
  Do:
    - Launch 4-6 specialized subagents in parallel (architect, backend-eng, frontend-eng, qa-eng, devops-eng, security)
    - Each provides INDEPENDENT architectural perspective
    - Focus: broad architectural thinking, not concrete designs
    - Each writes feedback to: ai-spec/{YYYY-MM-DD}-{description}/01-feedback-{role}.md
    - Use standardized template with sections: Summary, Risks, Improvements, Questions, Confidence
  Verify:
    - Uses Task tool with parallel execution
    - 4-6 feedback files created (01-feedback-*.md)
    - Each follows template structure
    - Feedback is independent (no cross-referencing yet)

- [ ] **Write Phase 4b: Architecture Second Consensus**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Phase 4b section)
  Do:
    - Same subagents read ALL first-pass feedback (01-feedback-*.md)
    - Apply positive-sum thinking to find common ground
    - Identify areas of agreement and disagreement
    - Each writes extended feedback to: ai-spec/{YYYY-MM-DD}-{description}/02-feedback-{role}.md
    - Use extended template adding: Changes from First Pass, Consensus Opportunities, Unresolved Disagreements, Positive-Sum Integrations
  Verify:
    - 4-6 feedback files created (02-feedback-*.md)
    - Each references other agents' feedback
    - Explicit consensus and disagreement documentation
    - Positive-sum thinking evident

- [ ] **Write Phase 4c: Architecture Concrete Alternatives**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Phase 4c section)
  Do:
    - Launch 2-3 code-architect agents in parallel with different optimization focuses:
      - Minimal changes (maximum reuse of existing code)
      - Clean architecture (long-term maintainability priority)
      - Pragmatic balance (speed + quality tradeoff)
    - Each proposes CONCRETE design with:
      - Specific file paths to create/modify
      - Component architecture
      - Build sequence and phases
      - Tradeoffs and rationale
    - Each writes to: ai-spec/{YYYY-MM-DD}-{description}/03-architecture-{approach}.md
    - Consolidate and provide recommendation
    - User selects preferred approach
  Verify:
    - 2-3 architecture files created (03-architecture-*.md)
    - Each includes concrete file paths and components
    - Tradeoffs clearly documented
    - User approval gate before Phase 5

- [ ] **Write Phase 5: Implementation Planning**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Phase 5 section)
  Do:
    - Based on selected architecture, create detailed implementation plan
    - Write to: ai-spec/{YYYY-MM-DD}-{description}/04-implementation-plan.md
    - Use Obra Superpowers-style task structure with Files/Do/Verify for each task
    - Each task is 2-5 minutes, bite-sized
    - Each verification step is executable (specific commands with expected output)
    - Group tasks into phases with dependency ordering
    - Include TDD approach (failing test → code → passing test)
    - Add sections: Testing Strategy, Rollback Strategy, Risk Register
    - User reviews and approves before Phase 6
  Verify:
    - 04-implementation-plan.md created
    - All tasks have Files/Do/Verify structure
    - Verification commands are copy-pasteable
    - Expected outputs specified
    - User approval gate before Phase 6

- [ ] **Write Phase 6: Plan Quality Review**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Phase 6 section)
  Do:
    - Launch 3 review agents in parallel focusing on:
      - Completeness (are all requirements addressed?)
      - Risks & Testability (what could go wrong? how to verify?)
      - Simplicity & Maintainability (is this over-engineered? DRY violations?)
    - Each reviews 04-implementation-plan.md and all feedback files
    - Issues categorized by severity: Critical, Important, Minor
    - Filter: only report issues with ≥80% confidence
    - Each writes to: ai-spec/{YYYY-MM-DD}-{description}/05-review-{focus}.md
    - Consolidate findings and ask user: fix now, fix later, or proceed?
  Verify:
    - 3 review files created (05-review-*.md)
    - Issues have severity levels and confidence scores
    - File:line references for each issue
    - User decision gate

- [ ] **Write Phase 7: Final Spec**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Phase 7 section)
  Do:
    - Read all artifacts: 01-feedback-*.md, 02-feedback-*.md, 03-architecture-*.md, 04-implementation-plan.md, 05-review-*.md
    - Synthesize into comprehensive spec.md
    - Write to: ai-spec/{YYYY-MM-DD}-{description}/spec.md
    - Include sections:
      - User Request & Context
      - Selected Architecture (with rationale)
      - Decision Log (disagreements resolved, alternatives rejected, why)
      - Risk Register (risks, severity, mitigation)
      - Implementation Plan (detailed tasks with executable verification)
      - Testing Strategy
      - Rollback Strategy
      - Open Questions & Future Considerations
    - All hard thinking done by this stage
    - Spec should be clear and easy to follow
  Verify:
    - spec.md created with all required sections
    - Decision log documents all key choices
    - Risk register includes mitigations
    - Implementation tasks are executable
    - No contradictions in final spec

### Phase 2: Update SPEC Directory Anatomy
**Goal**: Reflect new file structure in documentation

- [ ] **Update SPEC directory anatomy section** (SKILL.md:50-76)
  Files: plugins/kg/skills/sdd-plan/SKILL.md
  Do:
    - Replace example showing old file structure
    - Show new structure with:
      - 01-feedback-{role}.md (Phase 4a outputs)
      - 02-feedback-{role}.md (Phase 4b outputs)
      - 03-architecture-{approach}.md (Phase 4c outputs)
      - 04-implementation-plan.md (Phase 5 output)
      - 05-review-{focus}.md (Phase 6 outputs)
      - spec.md (Phase 7 output)
  Verify:
    - Example directory structure matches new workflow
    - All file naming conventions documented
    - Clear explanation of each file's purpose

### Phase 3: Create Feedback & Plan Templates
**Goal**: Standardize artifacts with explicit structure

- [ ] **Define 01-feedback-{role}.md template**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (new Templates section)
  Do:
    - Add template with required sections:
      ```markdown
      # {Role} - First Pass Architectural Feedback

      ## Summary
      [2-3 sentence perspective from this role]

      ## Risks Identified
      - [Risk 1]: [What could go wrong, severity, impact]

      ## Concrete Improvements
      - [file:line]: [Specific suggestion with rationale]

      ## Questions for Other Roles
      - [Cross-role concern requiring coordination]

      ## Confidence Level
      [High/Medium/Low for each major point above]
      ```
  Verify:
    - Template has all required sections
    - Includes examples in comments
    - Referenced in Phase 4a instructions

- [ ] **Define 02-feedback-{role}.md template**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (Templates section)
  Do:
    - Extend 01-feedback template with consensus-building sections:
      ```markdown
      # {Role} - Second Pass Architectural Feedback

      [All sections from 01-feedback template]

      ## Changes from First Pass
      [What changed after reading other feedback and why]

      ## Consensus Opportunities
      [Where agreement with other agents exists, positive-sum thinking]

      ## Unresolved Disagreements
      [Where consensus isn't possible, with rationale and recommendations]

      ## Positive-Sum Integrations
      [How other agents' feedback improved this perspective]
      ```
  Verify:
    - Extends 01-feedback template
    - Emphasizes cross-agent awareness
    - Referenced in Phase 4b instructions

- [ ] **Define 03-architecture-{approach}.md template**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (Templates section)
  Do:
    - Create template for concrete architectural proposals:
      ```markdown
      # Architecture Proposal: {Approach}

      ## Optimization Focus
      [Minimal changes | Clean architecture | Pragmatic balance]

      ## Component Architecture
      [Specific components, their responsibilities, interactions]

      ## Files to Create/Modify
      - path/to/file.ext: [Purpose]

      ## Build Sequence
      Phase 1: [Foundation components]
      Phase 2: [Integration]
      Phase 3: [Testing & validation]

      ## Tradeoffs
      **Pros**: [Benefits of this approach]
      **Cons**: [Costs and limitations]

      ## Rationale
      [Why this approach matches the optimization focus]
      ```
  Verify:
    - Template requires concrete specifics (file paths, components)
    - Tradeoffs section is structured
    - Referenced in Phase 4c instructions

- [ ] **Define 04-implementation-plan.md template**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (Templates section)
  Do:
    - Create Obra Superpowers-style task template:
      ```markdown
      # Implementation Plan

      ## Selected Architecture
      [Which approach from Phase 4c and why]

      ## Testing Strategy
      [Unit, integration, E2E test expectations]

      ## Rollback Strategy
      [Detection, rollback procedure, validation]

      ## Risk Register
      | Risk | Severity | Mitigation | Owner |
      |------|----------|------------|-------|

      ## Implementation Tasks

      ### Phase 1: [Phase Name]

      #### Task 1: [Task Name]
      **Files**: path/to/file1.ext, path/to/file2.ext

      **Do**:
      - Write failing test for [functionality]
      - Implement [specific component]
      - Verify test passes

      **Verify**:
      ```bash
      npm test -- path/to/test
      # Expected: ✓ test description
      # Expected: All tests passed
      ```
      ```
  Verify:
    - Every task has Files/Do/Verify structure
    - Verification commands are executable
    - Expected outputs specified
    - Referenced in Phase 5 instructions

- [ ] **Define spec.md template**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (Templates section)
  Do:
    - Create comprehensive final spec template combining all artifacts:
      ```markdown
      # Specification: {Feature Name}

      ## User Request & Context
      ## Selected Architecture
      ## Decision Log
      ## Risk Register
      ## Implementation Plan
      ## Testing Strategy
      ## Rollback Strategy
      ## Open Questions & Future Considerations
      ```
  Verify:
    - All key sections present
    - References all previous artifacts
    - Referenced in Phase 7 instructions

### Phase 4: Add Supporting Documentation
**Goal**: Provide guidance and examples

- [ ] **Add Decision Framework section**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (after Subagents section)
  Do:
    - Document principle hierarchy for conflict resolution
    - Default: Security > Maintainability > Performance > Optimization
    - Note: can be overridden in project CLAUDE.md
    - Explain when to prioritize different concerns
  Verify:
    - Clear hierarchy documented
    - Override mechanism explained
    - Referenced in Phase 4b and Phase 7

- [ ] **Add Best Practices section**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (near end, before "Remember" section)
  Do:
    - Document key practices from Obra Superpowers:
      - Executable verification (specific commands, expected outputs)
      - Bite-sized tasks (2-5 minutes each)
      - TDD-first (failing test → code → passing test)
      - DRY/YAGNI enforcement
      - Clear completion criteria
  Verify:
    - Each practice explained with example
    - Linked to relevant phases
    - Emphasizes practical application

- [ ] **Add Examples section**
  Files: plugins/kg/skills/sdd-plan/SKILL.md (after Best Practices)
  Do:
    - Provide real examples of:
      - Clarifying questions for different project types
      - Decision log entries showing disagreement resolution
      - Risk register entries with mitigations
      - Executable verification commands for common scenarios
  Verify:
    - At least 2-3 examples per category
    - Examples are realistic and helpful
    - Cover diverse project types

### Phase 5: Validation & Testing
**Goal**: Ensure new workflow works end-to-end

- [ ] **Create test SPEC using new workflow**
  Files: ai-spec/2026-01-21-test-hybrid-workflow/
  Do:
    - Use updated SKILL.md to plan a test feature
    - Task: "Add user authentication to a mock web application"
    - Execute all 7 phases
    - Generate all expected artifacts
  Verify:
    - All phases complete successfully
    - All expected files created:
      - 01-feedback-*.md (4-6 files)
      - 02-feedback-*.md (4-6 files)
      - 03-architecture-*.md (2-3 files)
      - 04-implementation-plan.md
      - 05-review-*.md (3 files)
      - spec.md
    - Templates followed correctly
    - No workflow blockers encountered

- [ ] **Review with DX and LLM engineers (subagents)**
  Files: ai-spec/2026-01-21-sdd-inspirations/06-validation-{role}.md
  Do:
    - Launch DX engineer and LLM engineer subagents
    - Each reviews the updated SKILL.md for:
      - Clarity and usability
      - AI-friendliness and prompt quality
      - Missing guidance or confusing sections
      - Potential workflow friction
    - Each writes feedback to validation file
  Verify:
    - 2 validation files created
    - Feedback addresses usability and AI experience
    - Actionable improvements identified

- [ ] **Address validation feedback**
  Files: plugins/kg/skills/sdd-plan/SKILL.md
  Do:
    - Review validation feedback from DX and LLM engineers
    - Implement high-priority improvements
    - Document any deferred improvements for future iterations
  Verify:
    - Critical feedback addressed
    - SKILL.md refined based on validation
    - No major usability issues remain

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
