# Architectural Review: kg Claude Code Plugin Improvement Plan

## Executive Summary

The proposed plan to improve the kg plugin is well-scoped and addresses real maintainability issues. However, there are significant architectural concerns regarding skill design, feature references, and the overall skill system maturity that merit careful consideration before implementation.

## Architecture Improvements to Consider

### 1. Skill Activation and Invocation Model (CRITICAL)

**Current Issue**: The improve-skills skill references non-existent features like `user-invocable: false`, `context: fork`, and `argument-hint`. These represent a fundamentally different skill activation model that doesn't exist in the current system.

**Impact on Plugin**: This creates a semantic mismatch between what the skill documents and what Claude Code actually supports. Future developers will be confused about what's theoretically possible vs. what's actually implemented.

**Recommendation**:
- Separate "planned features" from "current features" explicitly in skill documentation
- Create a clear feature matrix document at repository level showing which frontmatter options are supported vs. planned
- Consider creating a separate "future-vision.md" that describes long-term skill system enhancements without polluting current skill instructions

### 2. Skill Scope and Responsibility Separation

**Current Issue**: The sdd-plan skill tries to be both a process orchestrator AND an instruction template. The skill description says "follow the SDD planning process" but then provides detailed instructions about SPEC directory anatomy, phases, and subagent patterns.

**Better Architecture**: Skills should either be:
- **Type A (Orchestrator)**: Invoke subagents and coordinate work - has minimal instructions, mostly defines process flow
- **Type B (Guided Process)**: Provide detailed step-by-step guidance - is self-contained and doesn't spawn agents

The current sdd-plan skill conflates both approaches, which makes it harder to reason about what the skill actually does.

**Recommendation**:
- Decide: Should sdd-plan be an orchestrator that spawns subagents, or a detailed guide for human-driven SDD planning?
- If orchestrator: Move most detail to a separate "sdd-process.md" reference document, keep skill as lightweight coordinator
- If guide: Remove references to subagent spawning and focus on guiding a single AI instance through SDD phases

### 3. Context Injection and Dynamic Configuration

**Current Issue**: The improve-skills skill references `${CLAUDE_SESSION_ID}` variable interpolation without clarity on:
- Where/when this interpolation happens
- What other variables are available
- How this affects skill portability

**Architectural Concern**: If variables are interpolated at plugin load time, this creates tight coupling between the plugin system and the skill content. If interpolated at invocation time, there's unclear state management.

**Recommendation**:
- Document the variable interpolation lifecycle explicitly
- Create a "variables reference" document if more variables exist
- Consider whether dynamic context is better handled via explicit arguments vs. automatic injection

## Impact on Entire Plugin Architecture

### Positive Impacts
1. **Clarity**: Fixing typos and non-existent references will make the plugin more maintainable
2. **Scaffolding**: Adding CLAUDE.md and README establishes clear repository patterns
3. **Discoverability**: Documentation improvements help future contributors understand structure

### Negative Impacts / Risks
1. **False Promises**: Documenting non-existent features in CLAUDE.md creates expectation debt
2. **Version Fragility**: Version bump to 0.0.2 suggests stability, but feature gaps suggest early alpha (0.0.1 may be more appropriate)
3. **Skill System Clarity**: Without clear feature documentation, future skills may also reference unsupported features

### Structural Concerns
- **No skill registry or discovery mechanism**: With only 2 skills, this is fine, but doesn't scale
- **No skill versioning**: If skills change, there's no way to pin to a specific skill version
- **No dependency declarations**: Skills can't declare dependencies on other skills or plugins
- **No error handling contracts**: No standards for how skills should fail or report errors

## Implementation Concerns

### The sdd-plan Skill

**Issue 1: Subagent Invocation Clarity**
- The skill mentions "Run specialised subagents in parallel" but doesn't explain HOW
- Recommendation in plan says "use Task tool" but this is vague
- Better approach: Provide actual example code showing Skill tool invocation with concrete arguments

**Issue 2: Process vs. Execution Ambiguity**
- Steps 1-3 describe a thinking process (ultrathink, explore, propose)
- Steps 4-7 describe execution of SPEC directory operations
- The transition between thinking and doing isn't clear for an AI agent
- Better approach: Explicitly mark which steps are "thinking" vs. "doing" with different formatting/structure

**Issue 3: Two-Stage Subagent Feedback Loop**
- Running subagents twice (stage 1 and stage 2) is expensive and unclear when this is justified
- Recommendation misses: When should a developer use single-stage vs. two-stage?
- Better approach: Add decision criteria for when two-stage feedback is worth the cost

### The improve-skills Skill

**Issue 1: Session Scope Ambiguity**
- References `${CLAUDE_SESSION_ID}` but not clear what "the current session" means
- Recommendation: Add explicit clarification: "the session in which this skill was invoked"

**Issue 2: Analysis Heuristics Aren't Actionable**
- Lists conversation patterns (negative, neutral, positive signals) but doesn't define thresholds
- When is (-) signal serious enough to act on?
- Better approach: Add scoring/threshold guidance for when to recommend improvements

**Issue 3: Feature Proposal Scope**
- Example shows proposing entirely new skills (gitlab skill) as "improvement"
- This extends beyond "improve the skill" into "improve the plugin architecture"
- Better approach: Clarify whether improve-skills should suggest new skills or just refine existing ones

## Previously Unnoticed Tradeoffs

### 1. Documentation Completeness vs. Skill Clarity
- Adding comprehensive CLAUDE.md helps future developers
- BUT it adds maintenance burden - documentation can become stale
- Tradeoff: Document only what must be documented; let code/skills speak for themselves where possible

### 2. Feature Completeness vs. Early Feedback
- Including non-existent features in documentation sets expectations
- Alternative: Ship 0.0.1 with only implemented features, gather feedback, plan 0.1.0 with new features
- Current plan tries to document future state, which is higher risk

### 3. Subagent Generalization vs. Plugin Specificity
- sdd-plan is extremely detailed and specific to SDD methodology
- But skills should probably be reusable across repositories
- Question: Should this be a plugin-specific skill, or a general-purpose skill that can be loaded by any plugin?

### 4. Instruction Tokens vs. Skill Effectiveness
- improve-skills skill includes detailed pattern detection and examples
- This consumes significant instruction tokens from every invocation
- But simpler guidance might miss important patterns
- Tradeoff: Current verbosity is probably justified given the skill's purpose, but should be monitored

## Functional and Non-Functional Requirements to Add

### FR6: Skill Feature Parity
- **Requirement**: All frontmatter options referenced in skills must be either implemented or explicitly marked as "planned"
- **Test**: Run each skill and verify frontmatter is recognized; document parsing errors

### FR7: Variable Documentation
- **Requirement**: All variable references (e.g., `${CLAUDE_SESSION_ID}`) must be documented with scope, availability, and examples
- **Test**: Create a reference document and have a test invocation verify variable is populated

### FR8: Error Scenarios
- **Requirement**: Each skill must document what constitutes failure and how to recover
- **Test**: Trigger error conditions and verify skill behavior matches documentation

### NFR5: Skill System Compatibility
- **Requirement**: Skills must only reference features supported by Claude Code version they target
- **Implementation**: Add version metadata to frontmatter, check compatibility in CI/CD

### NFR6: Instruction Token Budget
- **Requirement**: Keep skill instructions under token limit to preserve main conversation context
- **Implementation**: Add size checks to build process

### NFR7: Skill Discoverability
- **Requirement**: All available skills must be discoverable without reading code
- **Implementation**: Generate skills catalog from metadata

## Concerns and Warnings

### 1. **Feature Validation Debt** (CRITICAL)
The plan doesn't include verification that referenced features actually work:
- `disable-model-invocation: true` - is this parsed and respected?
- `${CLAUDE_SESSION_ID}` - does this actually get populated?
- Variable syntax - is this actually supported?

**Warning**: Merging non-validated feature references creates technical debt that compounds over time.

**Mitigation**: Before merging any documentation update, verify every feature works with a test invocation.

### 2. **Skill Coupling Risk**
The improve-skills skill has implicit coupling to sdd-plan (references SDD patterns in examples). If sdd-plan changes significantly, improve-skills examples may become misleading.

**Mitigation**: Keep skills independent; if they reference each other, use explicit version markers.

### 3. **Documentation Maintenance Burden**
Adding CLAUDE.md, README, and inline documentation increases maintenance surface area. As skills grow, this documentation must evolve.

**Mitigation**: Automate what can be automated (skill catalog generation, metadata validation).

### 4. **Skill System Maturity Gap**
The plugin is trying to use a skill system that doesn't yet support planned features. This creates ambiguity about what's a blocker vs. what's a nice-to-have.

**Mitigation**: Create a feature roadmap document that separates "supported", "planned", and "investigating" categories.

### 5. **Versioning Strategy Unclear**
Bumping to 0.0.2 suggests incremental improvement, but the plugin seems to be in active design phase. Should it even have a version yet, or should it remain 0.0.1 until API stabilizes?

**Recommendation**: Consider staying at 0.0.1 until the skill system features stabilize.

## Architectural Recommendations Summary

| Recommendation | Priority | Effort | Impact |
|---|---|---|---|
| Separate "planned" from "implemented" features | HIGH | Low | Prevents confusion and future technical debt |
| Add feature validation tests to CI/CD | HIGH | Medium | Prevents non-functional feature references |
| Clarify skill scope (orchestrator vs. guide) for sdd-plan | MEDIUM | Medium | Makes skill purpose clearer and more maintainable |
| Create feature matrix/roadmap document | MEDIUM | Low | Establishes clear expectations for Claude Code integration |
| Add error handling contracts to skill documentation | MEDIUM | Low | Improves skill robustness and debuggability |
| Reconsider version number (keep 0.0.1 longer) | LOW | Trivial | Signals true stability level |
| Automate skill documentation generation | LOW | High | Reduces documentation maintenance burden long-term |

## Final Architectural Assessment

The plan is **sound but incomplete**. The identified issues (typos, missing docs, feature clarity) are real and should be fixed. However, the plan doesn't address the underlying skill system maturity questions that will become critical as more skills are added.

**Recommendation**: Proceed with Phase 1-2 fixes as planned, but add a **Phase 0** that clarifies skill system feature support before updating skill instructions with promises about unsupported features.
