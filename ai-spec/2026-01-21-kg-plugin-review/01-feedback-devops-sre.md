# DevOps / SRE Review: kg Plugin Improvements

**Perspective:** Easy deployment, simple operations, observability, and CI/CD considerations

---

## 1. Deployment & Installation Reliability

### Current State
- Plugin is distributed via GitHub repository with version 0.0.1 in `marketplace.json`
- No CI/CD pipeline exists to validate plugin structure or marketplace.json correctness
- No automated tests to ensure skills load correctly
- Version management is manual

### Issues & Gaps

1. **No Validation of marketplace.json**
   - JSON syntax errors would only be caught when Claude Code loads the plugin
   - Missing fields or incorrect schema would fail silently until used
   - No schema validation documented

2. **No Plugin Integrity Checks**
   - No verification that all referenced files exist (skills point to non-existent paths)
   - No validation of SKILL.md frontmatter syntax
   - Broken references would only be discovered during skill invocation

3. **Missing Deployment Checkpoints**
   - No clear process for verifying a release is safe to deploy
   - No rollback procedures documented
   - No version tagging strategy in git

4. **License Compliance Risk**
   - AGPL-3.0-only is restrictive; unclear if this is intentional for a plugin
   - No tooling to verify license headers in source files
   - No distribution terms documented

### Recommendations

**Action 1.1: Create CI/CD Pipeline**
- Add GitHub Actions workflow to:
  - Validate `marketplace.json` against schema on every commit
  - Verify all SKILL.md files exist and are readable
  - Lint SKILL.md frontmatter for required fields
  - Validate JSON syntax and format
  - Run spell-check on documentation

**Action 1.2: Add Plugin Validation Script**
```bash
# plugins/kg/validate.sh
#!/bin/bash
set -e

# Check marketplace.json exists and is valid JSON
jq empty ".claude-plugin/marketplace.json" || exit 1

# Check all skills referenced in marketplace exist
PLUGIN_PATH="plugins/kg"
if [ -d "$PLUGIN_PATH" ]; then
  [ -d "$PLUGIN_PATH/skills" ] || exit 1

  # Verify each skill has SKILL.md
  find "$PLUGIN_PATH/skills" -mindepth 1 -maxdepth 1 -type d | while read skill_dir; do
    [ -f "$skill_dir/SKILL.md" ] || { echo "Missing SKILL.md in $skill_dir"; exit 1; }
  done
fi

echo "Plugin validation passed"
```

**Action 1.3: Implement Semantic Versioning**
- Use git tags for releases (e.g., `v0.0.2`, `v0.1.0`)
- Update version in `marketplace.json` on each release
- Document versioning policy in repository

**Action 1.4: Document Rollback Procedure**
- Create `docs/operations/rollback.md`:
  - How to revert to a previous plugin version
  - Git tags reference
  - Clearing cache procedures for Claude Code

---

## 2. Operational Considerations

### Observability Gaps

1. **No Skill Invocation Logging**
   - No guidance on how to monitor skill usage
   - No error metrics defined
   - Cannot distinguish between skill failures and user errors
   - No way to track skill performance

2. **No Health Checks**
   - No mechanism to verify plugin health
   - No self-test capability
   - No heartbeat or smoke test defined

3. **No Documentation of Common Failures**
   - Users won't know what to do if a skill fails
   - No troubleshooting guide
   - No error message explanations

### Recommendations

**Action 2.1: Add Operational Documentation**
Create `docs/operations/README.md`:
- How skills are loaded and executed
- Expected behavior and outputs
- Common failure modes and resolutions
- Debug procedures

**Action 2.2: Add Skill Runtime Guidance**
For each skill (sdd-plan, improve-skills), add:
- Expected execution time
- Resource requirements (context window, token usage)
- Common error scenarios and recovery steps
- Prerequisites or dependencies

**Action 2.3: Document Session State Requirements**
- Clarify what state each skill requires (e.g., improve-skills needs `${CLAUDE_SESSION_ID}`)
- Document fallback behavior if state unavailable
- Add warnings for incomplete sessions

---

## 3. CI/CD Improvements

### Current State
- 5 commits in git history with informal commit messages
- No automated quality gates
- No deployment artifacts created
- No release notes generated

### Issues & Gaps

1. **Missing Quality Gates**
   - No requirement to pass validation before merge
   - No documentation requirements checked
   - No version bump requirement

2. **No Automated Testing**
   - Skills cannot be automatically executed in test environment
   - No validation that skills produce expected outputs
   - No regression testing between versions

3. **No Release Management**
   - No release checklist
   - No changelog generation
   - No deployment coordination

4. **No Dependency Management**
   - No explicit dependencies documented
   - Claude Code version compatibility unclear
   - No minimum version requirements specified

### Recommendations

**Action 3.1: Create GitHub Actions Workflow**
```yaml
# .github/workflows/validate.yml
name: Plugin Validation
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Validate marketplace.json
        run: jq empty .claude-plugin/marketplace.json
      - name: Validate SKILL.md files
        run: |
          find plugins -name "SKILL.md" | while read f; do
            [ -f "$f" ] || exit 1
          done
      - name: Check for required documentation
        run: |
          [ -f "README.md" ] || exit 1
          [ -f "CLAUDE.md" ] || exit 1
          [ -f "docs/operations/README.md" ] || exit 1
```

**Action 3.2: Establish Release Checklist**
Create `.github/RELEASE_CHECKLIST.md`:
- [ ] All tests passing
- [ ] Documentation updated
- [ ] Changelog entry created
- [ ] Version bumped in marketplace.json
- [ ] Git tag created
- [ ] Release notes published

**Action 3.3: Define Compatibility Matrix**
Create `docs/compatibility.md`:
```
# Compatibility Matrix

| kg Plugin Version | Claude Code Min Version | Claude Model Support |
|------------------|-------------------------|----------------------|
| 0.0.x            | 1.0.0+                  | Haiku, Opus 4.5      |
| 1.0.0            | 2.0.0+                  | Haiku, Opus 4.5+     |
```

**Action 3.4: Add Pre-commit Hooks**
Create `.pre-commit-config.yaml`:
```yaml
repos:
  - repo: local
    hooks:
      - id: validate-plugin
        name: Validate plugin structure
        entry: ./plugins/kg/validate.sh
        language: script
        pass_filenames: false
      - id: check-marketplace-json
        name: Check marketplace.json is valid
        entry: bash -c 'jq empty .claude-plugin/marketplace.json'
        language: system
        files: marketplace.json$
```

---

## 4. Previously Unnoticed Tradeoffs

### 4.1 AGPL-3.0 License
**Tradeoff:** Using AGPL-3.0-only on a Claude Code plugin creates friction for users
- **Risk:** Corporate users cannot use in closed-source environments
- **Impact:** Limits adoption to open-source projects
- **Alternative:** Consider MIT, Apache 2.0, or GPL-3.0 (non-AGPL variant)
- **Action:** Clarify licensing intent; document if this is intentional

### 4.2 Subagent Invocation Pattern
**Tradeoff:** The sdd-plan skill uses subagent-based planning which is resource-intensive
- **Cost:** Multiple model invocations (haiku + sonnet layers) consume significant tokens
- **Benefit:** Higher quality, more thorough planning
- **Constraint:** Users pay for thoroughness; cannot opt for faster/cheaper alternative
- **Action:** Document cost implications; consider adding lightweight planning mode

### 4.3 Session-Dependent Skills
**Tradeoff:** improve-skills requires `${CLAUDE_SESSION_ID}` but fails silently if unavailable
- **Risk:** Skill may appear to work but not function correctly
- **Impact:** Hard to debug; user confusion
- **Action:** Add explicit validation with helpful error messages

### 4.4 Non-Existent Feature References
**Tradeoff:** Mentioning planned features (user-invocable, context: fork, argument-hint)
- **Risk:** Confusion about current capabilities vs. future roadmap
- **Impact:** Users may spend time on unsupported features
- **Action:** Remove or clearly mark as "planned for future version"

---

## 5. Functional & Non-Functional Requirements

### New Functional Requirements

**FR-DEPLOY-1: Plugin Validation**
- The plugin must provide a validation script that checks:
  - marketplace.json is valid JSON and matches schema
  - All referenced SKILL.md files exist
  - All skill frontmatter has required fields
  - Exit code 0 on success, non-zero on failure

**FR-DEPLOY-2: Version Compatibility**
- Plugin must document minimum Claude Code version required
- Plugin must document compatible Claude models
- Marketplace.json must include version metadata

**FR-DEPLOY-3: Installation Instructions**
- Repository must include clear, step-by-step installation guide
- Guide must cover both manual and package manager installation
- Guide must include verification steps

**FR-OPS-1: Skill Runtime Documentation**
- Each skill must document:
  - Expected execution time range
  - Approximate token usage
  - Prerequisites (environment, state, permissions)
  - Common failure scenarios and recovery steps

**FR-OPS-2: Error Handling Guidance**
- Skills must provide clear error messages
- Error messages must indicate recovery action
- Documentation must explain all possible error states

**FR-RELEASE-1: Versioning**
- Plugin uses semantic versioning (MAJOR.MINOR.PATCH)
- Versions are tagged in git with 'v' prefix
- marketplace.json version matches git tag

**FR-RELEASE-2: Changelog**
- Repository maintains CHANGELOG.md
- Each version documents new features, fixes, breaking changes
- Changelog links to relevant git commits

### New Non-Functional Requirements

**NFR-DEPLOY-1: Deployment Reliability**
- Deployment validation must complete in < 30 seconds
- Plugin validation must be deterministic (no flakiness)
- Error messages must be actionable (not generic)

**NFR-OPS-1: Observability**
- Skills must provide clear success/failure indicators
- Skill output must be structured and parseable
- Documentation must explain expected output format

**NFR-OPS-2: Troubleshooting**
- Plugin must include troubleshooting guide
- Common issues must be documented with solutions
- Debug mode or verbose output must be available

**NFR-RELEASE-1: Backward Compatibility**
- Patch versions must be backward compatible
- Minor versions may add features but remain compatible
- Major versions may introduce breaking changes (documented)

**NFR-RELEASE-2: Release Velocity**
- Releases should be processable in < 5 minutes
- Release process must be fully automated except approval
- No manual steps should be required post-approval

---

## 6. Operational Concerns & Warnings

### 6.1 Critical: Unclear Variable Interpolation
**Issue:** `${CLAUDE_SESSION_ID}` in improve-skills skill
- **Risk:** Skill silently fails if variable not available
- **Impact:** Users cannot debug why skill isn't working
- **Severity:** HIGH
- **Action:**
  - Verify this variable is available in Claude Code
  - Add validation with error message if unavailable
  - Document fallback behavior

### 6.2 Critical: Non-Existent Feature References
**Issue:** Skills reference features not implemented (user-invocable, context: fork, argument-hint)
- **Risk:** Users follow documentation for features that don't exist
- **Impact:** Wasted user time, frustration, support burden
- **Severity:** HIGH
- **Action:**
  - Remove all references to non-existent features
  - Create separate "Roadmap" document if features planned
  - Update frontmatter to only use verified features

### 6.3 Critical: No Rollback Mechanism
**Issue:** Plugin version 0.0.1 with no tagged releases or rollback procedure
- **Risk:** If deployed version has issues, no clear way to revert
- **Impact:** Production outage duration increases
- **Severity:** HIGH
- **Action:**
  - Implement git tagging for releases
  - Document rollback procedure
  - Test rollback procedure before first production deployment

### 6.4 Warning: License Restrictiveness
**Issue:** AGPL-3.0-only license on a tool plugin
- **Risk:** Corporate users cannot use (legal risk)
- **Impact:** Limited adoption, potential GPL violations
- **Severity:** MEDIUM
- **Action:**
  - Review licensing intent with stakeholders
  - Consider more permissive alternative (MIT, Apache 2.0)
  - Document if AGPL is intentional

### 6.5 Warning: Subagent Cost Not Controlled
**Issue:** sdd-plan skill spawns multiple subagents with no cost awareness
- **Risk:** Users may run up unexpected token bills
- **Impact:** User dissatisfaction, cost overruns
- **Severity:** MEDIUM
- **Action:**
  - Document estimated token costs in skill description
  - Add cost warning in skill introduction
  - Consider adding cost-aware mode

### 6.6 Warning: No Monitoring of Skill Failures
**Issue:** No way to detect if skills are failing in production
- **Risk:** Issues go unnoticed until user reports them
- **Impact:** Reactive vs. proactive incident response
- **Severity:** MEDIUM
- **Action:**
  - Add guidance for users to report issues
  - Create template for bug reports
  - Document monitoring integration points

### 6.7 Warning: Version Skew Risk
**Issue:** marketplace.json version not synchronized with anything
- **Risk:** Version in marketplace.json diverges from actual code
- **Impact:** Distribution of wrong version, confusion
- **Severity:** MEDIUM
- **Action:**
  - Enforce version sync in CI/CD
  - Require version bump in marketplace.json for releases
  - Tag git commits with version

### 6.8 Information: Type Checking Not Available
**Issue:** No way to validate skill arguments at load time
- **Risk:** Invalid arguments pass through to runtime
- **Impact:** Harder debugging of skill misuse
- **Severity:** LOW
- **Workaround:** Document expected argument format clearly
- **Action:** Add validation examples to skill documentation

---

## 7. Deployment Roadmap (Prioritized)

### Phase 0: Foundation (CRITICAL - Do before any deployment)
1. Tag current commit as v0.0.1 in git
2. Verify marketplace.json schema is correct
3. Add CI/CD validation pipeline (GitHub Actions)
4. Add plugin validation script
5. Establish rollback procedure

### Phase 1: Documentation (Before v0.0.2 release)
1. Create README.md with installation instructions
2. Create docs/operations/README.md with operational guidance
3. Create docs/compatibility.md with version matrix
4. Create CLAUDE.md with contribution guidelines
5. Add troubleshooting section to each skill

### Phase 2: Reliability (For v0.0.2)
1. Remove non-existent feature references
2. Add validation for ${CLAUDE_SESSION_ID} variable
3. Fix typos and documentation gaps
4. Update marketplace.json to v0.0.2
5. Tag v0.0.2 release in git

### Phase 3: Observability (For v0.1.0)
1. Add structured error handling to skills
2. Document expected output format
3. Add verbose/debug mode
4. Create monitoring integration guide
5. Establish feedback collection mechanism

### Phase 4: Automation (For v1.0.0)
1. Fully automated release pipeline
2. Pre-commit hooks for validation
3. Automated changelog generation
4. Automated compatibility testing
5. Automated performance benchmarking

---

## Summary

The kg plugin has a solid foundation with two well-designed skills, but lacks the operational infrastructure needed for reliable deployment and maintenance. Key gaps are:

1. **No CI/CD pipeline** - Risks deployment of broken versions
2. **No version management** - Makes rollback and compatibility tracking difficult
3. **References to non-existent features** - Creates user confusion
4. **Missing operational documentation** - Users don't know how to troubleshoot failures
5. **No defined rollback procedures** - Increases incident recovery time

Implementing the recommended actions in phases will transform this from a personal project into a production-ready plugin suitable for broader adoption.

**Recommended Next Steps:**
- Prioritize Phase 0 (foundation) - establish basic safety guardrails
- Complete Phase 1 & 2 before releasing v0.0.2
- Plan Phase 3 & 4 for future versions
