---
author: Opus 4.5
description: Security audit of the Superpowers repository checking for malicious payloads, suspicious code patterns, and supply chain risks
date: 2026-01-21T15:50:16+01:00
---

# Security Review: Superpowers Repository

## Summary

**Result: NO MALICIOUS PAYLOAD DETECTED**

The repository is **CLEAN** and contains legitimate, well-maintained open-source code.

---

## Repository Overview

| Field | Value |
|-------|-------|
| Name | Superpowers |
| Purpose | Software development workflow framework/plugin for AI coding agents |
| Author | Jesse Vincent (@obra) |
| License | MIT |
| Version | 4.0.3 |

Superpowers provides reusable skills, hooks, and commands that guide AI through professional development workflows including TDD, code review, systematic debugging, and more.

---

## Security Audit Results

### 1. Suspicious Code Patterns

| Check | Result |
|-------|--------|
| Obfuscated code (base64, eval, Function()) | Not found |
| Network calls to external servers | Not found |
| Destructive file operations (rm -rf, unlink) | Not found |
| Credential/environment exfiltration | Not found |
| Encoded/encrypted payloads | Not found |
| Malicious npm hooks (preinstall, postinstall) | Not found |

### 2. Shell Command Execution

Two files use `child_process.execSync()`:

| File | Line | Purpose | Risk |
|------|------|---------|------|
| `skills/writing-skills/render-graphs.js` | 18, 72-76, 112 | Graphviz diagram rendering | Low - hardcoded commands |
| `lib/skills-core.js` | 151 | Git repository status checking | Low - hardcoded commands |

**Assessment:** All shell execution is hardcoded (not user-controlled), properly error-handled, and serves legitimate development purposes.

### 3. Dependencies

- **No package.json** - This is not a traditional Node.js project
- **No npm dependencies** - No risk of typosquatting or supply chain attacks
- **No CI/CD pipelines** - Only `.github/FUNDING.yml` exists

### 4. Automatic Script Execution

Only one automatic execution point exists:

| Hook | File | Trigger | Action |
|------|------|---------|--------|
| Session start | `hooks/session-start.sh` | Claude Code session start | Reads local skill files and injects context |

**Safety assessment:** No remote code execution, no network calls, no credential access. Only reads local files from the repository.

### 5. Binary Files

| Check | Result |
|-------|--------|
| `.exe` files | None |
| `.dll` files | None |
| `.so` files | None |
| `.wasm` files | None |
| Other compiled artifacts | None |

All files are human-readable source code.

### 6. File Type Distribution

| Extension | Purpose | Count |
|-----------|---------|-------|
| `.md` | Skill documentation | 14+ |
| `.sh` | Shell scripts (test runners, utilities) | Multiple |
| `.js` | Utility scripts | 5 |
| `.json` | Plugin configuration | 3 |
| `.ts` | TypeScript example | 1 |
| `.dot` | Graphviz diagram source | 1 |
| `.cmd` | Windows batch wrapper | 1 |

No suspicious naming patterns or double extensions detected.

---

## Files Examined

### JavaScript Files (Full Audit)

1. `/lib/skills-core.js` - Core skill loading library
2. `/.opencode/plugin/superpowers.js` - OpenCode plugin
3. `/.codex/superpowers-codex` - Codex CLI tool
4. `/skills/writing-skills/render-graphs.js` - Diagram rendering utility

### Configuration Files

1. `/.claude-plugin/plugin.json` - Claude plugin configuration
2. `/.claude-plugin/marketplace.json` - Marketplace metadata
3. `/hooks/hooks.json` - Hook registration

### Shell Scripts

1. `/hooks/session-start.sh` - Session start hook
2. `/hooks/run-hook.cmd` - Cross-platform wrapper
3. `/tests/**/*.sh` - Test runners

---

## Conclusion

**Risk Level: LOW**

This repository is safe to use and install. It is a legitimate open-source project with:

- No external dependencies to compromise
- No suspicious automation or hidden execution
- Only local file operations
- Clean git history with meaningful commits
- Active maintenance (recent security fix in #297)
- Transparent development by established maintainer

### Recommendations

1. Review skills before using them in workflows
2. Keep the plugin updated via `/plugin update superpowers`
3. Verify file integrity from the official GitHub repository
