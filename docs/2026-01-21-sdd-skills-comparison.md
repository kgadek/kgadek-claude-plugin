---
description: Comprehensive comparison of four specification-driven development approaches for Claude Code (sdd-plan1, sdd-plan2, obra-superpowers, feature-dev) analyzing planning rigor, token efficiency, consensus mechanisms, and execution support.
author: Opus 4.5
created_at: 2026-01-21 16:45 +0000
---

# Comprehensive SDD Skills Comparison

## Executive Summary

This analysis compares four specification-driven development approaches for Claude Code, ranging from the minimalist **feature-dev** (126 lines) to the comprehensive **sdd-plan2** (916 lines). Each represents different philosophies on the planning-to-implementation continuum.

| Skill | Size | Agents | Consensus Passes | Primary Focus |
|-------|------|--------|------------------|---------------|
| sdd-plan1 | 232 lines | 4-6 | 2 | Planning-only |
| sdd-plan2 | 916 lines | 3-6 (mode-dependent) | 1-3 (mode-dependent) | Planning-only |
| obra-superpowers | ~400 lines (composite) | 1 planner + N executors | 0 (review-based) | Full lifecycle |
| feature-dev | 126 lines | 5-8 | 0 | Full lifecycle |

---

## Individual Skill Analysis

### 1. sdd-plan1 (Original)

**Architecture**: 8-stage sequential workflow with two-pass consensus among specialized subagents.

**Workflow**:
1. Understand request
2. Explore codebase (parallel Haiku agents)
3. Propose design (ultrathink)
4. Create initial plan (00-initial-plan.md)
5. User refinement loop
6. **First subagent reading**: 4-6 agents provide independent feedback (01-feedback-*.md)
7. **Second subagent reading**: Same agents read all feedback, apply positive-sum thinking (02-feedback-*.md)
8. Final review -> spec.md

**Strengths**:
- **Focused scope**: Planning-only constraint prevents scope creep
- **Cross-functional review**: Multiple specialist perspectives (backend, frontend, security, devops, QA, architect)
- **Positive-sum consensus**: Second pass explicitly seeks common ground
- **Manageable size**: 232 lines provides complete guidance without overwhelming context

**Weaknesses**:
- **No context budget**: No token limits on feedback files; risk of context exhaustion
- **Fixed workflow**: No mode selection for simple vs complex tasks
- **No progressive summarization**: Phase 8 must read all feedback files
- **No state persistence**: Cannot resume interrupted workflows

**Token Efficiency**: Moderate. Two passes generate 8-12 feedback files without summarization, potentially 40-60k tokens before synthesis.

---

### 2. sdd-plan2 (Evolution)

**Architecture**: Hybrid 7-phase workflow with adaptive complexity modes (Express/Deep Dive) and progressive summarization.

**Workflow**:
- **Phase 0**: Mode selection (Express: 3 agents, Deep Dive: 4-6 agents)
- **Phase 1**: Discovery with user approval gate
- **Phase 2**: Codebase exploration (2-3 Haiku agents parallel)
- **Phase 3**: Clarifying questions (5-10, blocking)
- **Phase 4**: Architecture design
  - *Express*: Single-pass with 3 agents
  - *Deep Dive 4a*: First consensus (4-6 agents independent)
  - *Deep Dive 4b*: Second consensus (positive-sum) + **progressive summarization** (04-phase4-summary.md, 2000 tokens max)
  - *Deep Dive 4c*: **Optional** concrete alternatives (user-triggered)
- **Phase 5**: Implementation planning (Files/Do/Verify structure, 5-component verification)
- **Phase 6**: Plan quality review (Deep Dive only, 3 reviewers)
- **Phase 7**: Final spec synthesis

**Key Innovations over sdd-plan1**:
1. **Workflow modes**: Express (40-80 min, ~25k tokens) vs Deep Dive (90-180 min, ~60k tokens)
2. **Progressive summarization**: 2000-token summary after Phase 4b reduces context by 60-70%
3. **Context budget management**: 2000 tokens/feedback file, 60k total, auto-summarization at 50k
4. **State persistence**: `.sdd-state.json` enables resume, `ABORTED.md` for clean bailout
5. **Orchestration**: `.workflow-status.json` with 10-minute timeout per parallel phase
6. **Validation checkpoints**: Critical blocks, Important/Minor warns
7. **5-component verification**: Working directory, command, expected success/failure, stability note

**Strengths**:
- **Scalable complexity**: Simple features don't suffer 180-minute workflows
- **Context-aware**: Budget management prevents synthesis failures
- **Resumable**: State persistence handles interruptions
- **Quality gates**: Tiered validation prevents drift
- **Comprehensive templates**: Every artifact type specified

**Weaknesses**:
- **Skill size**: 916 lines is ~4x larger than sdd-plan1
  - **However**: Subagents receive focused prompts, not the full skill
  - **Risk**: Orchestrator may face context pressure from skill instructions
- **Complexity overhead**: Mode selection, state management, validation all add cognitive load
- **Potential over-engineering**: Express mode still has 5+ phases
- **Three-tier feedback question**: Does Phase 4c actually add value, or duplicate 4a/4b?

**Token Efficiency**: High. Progressive summarization + context budgets explicitly designed for 60k synthesis limit. Express mode targets ~25k total.

---

### 3. obra-superpowers (Modular Composition)

**Architecture**: Decomposed skill set with separate workflows for brainstorming, planning, and execution.

**Workflow** (composite):

**Phase A - Brainstorming** (`brainstorming/SKILL.md`):
- Check project state (files, docs, commits)
- One question at a time, prefer multiple choice
- Present 2-3 approaches with trade-offs
- Incremental design validation (200-300 word sections)
- Output: `docs/plans/YYYY-MM-DD-<topic>-design.md`

**Phase B - Planning** (`writing-plans/SKILL.md`):
- Creates implementation plan with bite-sized tasks (2-5 min each)
- TDD structure: write failing test -> verify fail -> implement -> verify pass -> commit
- **Exact file paths, complete code in plan**
- Output: `docs/plans/YYYY-MM-DD-<feature-name>.md`

**Phase C - Execution** (choose one):
- **subagent-driven-development**: Fresh subagent per task + two-stage review
  - Stage 1: Spec compliance review (did they build what was requested?)
  - Stage 2: Code quality review (is it well-built?)
  - **Review loops** until approved
- **executing-plans**: Batch execution (3 tasks) with human checkpoints

**Strengths**:
- **Modular skills**: Can use brainstorming alone, skip planning, etc.
- **Complete lifecycle**: From idea to merged code
- **TDD-native**: Built around red/green/refactor cycles
- **Fresh context per task**: Subagent dispatch avoids context pollution
- **Two-stage review**: Separates spec compliance from code quality
- **Practical execution**: Complete code in plan enables copy-paste implementation
- **Git worktree integration**: Parallel development isolation

**Weaknesses**:
- **No multi-agent consensus during planning**: Single orchestrator writes plan
  - **Counter**: Review-based correction catches issues during execution
- **Execution-heavy**: Most intelligence in execution phase, not planning
- **Less upfront rigor**: Errors caught during implementation, not specification
- **No explicit context budget**: Relies on skill modularity for context management

**Token Efficiency**: Good. Modular skills keep each phase focused. Subagent dispatch provides fresh context windows. But lacks explicit budget controls.

---

### 4. feature-dev (Anthropic Official)

**Architecture**: 7-phase linear workflow with parallel agent exploration and single-pass architecture design.

**Workflow**:
1. **Discovery**: Understand feature, confirm with user
2. **Codebase Exploration**: 2-3 code-explorer agents in parallel, return 5-10 key files each
3. **Clarifying Questions**: Identify all ambiguities, **block until answered**
4. **Architecture Design**: 2-3 code-architect agents with different focuses (minimal, clean, pragmatic)
5. **Implementation**: After explicit user approval
6. **Quality Review**: 3 code-reviewer agents (simplicity, bugs, conventions)
7. **Summary**: Document what was built

**Strengths**:
- **Minimal footprint**: 126 lines provides complete guidance
- **80-20 efficiency**: Covers essential bases without elaborate infrastructure
- **Clear user gates**: Approval required before implementation
- **Agent specialization**: Explorer, architect, reviewer roles
- **Practical focus**: Actually implements the feature

**Weaknesses**:
- **Single-pass architecture**: No consensus building between architects
- **No verification templates**: Implementation quality depends on reviewer agents
- **No state persistence**: Cannot resume workflows
- **No context budget**: Risk of context exhaustion on large features
- **Less rigorous planning**: Architecture selection is preference-based, not consensus-based

**Token Efficiency**: Moderate. Compact skill text but no explicit management of agent output sizes.

---

## Objective Metrics Framework

| Metric | sdd-plan1 | sdd-plan2 | obra-superpowers | feature-dev |
|--------|-----------|-----------|------------------|-------------|
| **Skill Size (lines)** | 232 | 916 | ~400 (composite) | 126 |
| **Planning Phases** | 8 | 7 (+subphases) | 2 (brainstorm + plan) | 4 |
| **Execution Support** | None | None | Full | Full |
| **Consensus Passes** | 2 | 1-3 (mode) | 0 | 0 |
| **Parallel Agents (peak)** | 6 | 6 | 3 | 3 |
| **Token Budget** | None | 60k explicit | None | None |
| **Progressive Summarization** | No | Yes | No | No |
| **State Persistence** | No | Yes | No | No |
| **Workflow Modes** | None | Express/Deep | Modular skills | None |
| **Verification Rigor** | Template | 5-component | TDD execution | Agent review |
| **Estimated Planning Time** | 60-120 min | 40-180 min (mode) | 30-90 min | 40-90 min |
| **GitHub Stars** | N/A | N/A | 31.9k | 4.5k (marketplace) |

---

## Key Tradeoffs Analysis

### 1. Skill Size vs. Subagent Clarity

**Question**: Does sdd-plan2's 4x size degrade performance?

**Analysis**: The skill size primarily affects the **orchestrating agent**, not subagents. Subagents receive:
- Task-specific prompts (not the full skill)
- Focused templates (01-feedback-{role}.md template is ~20 lines)
- Clear scope constraints

**Verdict**: Size is a concern for the orchestrator's context window, but progressive summarization and context budgets mitigate this. The comprehensive templates actually **help** subagents by providing unambiguous structure.

**Risk**: If the orchestrator must frequently re-read the skill, 916 lines competes with working context. Consider: can the orchestrator cache phase instructions?

---

### 2. Consensus Building vs. Brainstorming Flexibility

**sdd-plan1/2 approach**: Multiple agents provide feedback simultaneously, then converge through positive-sum thinking.

**obra-superpowers approach**: Single orchestrator brainstorms with user one question at a time, presents 2-3 approaches, user selects.

**Tradeoff**:
- **Consensus**: Catches more edge cases upfront, reduces implementation rework, but takes longer and may produce "design by committee" compromise
- **Brainstorming**: Faster, preserves bold/unconventional approaches, but may miss specialist concerns (security, devops)

**Verdict**: Consensus is superior for **complex, cross-cutting features** (auth systems, API designs). Brainstorming is superior for **creative/exploratory work** where unconventional approaches have value.

---

### 3. Three-Tier vs. Two-Tier Feedback

**sdd-plan1**: Two passes (independent -> positive-sum consensus)

**sdd-plan2**: Three passes (independent -> positive-sum -> optional concrete alternatives)

**Analysis**:
- Phase 4c generates 2-3 additional architecture documents
- Value proposition: When 4a/4b produce unresolved disagreements, concrete alternatives clarify tradeoffs
- Risk: May duplicate thinking already present in 4b's "Recommended Concrete Approach" section

**Verdict**: Making 4c **optional and user-triggered** (as sdd-plan2 does) is the right call. Default three-pass would add 10-20 min for marginal benefit. Monitor: if >30% of Deep Dive workflows need 4c, reconsider default.

---

### 4. Planning-Only vs. Full Lifecycle

**sdd-plan1/2**: Planning only, explicit prohibition on implementation

**obra-superpowers/feature-dev**: Planning + implementation in same workflow

**Tradeoff**:
- **Planning-only**: Forces human review before any code changes, cleaner separation of concerns, can hand off plan to different implementer
- **Full lifecycle**: Faster time-to-value, context preserved across phases, catches plan issues during implementation

**Verdict**: Depends on use case:
- **High-stakes changes**: Planning-only (sdd-plan) for explicit human review gate
- **Iteration velocity**: Full lifecycle (feature-dev, obra) when trusted to implement

---

### 5. Review During Execution vs. During Planning

**obra-superpowers**: Catches issues during execution via two-stage review (spec compliance -> code quality)

**sdd-plan2**: Catches issues during planning via multi-agent consensus + Phase 6 quality review

**Analysis**:
- obra's approach: Faster to first feedback, but may require implementation rework
- sdd-plan2's approach: More upfront investment, but implementation follows validated plan

**Verdict**: obra-superpowers' approach is more **empirical** (test in execution), sdd-plan2 is more **theoretical** (prove in planning). For complex architectural decisions, upfront validation is worth the investment. For incremental features, obra's execution-time review is efficient.

---

## Recommendations by Scenario

### Scenario 1: Simple Feature (1-3 files, <4 hours)

**Recommended**: **feature-dev** or **sdd-plan2 Express Mode**

**Rationale**:
- feature-dev's 126 lines provide sufficient structure without overhead
- sdd-plan2 Express (40-80 min, 3 agents) is acceptable if planning-only is required
- Avoid: sdd-plan1 (no lightweight mode), obra-superpowers (overkill for simple features)

### Scenario 2: Complex Feature (8+ files, new patterns)

**Recommended**: **sdd-plan2 Deep Dive Mode**

**Rationale**:
- Multi-agent consensus catches cross-cutting concerns
- Progressive summarization handles large feedback volume
- Context budget prevents synthesis failures
- Phase 6 quality review catches plan issues before implementation

### Scenario 3: Exploratory/Creative Work

**Recommended**: **obra-superpowers** (brainstorming skill alone)

**Rationale**:
- One-question-at-a-time preserves creative space
- Incremental validation (200-300 word sections) prevents over-commitment
- YAGNI enforcement prevents feature creep
- Can skip planning and execute directly if approach is clear

### Scenario 4: TDD-Heavy Implementation

**Recommended**: **obra-superpowers** (full workflow)

**Rationale**:
- TDD is core to skill design (failing test -> implement -> pass -> commit)
- Bite-sized tasks (2-5 min) natural for red/green cycles
- Fresh subagent per task avoids context pollution in test fixtures
- Two-stage review (spec then quality) catches both functional and non-functional issues

### Scenario 5: High-Stakes Architectural Change

**Recommended**: **sdd-plan2 Deep Dive** with **Phase 4c enabled**

**Rationale**:
- Maximum rigor: 4-6 agents, two consensus passes, optional alternatives
- Explicit verification templates with 5 components
- State persistence for long workflows
- Validation checkpoints prevent drift
- Planning-only constraint ensures human review before any changes

### Scenario 6: Rapid Prototyping

**Recommended**: **feature-dev**

**Rationale**:
- Minimal ceremony, fast time-to-implementation
- User can answer "whatever you think is best" to proceed quickly
- Quality review catches issues without blocking progress
- Summary documents decisions for future reference

---

## Conclusions

### Does sdd-plan2's size hurt performance?

**Partially**. The orchestrator bears the context cost, but:
- Subagents receive focused prompts, not the full skill
- Progressive summarization reduces downstream context pressure
- Express Mode provides escape hatch for simple tasks

**Recommendation**: Monitor orchestrator context usage. If consistently >40k before Phase 7, consider splitting skill into modular components (like obra-superpowers does).

### Does consensus defeat brainstorming?

**No, they solve different problems**:
- Consensus: Convergent thinking, cross-functional validation, risk mitigation
- Brainstorming: Divergent thinking, creative exploration, unconventional solutions

**Recommendation**: Use sdd-plan for **convergent** phases (architecture selection, implementation planning) and obra-superpowers brainstorming for **divergent** phases (ideation, approach exploration).

### Is three-tier feedback (sdd-plan2) better than two-tier (sdd-plan1)?

**Marginally, when optional**. Phase 4c adds value only when significant disagreements exist. Making it user-triggered (as sdd-plan2 does) prevents wasted effort.

### Does feature-dev achieve 80-20 efficiency?

**Yes, for its scope**. It covers exploration, clarification, architecture, implementation, and review in 126 lines. However, it lacks:
- Context budget management (risk for large features)
- State persistence (cannot resume)
- Consensus building (may miss specialist concerns)

**Recommendation**: feature-dev is excellent for **trusted, medium-complexity features** where developer has good domain knowledge.

---

## Final Scoring Matrix

| Criteria (1-5) | sdd-plan1 | sdd-plan2 | obra-superpowers | feature-dev |
|----------------|-----------|-----------|------------------|-------------|
| **Planning Rigor** | 4 | 5 | 3 | 3 |
| **Execution Support** | 1 | 1 | 5 | 5 |
| **Token Efficiency** | 2 | 5 | 4 | 3 |
| **Flexibility** | 2 | 4 | 5 | 3 |
| **Complexity Scaling** | 2 | 5 | 4 | 3 |
| **Resume/Recovery** | 1 | 5 | 2 | 1 |
| **Learning Curve** | 3 | 2 | 3 | 5 |
| **Time to Value** | 2 | 3 | 4 | 5 |
| **TOTAL** | **17** | **30** | **30** | **28** |

**sdd-plan2** and **obra-superpowers** tie for highest score but excel in different dimensions:
- sdd-plan2: Planning rigor, token efficiency, complexity scaling, recovery
- obra-superpowers: Execution support, flexibility, TDD integration

**Recommendation**: Use sdd-plan2 for **complex planning with human handoff**, use obra-superpowers for **end-to-end autonomous development**.

---

## Appendix: Source Files Analyzed

- `plugins/kg/skills/sdd-plan1/SKILL.md` (232 lines)
- `plugins/kg/skills/sdd-plan2/SKILL.md` (916 lines)
- `ai-spec/2026-01-21-sdd-inspirations/references/obra-superpowers/skills/brainstorming/SKILL.md`
- `ai-spec/2026-01-21-sdd-inspirations/references/obra-superpowers/skills/writing-plans/SKILL.md`
- `ai-spec/2026-01-21-sdd-inspirations/references/obra-superpowers/skills/subagent-driven-development/SKILL.md`
- `ai-spec/2026-01-21-sdd-inspirations/references/obra-superpowers/skills/executing-plans/SKILL.md`
- `ai-spec/2026-01-21-sdd-inspirations/references/anthropics-claude-plugins-official/plugins/feature-dev/commands/feature-dev.md`
- `ai-spec/2026-01-21-sdd-inspirations/spec.md` (sdd-plan2 specification)

---

## Appendix: user prompt

> You are an expert in LLMs, agent optimisation, Claude Code, implementing multi-agent workflows. I want you to do a comprehensive comparison of
>
> I want a thorough comparsion between the skills. Each has a similar objective: write a plan, only then proceed with implementation. However they differ.
> - sdd-plan2 is an evolution of sdd-plan1, however it's much larger. Does it consume context needlessly and degrades performance, or is it more accurate thanks to comprehensive steps and subagents starting with clear context are not concerned with this skill size? Will the consensus defeat the brainstorming capabilities? Is the three-tier feedback better in sdd-plan2 than a two-tier feedback in sdd-plan1?
> - obra-superpowers seems comprehensive and popular, however lacks two-stage feedback between subagents.
> - feature-dev is a much simpler. Does it strike 80-20 efficiency?
>
> Use
>
>
> You are an expert in Large Language Models (LLMs), agent optimization, Claude Code, and implementing multi-agent workflows. Your task is to perform a comprehensive comparison of Specification-Driven Development (SDD) skills.
>
> Here are the SDD skills you need to compare:
>
> - @plugins/kg/skills/sdd-plan1/
> - @plugins/kg/skills/sdd-plan2/
> - @ai-spec/2026-01-21-sdd-inspirations/references/obra-superpowers/
> - @ai-spec/2026-01-21-sdd-inspirations/references/
>
> Each of these skills shares a similar objective: write a plan first, then proceed with implementation. However, they differ in their specific approaches and details.
>
> - sdd-plan1 was the original `sdd-plan`
> - sdd-plan2 was conceived as updated `sdd-plan` via @ai-spec/2026-01-21-sdd-inspirations/spec.md
> - obra-superpowers is an opinionated and seemingly popular existing solution - this repo has 31.9k stars on GitHub. It's described at https://blog.fsck.com/2025/10/09/superpowers/
> - feature-dev is from anthropics/claude-plugins-official and is relatively popular (44.6K installs). Marketplace has 4.5k stars on GitHub.
>
> Your task is to conduct a thorough, expert-level comparison of these skills. To do this effectively:
>
> 1. **Deep Analysis**: Use chain of thought reasoning to analyze each skill individually. Consider:
>    - The specific methodology each skill employs
>    - How the planning phase is structured
>    - How implementation follows from the plan
>    - The level of detail and rigor in each approach
>    - Potential use cases where each skill excels
>
> 2. **Objective Metrics**: Formulate concrete, measurable criteria for comparison, such as:
>    - Complexity of planning phase
>    - Time investment required
>    - Clarity and specificity of outputs
>    - Scalability to different project sizes
>    - Error reduction potential
>    - Cognitive load on the developer/agent
>    - Token/context efficiency
>
> 3. **Tradeoff Analysis**: Research and reason about critical tradeoffs, including:
>    - Context usage vs performance gains
>    - Planning overhead vs implementation speed
>    - Specification detail vs flexibility
>    - Diminishing returns in multi-agent workflows
>    - When additional planning steps stop providing value
>    - Memory/context window constraints in practice
>
> 4. **Comparative Evaluation**: Directly compare the skills against each other on your established metrics
>
> Use the scratchpad below to work through your analysis systematically:
>
> <scratchpad>
> First, analyze each skill individually using chain of thought reasoning. Then develop your objective metrics. Finally, perform the comparative analysis considering all relevant tradeoffs.
> </scratchpad>
>
> After completing your analysis in the scratchpad, provide your final comparison. Your comparison should include:
>
> 1. **Individual Skill Analysis**: A detailed breakdown of each skill with chain of thought reasoning
> 2. **Objective Metrics Framework**: The specific metrics you've developed for comparison
> 3. **Comparative Matrix**: A direct comparison showing how each skill performs on your metrics
> 4. **Tradeoff Analysis**: Discussion of key tradeoffs including context usage vs performance, diminishing returns, and other relevant factors
> 5. **Recommendations**: Guidance on when to use each skill based on different scenarios and constraints
>
> Structure your final answer with clear sections and provide specific, actionable insights. Your output should consist of only your final comparison; no need to repeat the scratchpad content.

---

*Analysis conducted: 2026-01-21*
