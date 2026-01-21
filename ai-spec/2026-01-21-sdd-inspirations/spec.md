# Specification: Enhanced sdd-plan Skill with Hybrid 7-Phase Workflow

## Executive Summary

This specification describes improvements to the `sdd-plan` skill (plugins/kg/skills/sdd-plan/SKILL.md) based on analysis of three reference implementations: ralph-loop (iterative development), feature-dev (7-phase structured workflow), and Obra Superpowers (TDD-driven development system).

**Core Innovation**: Hybrid 7-phase workflow combining feature-dev's structural clarity with sdd-plan's sophisticated two-pass consensus building, enhanced with workflow modes (Express/Deep Dive) and progressive summarization to address context management and developer experience concerns.

**Key Outcome**: Specification-driven development workflow that scales from simple features (40-80 min, 3 agents) to complex architectural changes (90-180 min, 6 agents) while maintaining quality through multi-perspective review and positive-sum consensus building.

## User Request & Context

**Original Request**: Improve the `sdd-plan` skill by analyzing SDD-like reference implementations to identify strengths, weaknesses, and best practices applicable to planning workflows.

**Analysis Performed**:
- Ralph-loop: Iterative self-reference with safety mechanisms
- Feature-dev: 7-phase workflow with parallel agent exploration
- Obra Superpowers: TDD-first comprehensive development system with executable verification

**Perspectives Consulted**:
- LLM Engineer: Context management, agent orchestration, prompt engineering
- DX Engineer: Developer experience, workflow usability, cognitive load management

## Selected Architecture

### Hybrid 7-Phase Workflow with Adaptive Depth

**Foundation**: Feature-dev's 7-phase structure enhanced with sdd-plan's two-pass consensus in Phase 4.

**Workflow Modes**:
- **Express Mode**: 5 phases, 3 agents, 40-80 minutes, ~25k tokens
- **Deep Dive Mode**: 7 phases, 4-6 agents, 90-180 minutes, ~60k tokens

**Phase Structure**:

1. **Discovery** (5-10 min)
   - Clarify requirements through direct engagement
   - Ultrathink as architect and planner
   - User approval before proceeding

2. **Codebase Exploration** (10-20 min)
   - Launch 2-3 Haiku agents in parallel exploring different aspects
   - Return 5-10 key files with file:line references
   - Human reads identified files for deep context

3. **Clarifying Questions** (5-15 min)
   - Generate 5-10 questions in categories (edge cases, integration, performance, compatibility)
   - CRITICAL: Block until questions answered
   - Allow partial answers with deferred tracking

4. **Architecture Design** - THREE-PASS (Express: single pass, Deep Dive: two-pass + optional third)
   - **Phase 4a** (15-25 min): 4-6 specialized agents provide independent architectural feedback (Deep Dive only)
   - **Phase 4b** (15-25 min): Same agents apply positive-sum thinking after reading all first-pass feedback (Deep Dive only)
   - **Phase 4c** (10-20 min): OPTIONAL - User-triggered concrete alternatives from 2-3 architects with different tradeoff focuses
   - **Express Mode**: Single-pass with 3 agents providing both broad feedback AND concrete recommendations

5. **Implementation Planning** (15-30 min)
   - Based on selected architecture, create detailed plan
   - Files/Do/Verify structure for each task (Obra Superpowers pattern)
   - Executable verification commands with full context
   - User approval before quality review

6. **Plan Quality Review** (10-15 min) - Deep Dive only
   - 3 review agents in parallel: Completeness, Risks & Testability, Simplicity & Maintainability
   - Severity levels: Critical, Important, Minor
   - Confidence filtering (≥80%)
   - User decision: fix now, fix later, proceed

7. **Final Spec** (10-15 min)
   - Synthesize all artifacts into comprehensive spec.md
   - Include: User Request, Selected Architecture, Decision Log, Risk Register, Implementation Plan, Testing Strategy, Rollback Strategy

**Progressive Summarization**:
- After Phase 4b: Generate 2000-token summary capturing consensus/disagreements
- Later phases read summary first, full artifacts on-demand
- Reduces Phase 7 context by 60-70%

**Context Budget**:
- Maximum 2000 tokens per feedback file
- Maximum 60k tokens for Phase 7 synthesis
- Auto-summarization if approaching limits

## Decision Log

### Critical Decisions

#### Decision 1: Hybrid 7-Phase Structure vs. Minimal Enhancement
**Options**:
- Alternative 1: Keep current 8-stage workflow, add incremental improvements
- Alternative 2: Adopt hybrid 7-phase with workflow modes (SELECTED)
- Alternative 3: Modular skill decomposition

**Decision**: Alternative 2 (Hybrid 7-Phase)

**Rationale**:
- **LLM Concern**: Current workflow generates 40k-80k tokens before Phase 7 synthesis, causing context exhaustion
- **DX Concern**: 4-5 approval gates with unclear workflow structure cause abandonment
- **Resolution**: 7-phase structure provides clarity; workflow modes prevent bloat
- **Positive-Sum**: Solves both context management (technical) and time-to-value (UX) simultaneously

#### Decision 2: Phase 4c Default vs. Optional
**Options**:
- Keep three-pass (4a/4b/4c) as default
- Make Phase 4c optional and user-triggered (SELECTED)
- Eliminate Phase 4c entirely

**Decision**: Optional, user-triggered

**Rationale**:
- **LLM Analysis**: Phase 4c generates 6-9 additional artifacts that often duplicate 4a/4b thinking
- **DX Concern**: Three-pass Phase 4 represents 30-50% of total workflow time
- **Resolution**: Phase 4b includes "Recommended Concrete Approach"; 4c only for significant disagreements
- **Impact**: Reduces default path by 25% while preserving depth option

#### Decision 3: Template Enforcement Philosophy
**Options**:
- Strict enforcement with all sections required
- Flexible templates with "relevant sections only" (SELECTED)
- No templates (freeform)

**Decision**: Flexible with minimum structure

**Rationale**:
- **LLM Concern**: Strict templates force boilerplate, waste 20-30% of tokens on N/A sections
- **DX Concern**: Rigid structure creates "compliance theater" without value
- **Resolution**: Require only Summary (2-3 sentences); all other sections optional if relevant
- **Quality Gate**: Automated validation of critical sections (Summary, Verify commands)

#### Decision 4: Workflow Modes Strategy
**Options**:
- Single mandatory workflow (Deep Dive for all)
- Workflow modes based on complexity (SELECTED)
- Always use simplest (Express only)

**Decision**: Complexity-based modes (Express/Deep Dive)

**Rationale**:
- **LLM Analysis**: Simple features waste 60-70% of tokens on unnecessary agent feedback
- **DX Concern**: 90-175 min workflow for simple features causes process fatigue
- **Resolution**: Express (3 agents, ~25k tokens) for simple; Deep Dive (4-6 agents, ~60k) for complex
- **Impact**: 60% time savings and 70% token reduction on simple features

### Important Decisions

#### Decision 5: Progressive Summarization Timing
**Options**:
- No summarization (read full artifacts)
- Summarize after Phase 4b only (SELECTED)
- Summarize after both Phase 4b and Phase 6

**Decision**: Single summarization point after Phase 4b

**Rationale**:
- **LLM Analysis**: Phase 4 generates 12+ feedback files (40k+ tokens); later phases can't handle this
- **DX Benefit**: 2000-token summary readable by humans for checkpoint review
- **Implementation**: Generate {phase4-summary.md} with consensus points, disagreements, architectural direction
- **Future**: Monitor if second summarization point needed after Phase 6

#### Decision 6: Validation Checkpoint Strictness
**Options**:
- Block on all validation failures
- Warn and allow override for all failures
- Tiered approach by severity (SELECTED)

**Decision**: Tiered (Critical blocks, Important/Minor warn)

**Rationale**:
- **LLM Concern**: Lax validation creates inconsistent artifacts, degrades synthesis quality
- **DX Concern**: Blocking creates frustration, may encourage workarounds
- **Resolution**: Critical failures (e.g., missing Verify commands) block; others warn
- **Examples**: Critical = no executable verification; Important = missing confidence levels; Minor = short summary

#### Decision 7: Context Budget Thresholds
**Initial Values**: 2000 tokens/file, 60k total for Phase 7

**Rationale**:
- **LLM Proposal**: Based on production experience with multi-stage workflows
- **DX Concern**: Need visibility and predictability, not just enforcement
- **Resolution**: Start conservative, instrument actual usage, tune based on data (95th percentile fit)
- **Extensibility**: Per-project overrides in CLAUDE.md for special cases

### Resolved Disagreements

#### Disagreement 1: Async-Friendly Clarifying Questions
**LLM Position**: Phase 3 should support partial answers with deferred tracking
**DX Position**: Blocking creates workflow interruptions; support CRITICAL/IMPORTANT/NICE-TO-HAVE classification

**Resolution**: DEFERRED to post-MVP
- **Reason**: Progressive summarization and workflow modes address higher-priority concerns
- **Future**: Revisit if Phase 3 becomes common blocking point
- **Current Approach**: Keep Phase 3 blocking, time-box to 5-10 questions maximum

#### Disagreement 2: Template Minimum Structure
**LLM Position**: "Relevant sections only" with no minimums for maximum token efficiency
**DX Position**: Need minimum structure (Summary + one improvement) for consistency

**Resolution**: COMPROMISE - Summary required only
- **Rationale**: Summary ensures baseline communication; everything else context-dependent
- **Quality Gate**: If >30% of feedback files have only Summary in practice, revisit requirements
- **Validation**: Agents can skip irrelevant sections without penalty

## Risk Register

| Risk | Severity | Mitigation | Owner |
|------|----------|------------|-------|
| **Context window exhaustion in Phase 7 synthesis** | Critical | Progressive summarization (2000-token summary after Phase 4b); context budget limits (60k max); auto-summarization trigger at 50k | LLM Engineer |
| **Developer abandonment before completion** | Critical | Workflow modes (Express 40-80 min vs Deep Dive 90-180 min); checkpoint dashboard consolidating approval gates; phase duration estimates | DX Engineer |
| **Phase 4c optional triggers too rarely** | High | Monitor usage; if <10% of Deep Dive workflows trigger 4c but >30% have unresolved disagreements, reconsider default | Architect |
| **Template flexibility degrades consistency** | High | Automated validation of critical sections (Summary, Verify commands); manual review after 20-30 workflows; adjust if signal-to-noise degrades | LLM + DX |
| **Complexity heuristics misclassify features** | High | User can override recommendations; track override rate; if >40%, refine heuristics | DX Engineer |
| **Parallel agent coordination failures** | Medium | Orchestration layer with .workflow-status.json; timeout handling (10 min); partial progress with user approval | LLM Engineer |
| **Context budget too strict** | Medium | Instrument actual token usage; tune thresholds based on 95th percentile; allow per-project overrides | LLM Engineer |
| **Workflow state loss on interruption** | Medium | .sdd-state.json for resume capability; ABORTED.md for clean bailout; checkpoint dashboard for context restoration | DX Engineer |
| **Verification commands not actually executable** | Medium | 5-component template (directory, command, success output, failure output, stability note); Phase 6 QA review validates executability | DX + QA |
| **User fatigue from validation warnings** | Low | Tiered severity (Critical blocks only); consolidate warnings in checkpoint dashboard; avoid warning overload | DX Engineer |

## Implementation Plan

### Phase 1: Core Workflow Restructuring (Critical Priority)

**Goal**: Replace 8-stage workflow with hybrid 7-phase structure, implement workflow modes

#### Task 1: Rewrite Main Workflow Section
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (lines 79-204)

**Do**:
- Replace "The SDD planning process" section with new 7-phase workflow
- Keep PLANNING-ONLY MODE constraint and SPEC directory anatomy unchanged
- Add Phase 0: Workflow Mode Selection before Phase 1

**Verify**:
```bash
cd /Users/konrad/src/kgadek-claude-plugin
grep -A 5 "## The SDD planning process" plugins/kg/skills/sdd-plan/SKILL.md | grep "Phase 1: Discovery"
# Expected: "Phase 1: Discovery" appears in workflow section
grep "Phase 0: Workflow Mode Selection" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: Phase 0 section exists
wc -l plugins/kg/skills/sdd-plan/SKILL.md
# Expected: File length 800-1200 lines (increased from current ~230)
```

#### Task 2: Write Phase 0 (Workflow Mode Selection)
**Files**: plugins/kg/skills/sdd-plan/SKILL.md

**Do**:
- Document complexity analysis criteria (files impacted, architectural novelty, cross-team coordination, time estimate)
- Define Express Mode (3 agents, ~25k tokens) vs Deep Dive Mode (4-6 agents, ~60k tokens)
- Specify user override mechanism with rationale capture

**Verify**:
```bash
grep -A 20 "Phase 0: Workflow Mode Selection" plugins/kg/skills/sdd-plan/SKILL.md | grep "Express Mode"
# Expected: Express Mode definition appears
grep -A 20 "Phase 0: Workflow Mode Selection" plugins/kg/skills/sdd-plan/SKILL.md | grep "Deep Dive Mode"
# Expected: Deep Dive Mode definition appears
```

#### Task 3: Write Phase 1 (Discovery)
**Files**: plugins/kg/skills/sdd-plan/SKILL.md

**Do**:
- Direct user engagement with clarification
- Ultrathink as architect and planner
- Use AskUserQuestion for ambiguity resolution
- Approval gate before Phase 2

**Verify**:
```bash
grep -A 15 "Phase 1: Discovery" plugins/kg/skills/sdd-plan/SKILL.md | grep "AskUserQuestion"
# Expected: Tool mentioned for clarification
grep -A 15 "Phase 1: Discovery" plugins/kg/skills/sdd-plan/SKILL.md | grep "approval"
# Expected: Approval gate documented
```

#### Task 4: Write Phase 2 (Codebase Exploration)
**Files**: plugins/kg/skills/sdd-plan/SKILL.md

**Do**:
- Launch 2-3 Haiku agents in parallel with Task tool
- Each explores: existing SPECs, similar features, architecture patterns
- Return 5-10 key files per agent
- Human reads for deep context

**Verify**:
```bash
grep -A 20 "Phase 2: Codebase Exploration" plugins/kg/skills/sdd-plan/SKILL.md | grep "model=haiku"
# Expected: Haiku model specified
grep -A 20 "Phase 2: Codebase Exploration" plugins/kg/skills/sdd-plan/SKILL.md | grep "parallel"
# Expected: Parallel execution mentioned
```

#### Task 5: Write Phase 3 (Clarifying Questions)
**Files**: plugins/kg/skills/sdd-plan/SKILL.md

**Do**:
- Generate 5-10 questions in categories (edge cases, integration, performance, compatibility, design)
- Use AskUserQuestion tool
- CRITICAL: Block until answered
- Time-box to 5-10 questions maximum

**Verify**:
```bash
grep -A 20 "Phase 3: Clarifying Questions" plugins/kg/skills/sdd-plan/SKILL.md | grep "CRITICAL"
# Expected: Blocking requirement marked as CRITICAL
grep -A 20 "Phase 3: Clarifying Questions" plugins/kg/skills/sdd-plan/SKILL.md | grep "5-10 questions"
# Expected: Question limit documented
```

#### Task 6: Write Phase 4 (Architecture Design) - Three Variants
**Files**: plugins/kg/skills/sdd-plan/SKILL.md

**Do**:
- **Express Mode**: Single-pass (3 agents combine broad + concrete feedback)
- **Deep Dive Phase 4a**: First consensus (4-6 agents independent)
- **Deep Dive Phase 4b**: Second consensus (same agents, cross-awareness, positive-sum thinking)
- **Deep Dive Phase 4c**: OPTIONAL user-triggered alternatives (2-3 architects with different focuses)
- Document when to trigger Phase 4c (significant unresolved disagreements)

**Verify**:
```bash
grep -A 50 "Phase 4: Architecture Design" plugins/kg/skills/sdd-plan/SKILL.md | grep "Express Mode"
# Expected: Express variant documented
grep -A 50 "Phase 4: Architecture Design" plugins/kg/skills/sdd-plan/SKILL.md | grep "Phase 4a"
# Expected: Deep Dive first pass documented
grep -A 50 "Phase 4: Architecture Design" plugins/kg/skills/sdd-plan/SKILL.md | grep "OPTIONAL"
# Expected: Phase 4c marked as optional
```

#### Task 7: Write Phase 5 (Implementation Planning)
**Files**: plugins/kg/skills/sdd-plan/SKILL.md

**Do**:
- Create 04-implementation-plan.md with Files/Do/Verify structure for each task
- Each task: 2-5 minutes, bite-sized
- Executable verification (working directory, command, expected success/failure output, stability note)
- TDD approach (failing test → code → passing test)
- Include Testing Strategy, Rollback Strategy, Risk Register sections
- User approval before Phase 6

**Verify**:
```bash
grep -A 30 "Phase 5: Implementation Planning" plugins/kg/skills/sdd-plan/SKILL.md | grep "Files/Do/Verify"
# Expected: Task structure documented
grep -A 30 "Phase 5: Implementation Planning" plugins/kg/skills/sdd-plan/SKILL.md | grep "2-5 minutes"
# Expected: Task size guidance present
```

#### Task 8: Write Phase 6 (Plan Quality Review)
**Files**: plugins/kg/skills/sdd-plan/SKILL.md

**Do**:
- Deep Dive only (Express skips to Phase 7)
- Launch 3 agents: Completeness, Risks & Testability, Simplicity & Maintainability
- Severity levels: Critical, Important, Minor
- Confidence threshold: ≥80%
- Output: 05-review-{focus}.md
- User decision gate

**Verify**:
```bash
grep -A 25 "Phase 6: Plan Quality Review" plugins/kg/skills/sdd-plan/SKILL.md | grep "Deep Dive only"
# Expected: Mode restriction documented
grep -A 25 "Phase 6: Plan Quality Review" plugins/kg/skills/sdd-plan/SKILL.md | grep "80%"
# Expected: Confidence threshold present
```

#### Task 9: Write Phase 7 (Final Spec)
**Files**: plugins/kg/skills/sdd-plan/SKILL.md

**Do**:
- Read all artifacts (or summaries if available)
- Synthesize into spec.md
- Include: User Request, Selected Architecture, Decision Log, Risk Register, Implementation Plan, Testing Strategy, Rollback Strategy, Open Questions
- Document all disagreement resolutions

**Verify**:
```bash
grep -A 25 "Phase 7: Final Spec" plugins/kg/skills/sdd-plan/SKILL.md | grep "Decision Log"
# Expected: Decision Log section required
grep -A 25 "Phase 7: Final Spec" plugins/kg/skills/sdd-plan/SKILL.md | grep "spec.md"
# Expected: Output file documented
```

### Phase 2: Progressive Summarization & Context Management (Critical Priority)

**Goal**: Implement context budget system and progressive summarization

#### Task 10: Add Progressive Summarization to Phase 4b
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (Phase 4b section)

**Do**:
- After Phase 4b completes, generate 04-phase4-summary.md (max 2000 tokens)
- Include: key consensus points, unresolved disagreements with options, architectural direction, pointers to full artifacts
- Instruct later phases to read summary first, full artifacts on-demand

**Verify**:
```bash
grep -A 30 "Phase 4b" plugins/kg/skills/sdd-plan/SKILL.md | grep "04-phase4-summary.md"
# Expected: Summary file documented
grep -A 30 "Phase 4b" plugins/kg/skills/sdd-plan/SKILL.md | grep "2000 tokens"
# Expected: Token limit specified
```

#### Task 11: Define Context Budget Rules
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (new "Context Budget Management" section after workflow)

**Do**:
- Document limits: 2000 tokens/feedback file, 60k total for Phase 7
- Auto-summarization trigger: if cumulative context exceeds 50k before Phase 7, generate compressed summary
- User warning mechanism if budget will be exceeded
- Visibility: token consumption in checkpoint dashboard

**Verify**:
```bash
grep "Context Budget Management" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: Section exists
grep "2000 tokens" plugins/kg/skills/sdd-plan/SKILL.md | wc -l
# Expected: Multiple mentions (per-file limit appears in templates)
grep "60,000 tokens\|60k tokens" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: Total limit documented
```

### Phase 3: Templates & Validation (High Priority)

**Goal**: Create flexible templates with automated validation

#### Task 12: Define Feedback File Templates
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (new "Templates" section)

**Do**:
- **01-feedback-{role}.md**: Summary (required), Risks, Improvements, Questions, Confidence (optional if relevant)
- **02-feedback-{role}.md**: Extend 01 template + Changes from First Pass, Consensus Opportunities, Unresolved Disagreements, Positive-Sum Integrations
- **03-architecture-{approach}.md**: Optimization Focus, Component Architecture, Files to Create/Modify, Build Sequence, Tradeoffs, Rationale
- **04-implementation-plan.md**: Selected Architecture, Testing Strategy, Rollback Strategy, Risk Register, Implementation Tasks (Files/Do/Verify per task)
- **05-review-{focus}.md**: Focus area, Issues (with severity/confidence), Recommendations
- **spec.md**: User Request, Selected Architecture, Decision Log, Risk Register, Implementation Plan, Testing Strategy, Rollback Strategy, Open Questions
- All templates: "Include sections WHERE YOU HAVE SUBSTANTIVE FEEDBACK. Skip sections with nothing valuable to add."

**Verify**:
```bash
grep -A 50 "## Templates" plugins/kg/skills/sdd-plan/SKILL.md | grep "01-feedback"
# Expected: First-pass template documented
grep -A 50 "## Templates" plugins/kg/skills/sdd-plan/SKILL.md | grep "SUBSTANTIVE FEEDBACK"
# Expected: Flexibility instruction present
```

#### Task 13: Define Enhanced Verification Command Template
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (Templates section, 04-implementation-plan subsection)

**Do**:
- 5 required components for Verify blocks:
  1. Working Directory (absolute path)
  2. Verification Command (full command with args)
  3. Expected Success Output (exact string or regex)
  4. Expected Failure Output (to detect issues)
  5. Stability Note (Low/Medium/High flakiness risk, dependencies, runtime)
- Optional: Alternative Verification for flaky tests

**Verify**:
```bash
grep -A 40 "Verification Command Template\|Verify" plugins/kg/skills/sdd-plan/SKILL.md | grep "Working Directory"
# Expected: Component 1 documented
grep -A 40 "Verification Command Template\|Verify" plugins/kg/skills/sdd-plan/SKILL.md | grep "Stability Note"
# Expected: Component 5 documented
```

#### Task 14: Add Validation Checkpoints Section
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (new "Validation Checkpoints" section)

**Do**:
- Define tiered severity: Critical (blocks), Important (warns), Minor (info)
- Phase 4a: Verify >500 tokens and ≥1 substantive section per feedback (WARN)
- Phase 4b: Verify explicit consensus/disagreement documentation (WARN)
- Phase 5: Verify all tasks have Files/Do/Verify with 5 components (BLOCK if missing)
- Display validation report with severity levels
- User override for Important/Minor, not Critical

**Verify**:
```bash
grep "Validation Checkpoints" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: Section exists
grep -A 20 "Validation Checkpoints" plugins/kg/skills/sdd-plan/SKILL.md | grep "BLOCK\|blocks"
# Expected: Blocking behavior documented for critical failures
```

### Phase 4: Supporting Infrastructure (Medium Priority)

**Goal**: Add checkpoint dashboard, orchestration, abort/resume

#### Task 15: Add Checkpoint Dashboard Specification
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (new "Checkpoint Dashboard" section)

**Do**:
- Living document: checkpoints.md accumulating all approval requests
- Structure: Checkpoint N → Status (✓/⏳/❌) → User Decision → Rationale → Timestamp
- Include token consumption metrics per phase
- Updated after each phase
- Serves both UX (progress) and LLM (decision log for synthesis)

**Verify**:
```bash
grep "Checkpoint Dashboard\|checkpoints.md" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: Dashboard documented
grep -A 15 "Checkpoint Dashboard" plugins/kg/skills/sdd-plan/SKILL.md | grep "token"
# Expected: Token metrics mentioned
```

#### Task 16: Add Orchestration Layer Specification
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (new "Orchestration" section)

**Do**:
- Before parallel phases (4a/4b/6), create .workflow-status.json with expected agent count, completion status per agent, timestamp
- Timeout: 10 minutes per parallel phase
- On timeout: notify "X/N agents completed. Proceed with partial results or retry failed agents?"
- Atomic file writes with temp file + mv pattern

**Verify**:
```bash
grep "Orchestration\|.workflow-status.json" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: Orchestration documented
grep -A 15 "Orchestration" plugins/kg/skills/sdd-plan/SKILL.md | grep "10 minutes\|timeout"
# Expected: Timeout specified
```

#### Task 17: Add Abort/Resume Mechanism Specification
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (new "State Persistence" section)

**Do**:
- .sdd-state.json tracking: currentPhase, workflowMode, agentProgress, checkpoints, timestamps
- Resume command when returning to SPEC directory
- ABORTED.md marker with abort reason, lessons learned, restart guidance
- Document abort scenarios: scope creep, technical infeasibility, requirements misunderstood, resource constraints

**Verify**:
```bash
grep "State Persistence\|.sdd-state.json\|ABORTED.md" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: State mechanism documented
grep -A 20 "State Persistence" plugins/kg/skills/sdd-plan/SKILL.md | grep "Resume"
# Expected: Resume capability mentioned
```

### Phase 5: Documentation & Guidance (Medium Priority)

**Goal**: Provide usage guidance, examples, best practices

#### Task 18: Update SPEC Directory Anatomy
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (lines 50-76)

**Do**:
- Replace example showing new file structure:
  ```
  ai-spec/YYYY-MM-DD-{description}/
  ├── checkpoints.md (living decision log)
  ├── .sdd-state.json (workflow state)
  ├── .workflow-status.json (parallel agent tracking)
  ├── 01-feedback-{role}.md (Phase 4a)
  ├── 02-feedback-{role}.md (Phase 4b)
  ├── 03-architecture-{approach}.md (Phase 4c - optional)
  ├── 04-phase4-summary.md (progressive summarization)
  ├── 04-implementation-plan.md (Phase 5)
  ├── 05-review-{focus}.md (Phase 6)
  └── spec.md (Phase 7 final)
  ```

**Verify**:
```bash
grep -A 25 "SPEC directory anatomy\|Directory Structure" plugins/kg/skills/sdd-plan/SKILL.md | grep "checkpoints.md"
# Expected: New files in example
grep -A 25 "SPEC directory anatomy\|Directory Structure" plugins/kg/skills/sdd-plan/SKILL.md | grep "04-phase4-summary.md"
# Expected: Summary file in example
```

#### Task 19: Add Decision Framework Section
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (after Subagents section)

**Do**:
- Document principle hierarchy: Security > Maintainability > Performance > Optimization
- Note: can be overridden in project CLAUDE.md
- Explain when to prioritize different concerns
- Reference in Phase 4b and Phase 7 for disagreement resolution

**Verify**:
```bash
grep "Decision Framework" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: Section exists
grep -A 10 "Decision Framework" plugins/kg/skills/sdd-plan/SKILL.md | grep "Security > Maintainability"
# Expected: Hierarchy documented
```

#### Task 20: Add Best Practices Section
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (near end, before "Remember")

**Do**:
- Executable verification (5-component template, copy-pasteable)
- Bite-sized tasks (2-5 minutes each)
- TDD-first (failing test → code → passing test)
- DRY/YAGNI enforcement
- Clear completion criteria
- Progressive disclosure (summaries before details)

**Verify**:
```bash
grep "Best Practices" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: Section exists
grep -A 20 "Best Practices" plugins/kg/skills/sdd-plan/SKILL.md | grep "TDD\|Test-Driven"
# Expected: TDD mentioned
```

#### Task 21: Add Examples Section
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (after Best Practices)

**Do**:
- Example clarifying questions for: backend API feature, frontend UI feature, infrastructure change
- Example decision log entries: architecture selection, alternative rejection, disagreement resolution
- Example risk register: with risks, severity, mitigations
- Example executable verification: backend test, frontend test, integration test

**Verify**:
```bash
grep "Examples" plugins/kg/skills/sdd-plan/SKILL.md | tail -1
# Expected: Examples section near end
grep -A 50 "## Examples" plugins/kg/skills/sdd-plan/SKILL.md | grep "decision log\|Decision Log"
# Expected: Decision log example present
```

#### Task 22: Add Phase Duration Estimates Section
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (in workflow description or Best Practices)

**Do**:
- **Express Mode**: Total 40-80 minutes
  - Phase 0: 2-5 min, Phase 1: 5-10 min, Phase 2: 10-20 min
  - Phase 3: 5-10 min, Phase 4: 15-25 min (single-pass, 3 agents)
  - Phase 5: 15-30 min, Phase 7: 10-15 min
- **Deep Dive Mode**: Total 90-180 minutes (without Phase 4c), 105-200 min (with 4c)
  - Phase 0: 2-5 min, Phase 1: 5-10 min, Phase 2: 10-20 min
  - Phase 3: 5-15 min, Phase 4a: 15-25 min, Phase 4b: 15-25 min, Phase 4c: 10-20 min (optional)
  - Phase 5: 15-30 min, Phase 6: 10-15 min, Phase 7: 10-15 min

**Verify**:
```bash
grep -A 30 "Phase Duration\|Duration Estimates" plugins/kg/skills/sdd-plan/SKILL.md | grep "40-80 minutes\|Express Mode.*minutes"
# Expected: Express timing documented
grep -A 30 "Phase Duration\|Duration Estimates" plugins/kg/skills/sdd-plan/SKILL.md | grep "90-180 minutes\|Deep Dive.*minutes"
# Expected: Deep Dive timing documented
```

#### Task 23: Add "When to Use This Workflow" Guidance
**Files**: plugins/kg/skills/sdd-plan/SKILL.md (introduction or Phase 0)

**Do**:
- **Use sdd-plan when**:
  - Feature impacts >3 files
  - Estimated implementation >4 hours
  - Introduces new architectural patterns
  - Requires cross-team coordination
  - Has significant operational/security impact
- **Don't use sdd-plan when**:
  - Simple bug fixes (1-2 files)
  - Trivial feature additions (<2 hours)
  - Well-understood, repetitive changes
  - Prototyping or experimentation

**Verify**:
```bash
grep "When to Use\|When NOT to use" plugins/kg/skills/sdd-plan/SKILL.md
# Expected: Usage criteria section exists
grep -A 15 "When to Use" plugins/kg/skills/sdd-plan/SKILL.md | grep ">3 files\|>4 hours"
# Expected: Thresholds documented
```

### Phase 6: Testing & Validation

**Goal**: Ensure new workflow functions correctly

#### Task 24: Create Test SPEC Directory
**Files**: ai-spec/2026-01-21-test-hybrid-workflow/

**Do**:
- Use updated SKILL.md to plan test feature: "Add user authentication to a mock web application"
- Execute workflow in Deep Dive mode
- Validate all expected files created per phase
- Check template compliance and context budget adherence

**Verify**:
```bash
cd /Users/konrad/src/kgadek-claude-plugin
ls ai-spec/2026-01-21-test-hybrid-workflow/
# Expected: Directory exists
ls ai-spec/2026-01-21-test-hybrid-workflow/ | grep "spec.md"
# Expected: spec.md created (workflow completed to Phase 7)
ls ai-spec/2026-01-21-test-hybrid-workflow/ | grep "checkpoints.md"
# Expected: checkpoint dashboard created
```

#### Task 25: Validate Backward Compatibility
**Files**: Review existing SPEC directories

**Do**:
- Review ai-spec/2026-01-21-sdd-inspirations/ (this SPEC)
- Confirm old naming conventions still readable
- Ensure no breaking changes to existing references

**Verify**:
```bash
ls ai-spec/2026-01-21-sdd-inspirations/ | grep "00-initial-plan.md\|01-feedback\|02-feedback\|spec.md"
# Expected: Current SPEC uses compatible structure
# Note: Old 00-initial-plan.md is compatible, new starts with Phase 0
```

## Testing Strategy

### Unit-Level Testing
- **Template Validation**: Each template section renders correctly with example data
- **Verification Command Format**: 5-component template enforced in validation
- **Context Budget Calculation**: Token counting logic accurate within 5% margin

### Integration Testing
- **End-to-End Express Mode**: Simple feature completes in 40-80 minutes with ~25k tokens
- **End-to-End Deep Dive Mode**: Complex feature completes in 90-180 minutes with ~60k tokens
- **Progressive Summarization**: Phase 4b summary captures consensus/disagreements in <2000 tokens
- **Checkpoint Dashboard**: All approval gates logged with status/decision/rationale

### Workflow Testing Scenarios
1. **Simple Backend API** (Express Mode):
   - 2 files modified, no new patterns, ~3 hour estimate
   - Expected: 3 agents, 40-60 min total, <30k tokens

2. **Complex Frontend Feature** (Deep Dive Mode):
   - 8 files modified, new state management pattern, cross-team coordination
   - Expected: 5-6 agents, 120-150 min total, 50-60k tokens

3. **Workflow Interruption** (Abort/Resume):
   - Start Deep Dive, abort at Phase 4a
   - Resume from .sdd-state.json
   - Expected: Restore context from checkpoints.md, continue without loss

### Validation Criteria
- **Context Management**: 95% of workflows stay under 60k token budget
- **Completion Rate**: >80% of started workflows reach Phase 7
- **Mode Distribution**: Express used 50-70% of time (validates simplicity bias)
- **Phase 4c Trigger Rate**: <30% of Deep Dive workflows trigger optional 4c
- **Template Compliance**: >90% of feedback files include Summary
- **Verification Executability**: >85% of Verify commands succeed when copy-pasted

## Rollback Strategy

### Detection
- **Context Budget Exceeded**: >10% of workflows hit 60k limit
- **Low Completion Rate**: <70% of workflows reach Phase 7
- **User Complaints**: Workflow feels too heavy or confusing
- **Template Drift**: <80% of feedback files follow minimum structure

### Rollback Procedure
1. **Immediate**: Revert plugins/kg/skills/sdd-plan/SKILL.md to previous version
2. **Preserve Data**: Existing SPEC directories remain valid (backward compatible)
3. **Document Issues**: Create issue in ai-spec/2026-01-21-sdd-retrospective.md with specific problems encountered
4. **Staged Rollback Options**:
   - Rollback workflow modes only (revert to single Deep Dive mode)
   - Rollback Phase 4c optional (make it default again)
   - Rollback progressive summarization (Phase 7 reads full artifacts)

### Validation After Rollback
- Run 2-3 workflows with reverted version
- Confirm completion rate improves or issues resolved
- Decide: full rollback permanent, or identify specific component to fix

## Open Questions & Future Considerations

### Questions Requiring Further Validation

1. **Progressive Summarization Timing**:
   - Current: Single summarization after Phase 4b
   - Question: Should we also summarize after Phase 6 (review feedback) before Phase 7?
   - Test criterion: Monitor Phase 7 context consumption; if >50k tokens common, add second summarization point

2. **Context Budget Thresholds**:
   - Current: 2000 tokens/file, 60k total
   - Question: Are these optimal or too strict/loose?
   - Test criterion: After 20-30 workflows, analyze 95th percentile; adjust if >5% exceed limits

3. **Workflow Mode Switching Mid-Flight**:
   - Current: No support for upgrading Express → Deep Dive during workflow
   - Question: Should users be able to "escalate" if complexity discovered?
   - Test criterion: Track abort rate after Phase 4 in Express mode; if >20% restart as Deep Dive, add escalation

4. **Incremental Consensus Alternative**:
   - Current: Two-pass consensus (4a independent, 4b cross-aware)
   - Question: Would live facilitator agent improve efficiency?
   - Test criterion: Prototype and compare token usage and consensus quality; proceed if >20% efficiency gain

5. **Template Minimum Structure**:
   - Current: Summary required only, all else optional
   - Question: Is this too flexible or just right?
   - Test criterion: If >30% of feedback files have only Summary after 20 workflows, add requirements

6. **Context-Aware Agent Selection Heuristics**:
   - Current: Simple rules (files, novelty, coordination, time)
   - Question: Could ML-based classification improve accuracy?
   - Test criterion: If user override rate >40%, explore ML classification

### Future Enhancements to Consider

1. **Multi-Point Summarization**: Add second summary after Phase 6 for very complex projects
2. **Solo Developer Mode**: Skip consensus for single-developer projects (requires validation)
3. **Workflow Metrics Dashboard**: Real-time visualization of token consumption, phase progress, completion estimates
4. **AI-Assisted Decision Resolution**: Mediator agent for disagreement synthesis (low confidence, needs research)
5. **Per-Project Workflow Customization**: CLAUDE.md overrides for mode defaults, phase skipping, agent selection
6. **Automated Complexity Classification**: Replace heuristics with learned model based on historical workflow data
7. **Traceability Matrix**: Link spec requirements to implementation tasks (heavy lift, unclear ROI)
8. **Specification Versioning**: Amendment process for evolving specs post-implementation

### Success Metrics to Track

**Adoption Metrics**:
- Workflow initiation rate (how often sdd-plan invoked)
- Workflow completion rate (started → spec.md created)
- Mode distribution (Express vs Deep Dive usage)
- User override rate (recommendations rejected)

**Efficiency Metrics**:
- Average workflow duration (Express: 40-80 min target, Deep Dive: 90-180 min target)
- Token consumption per workflow (Express: <40k target, Deep Dive: <70k target)
- Phase 4c trigger rate (target <30% of Deep Dive workflows)
- Context budget violation rate (target <5%)

**Quality Metrics**:
- Template compliance rate (target >90% have Summary)
- Verification executability rate (target >85% copy-pasteable)
- Disagreement resolution rate (unresolved disagreements in spec.md)
- Spec completeness (all sections present, decision log captures rationale)

**User Experience Metrics**:
- Checkpoint dashboard usage (how often users reference)
- Abort rate by phase (identify friction points)
- Resume rate (workflows interrupted and continued)
- Time-to-first-value (Phase 1 → Phase 4b decision)

---

## Implementation Notes

**Priority Order**:
1. **Phase 1**: Core workflow restructuring (enables all other improvements)
2. **Phase 2**: Progressive summarization & context management (critical for scalability)
3. **Phase 3**: Templates & validation (enables quality enforcement)
4. **Phase 4**: Supporting infrastructure (reliability and UX)
5. **Phase 5**: Documentation & guidance (user adoption)
6. **Phase 6**: Testing & validation (ensures correctness)

**Estimated Implementation Time**: 8-12 hours total
- Phase 1: 3-4 hours (workflow rewrite)
- Phase 2: 1-2 hours (summarization logic)
- Phase 3: 2-3 hours (templates and validation)
- Phase 4: 1-2 hours (infrastructure specs)
- Phase 5: 1-2 hours (documentation)
- Phase 6: 2-3 hours (testing and validation)

**Key Dependencies**:
- Phase 2-6 depend on Phase 1 (workflow structure must exist first)
- Phase 6 testing requires Phases 1-5 complete
- All phases can reference templates defined in Phase 3

**Non-Goals for Initial Implementation**:
- Actual implementation (this is PLANNING only per SDD constraint)
- Multi-point summarization (deferred to future)
- ML-based complexity classification (deferred to future)
- Workflow mode switching mid-flight (deferred pending validation)
- Incremental consensus alternative (requires research)

---

**End of Specification**

This specification represents the synthesis of LLM engineering and DX engineering perspectives through positive-sum consensus building. The hybrid 7-phase workflow with progressive summarization and workflow modes addresses both context management (technical constraint) and developer experience (usability constraint) simultaneously, demonstrating that the best architectures emerge when cross-domain constraints reveal shared solutions.
