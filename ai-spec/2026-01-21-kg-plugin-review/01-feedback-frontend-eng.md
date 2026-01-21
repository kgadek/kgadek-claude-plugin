# Frontend Engineer Review: kg Plugin Skills UX & Clarity

**Perspective:** Frontend/UX Engineer
**Focus:** User-facing components, interaction design, clarity, and documentation quality
**Skills reviewed as:** "User interfaces" - how users interact with Claude Code capabilities

---

## Executive Summary

The two skills (`sdd-plan` and `improve-skills`) have solid conceptual foundations, but suffer from **clarity issues, undefined feature references, and unclear user workflows** that create friction. Treating skills as "user interfaces" reveals several UX problems that need attention before production use.

**Key finding:** The skills reference features that may not exist (`${CLAUDE_SESSION_ID}`, `disable-model-invocation`, `user-invocable`, `context: fork`, `argument-hint`), creating confusion about what's actually available vs. planned.

---

## 1. Are the skills user-friendly and clear?

### 1.1 `sdd-plan` Skill: MODERATE - Good structure, unclear expectations

**Strengths:**
- Clear role definition (software architect/planner)
- Well-organized sections (understand → explore → plan → refine)
- Good use of markdown formatting and structure
- The SPEC directory anatomy example is helpful

**Weaknesses:**
- **Cognitive load:** 221 lines is substantial; first-time users won't know what to expect
- **Unclear subagent invocation:** Section mentions running "specialised subagents in parallel" but provides NO guidance on HOW to invoke them
  - Users won't know if they should use the Task tool, create new threads, or if Claude handles this automatically
  - The instructions say "run specialised subagents" but the implementation mechanism is invisible
- **Model selection ambiguity:** No guidance on which Claude model to use (Haiku vs. Sonnet vs. Opus)
- **Entry point confusion:** Users must decide between following the SDD process exactly vs. adapting it; unclear where flexibility exists
- **No failure path:** What happens if exploration reveals the task is infeasible? No guidance provided

### 1.2 `improve-skills` Skill: LOW - Multiple blockers to clarity

**Critical clarity issues:**
1. **Undefined variable:** `${CLAUDE_SESSION_ID}` on line 21
   - Blocks understanding of the core premise
   - User can't even verify if the skill can work as written
   - **UX impact:** Immediate confusion on first read

2. **Non-existent features referenced:**
   - `disable-model-invocation: true` (line 4) - no explanation of what this does
   - `user-invocable: false` (lines 133, 148) - not documented anywhere
   - `context: fork` (line 149) - not documented anywhere
   - `argument-hint` (line 114) - not documented anywhere
   - **UX impact:** Reads like a spec for planned features, not a usable skill

3. **Balance instruction unclear:** Lines 51-65 discuss balancing general vs. detailed improvements, but the actual guidance is vague
   - What defines "rare"? What defines "worth improving"?
   - No concrete decision framework provided

4. **Example is GitLab-specific:** The extended example (lines 68-180) uses GitLab terminology exclusively
   - Users without GitLab context will struggle to extract the pattern
   - **UX impact:** Reduces accessibility for non-GitLab users

5. **Missing context on session reading:**
   - How does the skill access conversation history?
   - Can it read arbitrary sessions or only the current one?
   - How much history is available?
   - **UX impact:** Prevents understanding of scope and limitations

---

## 2. Is the output/interaction well-designed?

### 2.1 `sdd-plan` Output Design

**Positive:**
- Creates a directory structure that organizes feedback naturally (`01-feedback-{role}.md`, `02-feedback-{role}.md`)
- Multiple review rounds built in (round 1, round 2)
- Creates auditable feedback trail

**Concerns:**
- **Parallelization clarity:** Mentions running subagents "in parallel" but unclear how long this takes or if users should wait
- **File naming convention:** Using `01-feedback-`, `02-feedback-` is ordinal, but discipline isn't established - what if there are 3+ rounds?
- **No template for subagent feedback:** Each subagent writes freely; no guidance on expected length, format, or sections
  - Results in inconsistent output quality
  - Users can't predict what they'll get

### 2.2 `improve-skills` Output Design

**Issues:**
- No clear output format specified
- Example shows markdown, but is that mandatory?
- No guidance on how findings should be presented to the user
- Unclear if the output is:
  - A PR review?
  - A written report?
  - Recommendations for the next session?
  - A skill modification specification?

---

## 3. What UX improvements could be made?

### 3.1 Clarity Improvements

**Priority 1: Remove/Clarify Feature References**

The skill references multiple undefined features. Suggested actions:

1. **For `${CLAUDE_SESSION_ID}` (improve-skills line 21):**
   - If available: Document what it returns and how to use it
   - If not available: Remove this line and explain how to get session context instead
   - **Suggested fix:** "The session context is automatically included in your message thread"

2. **For frontmatter features** (`disable-model-invocation`, `user-invocable`, `context: fork`, `argument-hint`):
   - **Best option:** Create a separate `SKILL-FRONTMATTER.md` document that lists:
     - Supported frontmatter keys
     - Which are implemented vs. planned
     - Examples of each
   - **Fallback:** Mark as "PLANNED FEATURE" in the skill itself:
     ```yaml
     # PLANNED: argument-hint parameter (not yet supported)
     # argument-hint: "[mr-iid]"
     ```

### 3.2 Cognitive Load Reduction

**For `sdd-plan`:**
- Add a "Quick reference" section at the top (< 50 lines) summarizing:
  - What this skill does in 1 sentence
  - What users should provide
  - What they'll get back
  - How long it takes (estimate)

**Example:**
```markdown
## Quick Start

**What:** This skill structures a complex planning task through Specification-Driven Development.
**Provide:** A user request or problem statement.
**Get:** A comprehensive SPEC document with feedback from 6 specialized reviewers.
**Time:** 5-10 minutes (depends on complexity and parallelization).
**Output:** Created in `ai-spec/{date}-{description}/` directory.
```

### 3.3 Better Entry Point Design

**For `sdd-plan`:**
- Add a "When to use this skill" section:
  - ✓ Complex features requiring multiple perspectives
  - ✓ Architectural decisions needing validation
  - ✓ High-stakes changes with many moving parts
  - ✗ Simple bug fixes or one-line changes
  - ✗ When you just need a quick answer

**For `improve-skills`:**
- Add a "How to trigger this" section:
  - Do users call it after completing a task?
  - Do they provide arguments?
  - What context should they provide?

### 3.4 Standardize Subagent Feedback

**Create a template for consistency:**

```markdown
## {Agent Role} Feedback

### Summary
[1-2 sentence overview of this agent's main concern]

### Key Findings
- Finding 1
- Finding 2
- Finding 3

### Recommendations
- Recommendation 1 (impact: X)
- Recommendation 2 (impact: Y)

### Concerns & Tradeoffs
- Concern 1 (mitigation: ...)
- Concern 2 (mitigation: ...)
```

This ensures output is scannable and consistent.

---

## 4. What UX tradeoffs are unnoticed?

### 4.1 Completeness vs. Context Pollution

**Issue:** `sdd-plan` runs 6 specialized agents with full context each round
- **Cost:** High token usage for each agent
- **Benefit:** Comprehensive feedback
- **Tradeoff:** For small changes, this might be overkill

**Suggestion:** Add a "scope selector" option:
- `scope: lightweight` - run 2 agents (architect, primary role)
- `scope: standard` - run 4-5 agents (current setup)
- `scope: comprehensive` - run all 6 agents (current setup)

### 4.2 Flexibility vs. Guardrails

**Issue:** `improve-skills` is vague about what constitutes an "improvement"
- **Cost:** Users unsure what to expect; could propose silly changes
- **Benefit:** Flexibility for different use cases
- **Tradeoff:** Results in inconsistent quality

**Suggestion:** Provide explicit improvement categories:
```markdown
## Improvement Categories

1. **Critical Issues** - Skill doesn't work as documented
2. **Usability Issues** - Common user errors or confusion points
3. **Performance Issues** - Unnecessary token usage
4. **Generalization** - Extracting domain-specific patterns
5. **Edge Cases** - New use cases that should be covered
```

### 4.3 Session Scope Ambiguity

**Issue:** `improve-skills` references `${CLAUDE_SESSION_ID}` but doesn't explain session boundaries
- **Cost:** Users can't predict what will be analyzed
- **Benefit:** Automatic context gathering

**Suggestion:** Clarify explicitly:
```markdown
## What Gets Analyzed

This skill analyzes the current session:
- All messages from the start of this conversation thread
- Limited to the last N messages (if session is long) [DEFINE N]
- Includes code changes, command outputs, and user feedback
- Does NOT include previous sessions or other branches
```

---

## 5. Functional and Non-functional Requirements

### Functional Requirements to Add

**FR-UX-1: Feature reference clarity**
- All frontmatter keys must be documented in a central reference
- Mark planned features explicitly
- Remove references to undefined features or clarify their status

**FR-UX-2: Skill templates and output standardization**
- Create reusable templates for subagent feedback
- Standardize output format for `improve-skills`
- Define expected length and structure

**FR-UX-3: Quick reference guides**
- Add "Quick Start" section to each skill (< 50 lines)
- Include: Purpose, inputs, outputs, time estimate
- Add "When to use" guidance

**FR-UX-4: Better error guidance**
- Document what happens if subagent runs fail
- Provide fallback options (e.g., "Run with 2 agents instead of 6")
- Clear recovery steps

### Non-Functional Requirements to Add

**NFR-UX-1: Discoverability**
- Skill names should be self-explanatory to newcomers
- Description in marketplace.json should answer: "What do I get if I use this?"
- Current: "Start a SDD workflow" - unclear what SDD means to casual user

**NFR-UX-2: Consistency**
- All skills should have consistent structure
- Same section ordering
- Same frontmatter documentation

**NFR-UX-3: Progressive Disclosure**
- Basic quick-start first (< 2 minutes to understand)
- Detailed documentation available but not mandatory
- Examples before edge cases

**NFR-UX-4: Scannability**
- Use headers consistently
- Use bullet points for lists
- Highlight key terms and limitations

**NFR-UX-5: Predictability**
- Users should know what they'll get before running a skill
- Time estimates provided
- Output location documented
- Success criteria clear

---

## 6. Concerns and Warnings

### 6.1 Undefined Feature References (CRITICAL)

**Risk:** Users will see feature names (`user-invocable`, `context: fork`) and either:
1. Assume they don't work (frustration)
2. Try to use them (confusion)
3. Propose improvements based on them (wasted effort)

**Mitigation:**
- Create definitive documentation of supported features
- Update skills to remove or explicitly mark unsupported features
- Consider a feature roadmap that users can reference

### 6.2 `${CLAUDE_SESSION_ID}` Variable Uncertainty (CRITICAL)

**Risk:** The entire `improve-skills` premise depends on this variable existing
- If it doesn't exist, the skill is non-functional
- If it exists but isn't documented, users will be confused

**Mitigation:**
- Verify variable existence before shipping
- Document how to access session context if variable doesn't exist
- Add error message if variable is undefined

### 6.3 Cognitive Load on First Use (HIGH)

**Risk:** New users might be overwhelmed by 221 lines of `sdd-plan` instructions
- May abandon the skill without trying it
- Might use it incorrectly due to misunderstanding

**Mitigation:**
- Create a "Getting Started" guide separate from the full skill
- Add visual examples of output directory structure
- Consider a "lightweight mode" for simple planning tasks

### 6.4 GitLab-Specific Examples (MEDIUM)

**Risk:** `improve-skills` uses GitLab terminology exclusively in its example
- Users on GitHub/Gitea/Bitbucket won't see themselves in the example
- Pattern extraction becomes harder for non-GitLab users

**Mitigation:**
- Add 1-2 examples from other platforms (GitHub, generic code review)
- Use platform-agnostic terminology where possible
- Explicitly state: "This pattern applies to any VCS platform"

### 6.5 Parallelization Expectations (MEDIUM)

**Risk:** `sdd-plan` mentions "run specialised subagents in parallel" but:
- Unclear if this happens automatically or if user must configure it
- Unclear how long parallelization takes
- Unclear if there's a sequential fallback

**Mitigation:**
- Document the parallelization mechanism explicitly
- Add time estimate for both parallel and sequential execution
- Provide fallback if parallelization isn't available

### 6.6 Version Mismatch Risk (MEDIUM)

**Risk:** Features might be added/removed between plugin versions
- Users on 0.0.1 might see features documented for 0.0.2
- No versioning guidance in skill documentation

**Mitigation:**
- Add version numbers to feature documentation
- Update marketplace.json version after changes
- Consider adding version compatibility notes

---

## Summary: Top 3 Improvements (Priority Order)

### 1. **Fix Undefined Feature References** (BLOCKING)
   - Create `SKILL-FRONTMATTER.md` documenting all supported/planned features
   - Update skills to remove or mark undefined features
   - **Impact:** Makes skills immediately more credible

### 2. **Add Quick Start Sections** (HIGH VALUE)
   - 50-line overview for each skill
   - Purpose, inputs, outputs, time estimate
   - **Impact:** Reduces cognitive load and improves discoverability

### 3. **Clarify Session Context in improve-skills** (HIGH VALUE)
   - Verify `${CLAUDE_SESSION_ID}` exists and document it
   - Explain session boundaries and what gets analyzed
   - Add fallback if variable doesn't exist
   - **Impact:** Unblocks the core functionality of improve-skills

---

## Typo Found

- Line 209 in `sdd-plan/SKILL.md`: "secrity specialist" → "security specialist"

---

## Closing Notes

The skills have strong conceptual foundations (SDD is a legitimate planning methodology, skill improvement analysis is valuable). The barriers are purely **clarity and documentation**, not design flaws. These are high-impact, relatively low-effort fixes that will significantly improve the user experience.

The biggest UX win would be creating a single, definitive reference document for supported skill features. This unlocks confidence in the entire plugin.
