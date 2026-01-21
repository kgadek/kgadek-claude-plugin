# QA / TDD Practitioner Feedback

## Overview

The kg plugin has two skills designed for AI-assisted development workflows (sdd-plan and improve-skills). From a QA/TDD perspective, these skills present unique testing challenges due to their nature as prompt-based agents rather than traditional executable code. This feedback focuses on testability, validation strategies, edge cases, and quality assurance concerns.

---

## 1. Testability Assessment

### Current Testing Gaps

**sdd-plan/SKILL.md:**
- No validation mechanism for the SDD process steps (6 phases are described but there's no way to verify if they're followed)
- No success criteria defined for each planning phase
- No guardrails to prevent incomplete or circular reasoning in the planning output
- No test vectors for subagent collaboration quality

**improve-skills/SKILL.md:**
- No validation that session context was actually read
- No way to verify the "improve-skills" analysis is actually analyzing the right session
- References undefined variable `${CLAUDE_SESSION_ID}` with no fallback or error handling
- Unknown behavior when the feature references don't exist (e.g., `user-invocable`, `context: fork`)

### Missing Test Infrastructure

1. **No test harness for skills** - How do you verify a skill works without running it manually?
2. **No acceptance criteria** - What defines "success" for a planning session or improvement analysis?
3. **No regression tests** - How do you ensure fixes don't break existing functionality?
4. **No golden files** - No reference outputs to validate against

---

## 2. Test Cases That Should Exist

### sdd-plan/SKILL.md Tests

#### Phase 1: User Request Understanding
```
TEST: sdd-plan-01-empty-request
  Input: No user request provided
  Expected: Skill should fail gracefully with clear error message
  Validation: Error message mentions user request is required

TEST: sdd-plan-02-ambiguous-request
  Input: User request with multiple conflicting interpretations
  Expected: Skill asks clarifying questions
  Validation: Output contains at least 2 clarifying questions

TEST: sdd-plan-03-valid-request
  Input: Well-formed, clear user request
  Expected: Skill moves to exploration phase
  Validation: Output contains "Exploring" and references to codebase investigation
```

#### Phase 2: Codebase Exploration
```
TEST: sdd-plan-04-repo-structure-discovery
  Input: User request for a feature in a multi-language repo
  Expected: Skill identifies relevant files and patterns
  Validation: Output references specific files/directories from the repo

TEST: sdd-plan-05-no-relevant-code
  Input: Request for feature in empty/new repository
  Expected: Skill handles absence gracefully, documents baseline state
  Validation: Output explains the lack of code and suggests starting point

TEST: sdd-plan-06-circular-exploration
  Input: User request for feature that requires understanding itself
  Expected: Skill breaks circular dependency with explicit assumptions
  Validation: Output documents key assumptions clearly
```

#### Phase 3-4: Initial Plan Creation
```
TEST: sdd-plan-07-plan-completeness
  Input: Valid user request and codebase
  Expected: Initial plan covers all sections (alternatives, FR, NFR, etc.)
  Validation: Output contains all required markdown sections

TEST: sdd-plan-08-plan-consistency
  Input: Initial plan with internal contradictions
  Expected: Skill identifies and resolves contradictions
  Validation: No requirement contradicts another

TEST: sdd-plan-09-plan-no-todos
  Input: Complex request
  Expected: Phase-based TODO lists with clear dependencies
  Validation: TODOs are ordered by dependency
```

#### Phase 5-6: Subagent Coordination
```
TEST: sdd-plan-10-subagent-invocation
  Input: Request requiring multiple subagents (architect, backend, QA, DevOps, security)
  Expected: All subagents are invoked with correct context
  Validation: Output contains feedback from all 6 subagent perspectives

TEST: sdd-plan-11-subagent-contradiction-resolution
  Input: Multiple subagent perspectives with conflicting recommendations
  Expected: Final plan explains tradeoffs and makes explicit decisions
  Validation: Plan documents why one approach was chosen over others

TEST: sdd-plan-12-subagent-iteration
  Input: Second round of subagent feedback
  Expected: Subagents reference previous feedback and explain evolution
  Validation: Each 02-feedback-* file references relevant 01-feedback-* files
```

#### Phase 7: Final Spec Generation
```
TEST: sdd-plan-13-spec-completeness
  Input: All phases completed with feedback
  Expected: Final spec.md is executable/implementable
  Validation: Spec contains no "TBD" sections, all open questions resolved

TEST: sdd-plan-14-spec-validation
  Input: Final spec for implementation
  Expected: Spec can be handed to developer without clarifications needed
  Validation: Spec reviewer (human) confirms it's implementable
```

### improve-skills/SKILL.md Tests

```
TEST: improve-skills-01-session-context
  Input: Session ID available in environment
  Expected: Skill reads and analyzes session conversation
  Validation: Output references specific messages/exchanges from the session

TEST: improve-skills-02-missing-session-id
  Input: ${CLAUDE_SESSION_ID} is undefined
  Expected: Skill fails with clear error or provides workaround
  Validation: Error message suggests how to provide session context

TEST: improve-skills-03-empty-session
  Input: Valid session ID but no conversation history
  Expected: Skill handles empty session gracefully
  Validation: Output explains that no improvements were found

TEST: improve-skills-04-pattern-detection
  Input: Session with clear positive and negative patterns
  Expected: Skill identifies patterns matching the checklist
  Validation: Output categorizes findings as negative (-), neutral (?), positive (+)

TEST: improve-skills-05-tradeoff-analysis
  Input: Potential improvements with multiple implementation approaches
  Expected: Skill analyzes context cost vs. improvement benefit
  Validation: Output explains why some improvements are/aren't worth making

TEST: improve-skills-06-nonexistent-features
  Input: Session used undefined frontmatter (user-invocable, context: fork, etc.)
  Expected: Skill detects and documents unsupported features
  Validation: Output flags these as "not implemented yet" or "investigate"
```

---

## 3. Edge Cases Not Covered

### sdd-plan/SKILL.md

1. **Timeout/Token Limit Scenarios**
   - Very large repositories with hundreds of files
   - Complex requests requiring extensive exploration
   - Test case: How does skill behave when exploring a 100MB codebase?

2. **Contradictory Requirements**
   - User provides conflicting functional/non-functional requirements
   - Different subagents propose fundamentally incompatible approaches
   - Test case: Request for "maximum security" + "maximum performance" with actual tradeoff needed

3. **Incomplete Exploration**
   - Repository is partially available (due to access restrictions)
   - Some SPEC files referenced by user request are missing
   - Test case: User request mentions files that don't exist in repo

4. **Scope Creep**
   - User request evolves during planning
   - New stakeholder perspectives emerge during subagent feedback
   - Test case: User adds new requirements after phase 3

5. **Subagent Consensus Failure**
   - All 6 subagents disagree fundamentally
   - No clear tradeoff resolution possible
   - Test case: Security specialist wants to disable all features, product manager wants maximum features

### improve-skills/SKILL.md

1. **Session Data Corruption**
   - Session ID is valid but conversation history is malformed
   - Conversation contains binary or non-text data
   - Test case: What happens if session contains images or code artifacts?

2. **Pattern Ambiguity**
   - Conversation contains mixed signals (positive and negative in same exchange)
   - Patterns are subtle or context-dependent
   - Test case: User says "that's all" after pointing out a bug (is this positive or negative?)

3. **Feature References to Nonexistent Things**
   - Session analyzed a skill using undefined frontmatter options
   - Session discussed features that don't exist in Claude Code
   - No clear way to validate if feature references are real
   - Test case: Detected feature `argument-hint` in session - is this a real feature or misunderstanding?

4. **Context Cost Underestimation**
   - Proposed improvement requires detailed context addition
   - Improvement seems valuable but testing shows it hurts performance
   - Test case: How to balance "improvement quality" vs "added context burden"?

5. **Session Too Long**
   - Conversation has 100+ messages
   - Analyzing entire session becomes impractical
   - Test case: Should skill sample the session or analyze comprehensively?

---

## 4. Previously Unnoticed Tradeoffs

### sdd-plan/SKILL.md

| Tradeoff | Option A | Option B | Current State |
|----------|----------|----------|---|
| **Subagent invocation strategy** | Run all 6 subagents in parallel (fast, high context usage) | Run subagents sequentially (slower, lower context) | Unspecified - plan doesn't detail this |
| **Model selection per subagent** | Use Haiku for all (fast, cheaper, less capable) | Use Opus for all (slower, more capable) | Unspecified - plan suggests Haiku but no guidance |
| **Plan iteration depth** | 2 rounds of feedback (current) | N rounds until convergence | Fixed at 2 - no early stopping mechanism |
| **Exploration scope** | Broad initial exploration (may miss details) | Deep focused exploration (may miss breadth) | Unspecified - depends on user request |

**Quality Impact**: Without explicit tradeoff decisions, SDD output quality is inconsistent and depends entirely on which model/subagent strategy happens to be used.

### improve-skills/SKILL.md

| Tradeoff | Option A | Option B | Current State |
|----------|----------|----------|---|
| **Session scope** | Analyze only recent messages (limited context) | Analyze entire session history (high context) | Unspecified - assumes full history |
| **Improvement specificity** | Generic improvements (apply broadly) | Specific improvements (narrow, high-impact) | Mixed - examples are GitLab-specific |
| **Risk tolerance** | Propose only low-risk improvements (safe) | Propose all improvements including breaking changes | Unspecified |

**Quality Impact**: Improvement recommendations may not be actionable because the tradeoff between brevity and completeness is undefined.

---

## 5. Functional Requirements (QA Perspective)

### FR-QA-01: Skill Invocation Validation
**Requirement**: Skills must validate their preconditions at startup.

- sdd-plan requires: valid user request, accessible repository
- improve-skills requires: valid session ID, session history readable
- **Test**: Verify each skill checks preconditions and fails gracefully if missing

**AC1**: If user request is empty, return error within 2 seconds
**AC2**: If session ID is invalid, suggest valid format

### FR-QA-02: Output Validation
**Requirement**: Skills must produce structured, validated output.

- sdd-plan output must have valid markdown with all required sections
- improve-skills output must categorize findings (-, ?, +)
- **Test**: Parse output as markdown and validate structure

**AC1**: All H1 and H2 headers are present in output
**AC2**: No undefined references to features

### FR-QA-03: Subagent Reproducibility
**Requirement**: Running the same skill with same input should produce similar quality output.

- **Test**: Run sdd-plan twice with identical request, compare output structure

**AC1**: Both runs have same major sections
**AC2**: Both runs identify same key tradeoffs

### FR-QA-04: Error Recovery
**Requirement**: Skills must handle runtime errors gracefully.

- If exploration fails, provide partial results with explanation
- If subagent invocation fails, document which subagent failed
- **Test**: Inject failures at each phase and verify error handling

**AC1**: Partial output is better than no output
**AC2**: Error messages identify the failure point

---

## 6. Non-Functional Requirements (QA Perspective)

### NFR-QA-01: Testability
Skills should be designed to be testable without extensive manual interaction.

- Acceptance criteria must be machine-checkable (not just "human reviews it")
- Output should be machine-parseable (structured markdown, not free text)
- **Measurement**: % of checks that can be automated

### NFR-QA-02: Determinism
Running a skill twice with identical context should produce similar outputs.

- Not identical (AI is probabilistic) but structurally equivalent
- **Measurement**: Structural similarity score (90%+ on repeated runs)

### NFR-QA-03: Failure Detection
Users should know when output quality is degraded.

- Skill should flag uncertainty ("multiple valid approaches exist")
- Skill should indicate incomplete exploration ("repo too large, sampled 50%")
- **Measurement**: % of outputs with explicit uncertainty flags

### NFR-QA-04: Documentation
Skills should be self-documenting about their limitations.

- Each skill should document: preconditions, assumptions, tradeoffs, limitations
- **Measurement**: No critical assumption is left implicit

---

## 7. Quality Assurance Concerns & Warnings

### CRITICAL CONCERNS

#### 1. Undefined Feature References (BLOCKING)
**Issue**: improve-skills/SKILL.md references features that don't exist in Claude Code:
- `${CLAUDE_SESSION_ID}` (line 21) - variable availability unknown
- `user-invocable: false` (line 133, 148) - not in current spec
- `context: fork` (line 149) - not in current spec
- `argument-hint` (line 114) - not in current spec
- Dynamic context injection with bang+backtick (lines 106-113) - not documented as supported

**Impact**:
- Skills may fail silently when these features are invoked
- Users get no error message, just no output
- Testing is impossible because success criteria are ambiguous

**Remediation**:
1. Validate all feature references against actual Claude Code API
2. Create explicit test to verify each feature is available
3. Add fallback behavior for missing features
4. Document feature availability and version requirements

---

#### 2. No Success Criteria (BLOCKING)
**Issue**: Both skills lack explicit success/completion criteria.

For sdd-plan:
- How do you know when planning is "done"?
- What makes a plan "good"?
- When should iteration stop?

For improve-skills:
- What qualifies as a valid improvement?
- How many improvements should be found?
- What's the difference between "no improvements" and "skill failed"?

**Impact**:
- No way to validate if skill output is correct
- Users can't judge if they should try again
- Impossible to write automated tests

**Remediation**:
1. Define explicit completion criteria for each phase
2. Add assertion checks to output (e.g., "Spec contains X sections")
3. Create acceptance test template

---

#### 3. Typo in sdd-plan (line 209) - "secrity specialist"
**Issue**: Line 209 has typo "secrity" should be "security specialist"

**Impact**:
- When skill references this role, it may fail to invoke the correct subagent
- Low severity but indicates lack of review

**Remediation**:
- Fix typo immediately
- Add spell-check to CI/CD

---

### HIGH PRIORITY CONCERNS

#### 4. No Version Pinning for Subagent Models
**Issue**: sdd-plan instructs to use "Haiku" vs "Sonnet" but doesn't specify versions.

**Impact**:
- Different model versions may produce different quality results
- No way to reproduce a specific planning session
- Updates to Claude models could break existing workflows

**Remediation**:
1. Specify exact model IDs (e.g., "claude-opus-4-5-20251101")
2. Document which models are compatible
3. Add version check to skill startup

---

#### 5. No Timeout/Token Budget Controls
**Issue**: Skills have no explicit bounds on execution.

For sdd-plan:
- Large repos + 6 subagents + 2 rounds = potentially huge token usage
- No way to limit context or execution time

For improve-skills:
- Very long sessions could exceed token budget
- No sampling or summarization strategy

**Impact**:
- Skills could fail silently due to token limits
- Users get incomplete output without knowing why
- Cost is unpredictable

**Remediation**:
1. Add explicit token budget checks
2. Implement graceful degradation (summarize instead of failing)
3. Document estimated token usage for different input sizes
4. Add progress reporting during long operations

---

#### 6. Subagent Coordination Risk
**Issue**: sdd-plan relies on coordinating 6 independent subagents but has no synchronization mechanism.

**Impact**:
- If one subagent fails, whole planning fails
- No way to retry a failed subagent
- No way to validate all subagents actually ran

**Remediation**:
1. Add health check for each subagent invocation
2. Implement retry logic (up to N attempts)
3. Document fallback if subagent is unavailable
4. Add explicit validation that all 6 subagents provided feedback

---

#### 7. Session Context Dependency (improve-skills only)
**Issue**: improve-skills depends on `${CLAUDE_SESSION_ID}` which has no clear:
- Definition (what is a "session"?)
- Availability (when is this variable populated?)
- Fallback (what if it's undefined?)

**Impact**:
- Skill may fail silently
- Users won't know they need to invoke it differently
- No way to call skill from other contexts

**Remediation**:
1. Document exact variable name and how to set it
2. Add precondition check at startup
3. Provide clear error message if unavailable
4. Consider alternative invocation methods (pass session ID as argument)

---

### MEDIUM PRIORITY CONCERNS

#### 8. No Regression Test Suite
**Issue**: No automated tests to prevent regressions.

**Impact**:
- Typo fixes might accidentally change skill behavior
- Feature additions might break existing workflows
- No CI/CD validation

**Remediation**:
1. Create golden file tests (save expected outputs)
2. Add basic smoke tests (skill runs without error)
3. Verify output structure (required sections present)
4. Set up CI to run tests on every commit

---

#### 9. No Documentation for Edge Cases
**Issue**: Skills don't document how they handle edge cases.

**Impact**:
- Users don't know what to do if skill fails
- No troubleshooting guide
- Users assume it's broken when actually they're using it wrong

**Remediation**:
1. Document common failure modes for each skill
2. Add troubleshooting section to SKILL.md
3. Provide example invocations
4. Document assumptions and limitations

---

#### 10. No Telemetry/Observability
**Issue**: Skills have no way to report success/failure metrics.

**Impact**:
- Can't identify if skills are actually being used
- Can't track which phases fail most often
- Can't measure quality improvements

**Remediation**:
1. Add explicit logging/success indicators
2. Log which subagents ran and their quality
3. Track phase completion times
4. Measure output quality metrics

---

## 8. Test Implementation Strategy

### Phase 1: Precondition Checks (Atomic Tests)

```
TEST SUITE: sdd-plan-preconditions
├── TEST: Empty user request fails
├── TEST: Invalid request format is detected
├── TEST: Accessible repo is detected
└── TEST: Missing repo is detected

TEST SUITE: improve-skills-preconditions
├── TEST: Session ID available
├── TEST: Session ID missing (graceful error)
└── TEST: Empty session (graceful error)
```

### Phase 2: Output Structure Validation (Atomic Tests)

```
TEST SUITE: sdd-plan-output-structure
├── TEST: All required markdown sections present
├── TEST: No undefined feature references
├── TEST: Plan is internally consistent
└── TEST: Plan is implementable (no TODOs in final spec)

TEST SUITE: improve-skills-output-structure
├── TEST: Findings are categorized (-, ?, +)
├── TEST: No invalid pattern flags
└── TEST: Improvement proposals are specific
```

### Phase 3: Behavior Validation (Integration Tests)

```
TEST SUITE: sdd-plan-behavior
├── TEST: Exploration phase finds relevant files
├── TEST: All 6 subagents provide feedback
├── TEST: Subagent feedback informs final spec
└── TEST: Final spec reflects stakeholder tradeoffs

TEST SUITE: improve-skills-behavior
├── TEST: Session patterns are identified
├── TEST: Improvement context cost is analyzed
└── TEST: Recommendations are actionable
```

### Phase 4: Regression Tests (Golden File Tests)

```
Store reference outputs:
├── tests/golden/sdd-plan-simple-request.md
├── tests/golden/sdd-plan-complex-request.md
├── tests/golden/improve-skills-positive-session.md
└── tests/golden/improve-skills-negative-session.md

Run skill, compare output structure to golden file
```

---

## 9. Recommendations Summary

| Priority | Area | Action | Success Metric |
|----------|------|--------|---|
| P0 | Feature validation | Validate all feature references exist in Claude Code | 100% of features documented as supported/unsupported |
| P0 | Success criteria | Define explicit completion criteria for each skill | All acceptance criteria are testable |
| P0 | Error handling | Add precondition validation at skill startup | Skills fail fast with clear error messages |
| P1 | Regression testing | Create golden file test suite | 90%+ automated test coverage |
| P1 | Documentation | Document limitations, assumptions, edge cases | No critical assumptions are implicit |
| P1 | Timeout controls | Add token budget and timeout handling | Skills degrade gracefully under load |
| P2 | Telemetry | Add logging/observability | Can measure success rate of skill invocations |
| P2 | Examples | Add worked examples for common use cases | 100% of skill features have at least one example |

---

## 10. Test Checklist for Implementation

Before considering improvements complete, verify:

- [ ] All undefined feature references are resolved
- [ ] All skill preconditions are validated at startup
- [ ] Output format is validated (valid markdown, required sections)
- [ ] Success/completion criteria are explicitly defined
- [ ] At least one golden file test exists for each skill
- [ ] Edge cases have documented handling (no silent failures)
- [ ] Typos are fixed (secrity -> security)
- [ ] Error messages are clear and actionable
- [ ] Subagent invocation is bulletproof (health checks, retry logic)
- [ ] Token budget is bounded and documented
- [ ] All assumptions are documented
- [ ] At least one integration test verifies end-to-end flow

---

## Conclusion

The sdd-plan and improve-skills are well-conceived but lack the testing infrastructure and explicit success criteria needed for production use. The most critical gaps are:

1. **Undefined feature references** that make the skills untestable
2. **Missing success criteria** that prevent validation
3. **No error handling** for common failure modes
4. **No regression tests** to prevent degradation

With focused effort on these areas, both skills can become robust and maintainable tools.
