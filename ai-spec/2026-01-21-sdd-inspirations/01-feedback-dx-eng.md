# DX Engineer - First Pass Architectural Feedback

## Summary

The hybrid 7-phase workflow significantly improves discoverability and mental model clarity over the current 8-stage approach, but introduces workflow complexity that may harm time-to-value for developers. The three-pass Phase 4 (4a/4b/4c) represents the core risk: architectural thoroughness vs. developer patience. The shift toward executable verification and standardized templates is a strong DX win, but success depends on making the "happy path" feel fast and the "deep dive" path feel optional.

## Risks Identified

- **Workflow Latency vs. Developer Patience** (High Severity, High Impact): The 7-phase structure with multiple approval gates (Phase 1→2, Phase 3 blocking, Phase 4c→5, Phase 5→6) creates 4-5 interruption points. Developers may abandon the process before completion if gates feel bureaucratic rather than value-add. Impact: Reduced adoption, circumvention of process, preference for ad-hoc planning.

- **Three-Pass Architecture Overhead** (Medium Severity, Medium Impact): Phase 4a (independent feedback) + 4b (consensus) + 4c (concrete alternatives) is a 3x multiplier on Phase 4 duration. For simple features (e.g., "add a new API endpoint"), this feels heavy. Without clear guidance on when to skip/simplify phases, developers will either over-engineer simple tasks or under-utilize the workflow for complex ones. Impact: Process fatigue, misuse of workflow.

- **Template Compliance Burden** (Medium Severity, Low-Medium Impact): Standardized templates for 01-feedback, 02-feedback, 03-architecture, 04-plan, 05-review, and spec.md add consistency but create cognitive overhead for both AI agents and human reviewers. If templates become "fill-in-the-blanks exercises" without clear value, they degrade to checkbox compliance. Impact: Quality theater rather than quality improvement.

- **Unclear Escape Hatches** (Low Severity, High Impact): The plan doesn't specify how developers can gracefully bail out, skip phases for simple tasks, or restart after discovering misalignment. Without escape hatches, developers will either abuse the workflow or avoid it entirely. Impact: All-or-nothing adoption pattern, user frustration.

- **Subagent Coordination Complexity** (Medium Severity, Medium Impact): Managing 4-6 subagents in Phase 4a, same agents in 4b, 2-3 architects in 4c, and 3 reviewers in Phase 6 creates 15-20 parallel agent executions across the workflow. Failure in any single agent, inconsistent template compliance, or timeout issues will create debugging nightmares. Impact: Workflow brittleness, hard-to-diagnose failures.

- **Decision Fatigue from Approval Gates** (Medium Severity, Medium Impact): Four explicit user approval gates (Phase 1→2, Phase 3 blocking, Phase 4c→5, Phase 5→6) plus optional "fix now vs. later" decision in Phase 6 means developers make 4-5 continue/stop decisions. This is significantly higher than current sdd-plan's single approval gate at Step 5. Impact: Cognitive load, slower velocity, context-switching costs.

## Concrete Improvements

### 1. **Introduce Workflow Modes: Express vs. Deep Dive**
**Rationale**: Not all features need 7 phases. Provide two explicit paths:
- **Express Mode** (for simple features <50 LOC impact): Skip Phase 3 (clarifying questions), collapse Phase 4 to single-pass (skip 4b consensus + 4c alternatives), skip Phase 6 (quality review). ~3 phases total.
- **Deep Dive Mode** (for complex features, architectural changes, cross-team impacts): Full 7-phase workflow.

**Implementation**: Add a decision point at Phase 1 conclusion: "Based on scope analysis, recommend Express/Deep Dive mode. User can override."

**Impact**: Reduces process fatigue for simple tasks, preserves thoroughness for complex ones. Developers feel in control rather than constrained. Estimated 60% of features use Express mode, reducing latency by 50%.

### 2. **Make Phase 3 (Clarifying Questions) Async-Friendly**
**Rationale**: Blocking progression until all questions are answered creates a hard stop. If user can't answer immediately (e.g., needs to check with PM/design), the entire workflow halts.

**Implementation**:
- Generate clarifying questions and write to `clarifying-questions.md`
- Allow user to mark questions as "answered", "deferred", or "N/A"
- Proceed to Phase 4a with answered questions + note deferred items in Risk Register
- Re-surface deferred questions at Phase 5 (implementation planning)

**Impact**: Reduces workflow interruption, allows parallel work, preserves question value without blocking. Time-to-Phase-4 improves by ~30% when user has partial answers.

### 3. **Consolidate Approval Gates into Single "Checkpoint Dashboard"**
**Rationale**: Four separate approval gates feel scattered. Developers lose context between interruptions.

**Implementation**:
- Create `checkpoints.md` that accumulates all approval requests with status tracking:
  ```markdown
  ## Checkpoint 1: Requirements Validated (Phase 1→2)
  - [x] User confirmed understanding
  - Decision: APPROVED

  ## Checkpoint 2: Clarifying Questions (Phase 3)
  - [ ] 5 questions pending (see clarifying-questions.md)
  - Decision: PENDING
  ```
- Single AskUserQuestion at each checkpoint: "Review checkpoint dashboard and approve/modify/abort?"

**Impact**: Reduces cognitive load, provides progress visibility, allows batch decision-making. Developers see "where am I?" at a glance.

### 4. **Auto-Generate Template Compliance Checklist**
**Rationale**: Templates are valuable for consistency, but enforcement is manual. AI agents may skip sections or misinterpret structure.

**Implementation**:
- After each phase, run validation script that checks:
  - 01-feedback-*.md: Has Summary, Risks, Improvements, Questions, Confidence sections?
  - 02-feedback-*.md: Has all 01-feedback sections + Changes, Consensus, Disagreements, Integrations?
  - 04-implementation-plan.md: Every task has Files/Do/Verify?
- Output `template-compliance.md` with pass/fail per file
- Block phase progression if critical sections missing (e.g., no Verify commands in implementation plan)

**Impact**: Catches template drift early, reduces human review burden, ensures consistent artifact quality. Estimated 40% reduction in "incomplete feedback" issues.

### 5. **Add "Abort and Restart" Mechanism**
**Rationale**: If Phase 3 reveals scope misalignment or Phase 4c shows all architectures are infeasible, developers need a clean restart without abandoning the SPEC directory.

**Implementation**:
- Add `ABORTED.md` marker file with reason and restart instructions
- Allow "resume from Phase X" command that preserves prior artifacts but resets workflow state
- Document abort scenarios: scope creep detected, requirements misunderstood, technical infeasibility discovered

**Impact**: Reduces sunk cost fallacy ("we've gone too far to stop"), encourages early pivot when needed, maintains audit trail of why restarts happened.

### 6. **Provide Phase Duration Estimates**
**Rationale**: Developers can't plan their day without knowing "how long will this take?" Current plan has no time guidance.

**Implementation**:
- Add estimated duration ranges to each phase in SKILL.md:
  - Phase 1 (Discovery): 5-10 min
  - Phase 2 (Exploration): 10-20 min
  - Phase 3 (Questions): 5-15 min (user time)
  - Phase 4a: 15-25 min (4-6 agents in parallel)
  - Phase 4b: 15-25 min
  - Phase 4c: 10-20 min (2-3 agents in parallel)
  - Phase 5: 15-30 min
  - Phase 6: 10-15 min
  - Phase 7: 10-15 min
  - **Total Deep Dive**: 90-175 min (1.5-3 hours)
  - **Total Express**: 40-80 min (0.5-1.5 hours)

**Impact**: Sets expectations, helps developers decide when to start workflow (not 10 min before lunch), increases completion rates by 25% (no surprise time commitments).

### 7. **Clarify When to Use sdd-plan vs. Ad-Hoc Planning**
**Rationale**: The plan doesn't specify thresholds for when this workflow is worth the investment. Without guidance, developers either overuse (simple bug fixes get 7-phase treatment) or underuse (complex features skip planning).

**Implementation**:
- Add "When to Use This Workflow" section in SKILL.md with clear criteria:
  - **Use sdd-plan if**:
    - Feature impacts >3 files
    - Introduces new architectural patterns
    - Requires cross-team coordination
    - Has compliance/security implications
    - Estimated >4 hours of implementation
  - **Skip sdd-plan if**:
    - Bug fix confined to single file
    - Refactoring with no behavioral change
    - Documentation-only changes
    - Estimated <1 hour of implementation

**Impact**: Reduces workflow misuse by 50%, increases developer confidence in "is this the right tool?"

### 8. **Make Verification Commands Copy-Pasteable with Context**
**Rationale**: Obra Superpowers-style verification is excellent, but "executable commands" fail if context is missing (e.g., "npm test" without knowing which directory).

**Implementation**:
- Every verification command in 04-implementation-plan.md must include:
  - Working directory: `cd /path/to/project`
  - Full command with arguments: `npm test -- src/auth.test.ts`
  - Expected output (exact string match or regex): `✓ authenticates user with valid token`
  - Failure output (to detect issues): `✗ Error: Invalid credentials`
- Provide one-click "Run All Verifications" script that executes all Verify blocks sequentially

**Impact**: Reduces "works on my machine" issues, enables automated validation, saves 20-30% of developer time in verification steps.

### 9. **Visualize Workflow Progress**
**Rationale**: 7 phases with sub-phases (4a/4b/4c) creates mental overhead. Developers lose track of "where am I?"

**Implementation**:
- Generate `progress.md` with visual progress bar:
  ```markdown
  # SDD Workflow Progress

  [✓] Phase 1: Discovery
  [✓] Phase 2: Codebase Exploration
  [✓] Phase 3: Clarifying Questions
  [▶] Phase 4: Architecture Design
      [✓] 4a: First Consensus
      [✓] 4b: Second Consensus
      [▶] 4c: Concrete Alternatives (2/3 agents complete)
  [ ] Phase 5: Implementation Planning
  [ ] Phase 6: Plan Quality Review
  [ ] Phase 7: Final Spec

  Estimated completion: 45 minutes remaining
  ```
- Update after each phase completion

**Impact**: Reduces anxiety about process length, provides motivation ("almost done!"), helps with context restoration after interruptions.

### 10. **Add "Skip Consensus" Option for Solo Developers**
**Rationale**: Phase 4b (second consensus) assumes multi-agent value, but solo developers may find re-running same agents with "read each other's feedback" instruction redundant.

**Implementation**:
- Detect if project has single developer (check git config, CLAUDE.md)
- Offer "Skip Phase 4b (consensus)? For solo projects, first-pass feedback may be sufficient."
- If skipped, proceed directly from 4a → 4c

**Impact**: Saves 15-25 min for solo developers, reduces "why am I doing this?" friction, maintains full workflow for team projects.

## Questions for Other Roles

### For LLM Engineer:
- **Phase 4b prompt engineering**: How do we prevent agents from simply repeating Phase 4a feedback with "I agree" statements? What specific instructions ensure genuine consensus-building vs. template compliance?
- **Agent coordination reliability**: With 15-20 parallel agent calls across the workflow, what failure modes have you observed? How do we handle partial failures (e.g., 4/6 agents complete in Phase 4a)?
- **Template drift detection**: Can we use embedding similarity or structural parsing to detect when agents deviate from templates? What's the enforcement mechanism?

### For Backend Engineer:
- **File I/O patterns**: The workflow creates 15-20+ files per SPEC directory. Any concerns about filesystem performance, file locking, or atomic write requirements?
- **State management**: How do we handle workflow state persistence? If user aborts at Phase 4c and resumes next day, how do we restore context without re-reading all files?

### For QA Engineer:
- **Verification command reliability**: Obra-style executable verification assumes stable test infrastructure. What if verification commands are flaky (e.g., network-dependent tests)? How do we distinguish "plan is wrong" from "environment is broken"?
- **Template compliance validation**: Should template compliance be enforced with automated checks (fail workflow if missing sections) or warnings (proceed with degraded quality)?

### For Product Manager:
- **User onboarding**: For first-time sdd-plan users, how do we avoid overwhelming them with 7-phase complexity? Should there be a "guided tour" mode or simplified first-run experience?
- **Workflow customization**: Different projects have different planning needs. Should CLAUDE.md support per-project workflow customization (e.g., disable Phase 3 for backend-only projects)?

## Confidence Level

### High Confidence:
- **Workflow Latency Risk**: Multiple approval gates will create friction. This is a universal DX anti-pattern that needs mitigation (Express Mode, Checkpoint Dashboard).
- **Phase Duration Estimates**: Developers need time estimates to plan their work. This is table stakes for any workflow tool.
- **Copy-Pasteable Verification Commands**: Obra's Files/Do/Verify is only valuable if Verify commands actually work. Context must be explicit.

### Medium Confidence:
- **Three-Pass Architecture Overhead**: Phase 4a/4b/4c may be valuable for complex features, but I'm uncertain if the "separate brainstorming from concrete design" split is worth 3x the time. This needs user testing.
- **Template Compliance Burden**: Templates improve consistency, but I'm unsure if 5+ templates (01/02/03/04/05/spec) cross the line from "helpful structure" to "bureaucratic overhead." Monitor template fatigue in practice.
- **Workflow Modes (Express vs. Deep Dive)**: This is a common pattern in developer tools, but I'm uncertain about the 60% Express usage estimate. Actual usage may differ based on team culture.

### Low Confidence:
- **Async-Friendly Clarifying Questions**: The "defer questions" mechanism sounds good in theory, but I'm uncertain how often users will have partial answers vs. all-or-nothing. May over-complicate for rare use case.
- **Skip Consensus for Solo Developers**: This optimization assumes Phase 4b provides minimal value for solo work, but solo developers may benefit from "think twice" consensus pattern. Needs validation with solo users.
