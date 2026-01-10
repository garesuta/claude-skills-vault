---
name: review-pr
description: Review PR for skills/commands/MCP - validates docs, quality, formatting
tools:
  - Bash(git diff *)
  - Bash(gh pr *)
  - Read
  - Glob
  - Grep
  - Task
  - Skill(gemini-cli)
---

# PR Review Command

**IMPORTANT: Only run when invoked via `/git-review-pr`. Do NOT auto-run.**

Goal: Review PRs for skills, commands, MCP servers.

## 0. Prerequisites

Check required tools:
```bash
command -v gh >/dev/null || echo "Warning: gh CLI not found - PR features unavailable"
command -v gemini >/dev/null || echo "Warning: gemini CLI not found - skip Gemini review"
```

If tools missing, continue w/ available features.

## 1. Identify Changes

```bash
git diff --name-only origin/main...HEAD  # full PR delta (multi-commit)
git diff --name-only HEAD~1..HEAD        # single commit only
gh pr diff --name-only                   # PR via GitHub CLI
```

| Path | Type |
|------|------|
| `.claude/skills/*/` | Skill |
| `.claude/commands/*.md` | Command |
| `mcp-servers/*/` | MCP Server |

## 2. Documentation Checks

### CHANGELOG.md

```bash
git diff origin/main...HEAD -- CHANGELOG.md
```

**Required:** Entry under `[Unreleased]` w/ correct category & format `- **scope**: Desc`

**If missing:** FAIL

### README.md

```bash
git diff origin/main...HEAD -- README.md
```

- New skills → Core/Document Skills table
- New commands → Commands table
- New MCP → appropriate section

**If missing:** FAIL

## 3. Content Quality

### Skills (`.claude/skills/*/SKILL.md`)

**Structure:**
- [ ] YAML frontmatter (name, desc)
- [ ] `# Title` heading
- [ ] `## When to Use`
- [ ] Invocation instructions
- [ ] Examples

**Issues:**
- [ ] No dead code/unused imports
- [ ] No hardcoded paths
- [ ] No duplicate content
- [ ] Not overly verbose
- [ ] Scripts functional

### Commands (`.claude/commands/*.md`)

**Structure:**
- [ ] YAML frontmatter (name, desc, tools)
- [ ] Goal statement
- [ ] Step-by-step flow
- [ ] Safety protocols

**Issues:**
- [ ] No duplicate instructions
- [ ] Safe commands (no destructive defaults)
- [ ] Clear prompts
- [ ] Proper escaping

### MCP Servers (`mcp-servers/*/README.md`)

**Structure:**
- [ ] Title & desc
- [ ] Install instructions
- [ ] Config example
- [ ] Source link

**Issues:**
- [ ] Valid JSON config
- [ ] Working URLs
- [ ] No placeholders

## 4. Tests & Validation

Run before deep review - fail early if broken:

```bash
# Validate Python scripts exist and have no syntax errors
find .claude/skills -name "*.py" -exec python -m py_compile {} \;

# Check markdown files render (no broken syntax)
# If markdownlint installed:
npx markdownlint-cli <changed_md_files> 2>/dev/null || echo "markdownlint not found - manual check"
```

If tests fail → stop review, fix first.

## 5. Code Quality

| Issue | Severity |
|-------|----------|
| Repetitive text (3+) | Medium |
| Verbose prose | Low |
| Obvious dead code | Medium |
| Hardcoded secrets | Critical |
| Broken links | High |

```bash
# -nH: line numbers + filename
grep -nH "TODO\|FIXME\|XXX" <files>
grep -nH "console.log\|print(\|debugger" <files>
grep -nH "api_key\|secret\|password" <files>
```

## 6. Token Optimization

**Ref:** `.claude/skills/token-formatter/SKILL.md`

Check:
- Filler words
- Verbose → tables
- Redundant explanations

**Report:** File, current tokens, potential savings, suggestions

**Action (if >20% savings possible):**
```bash
python .claude/skills/token-formatter/scripts/compress.py <file> --level 2
```

Ask: "Apply token compression to [file]? (Y/N)"

## 7. Markdown Validation

**Ref:** `.claude/skills/document-skills/md/SKILL.md`

- [ ] Code blocks closed
- [ ] Fences match
- [ ] Nested = longer fences
- [ ] Table columns equal
- [ ] Pipes escaped
- [ ] Links valid
- [ ] Space after #
- [ ] Blank lines around blocks

## 8. Review Report

```markdown
# PR Review Report

## Summary
Files: X | Skills: Y | Commands: Z | MCP: W
Issues: A (B Critical, C High, D Medium)

## Docs Status
| File | Status |
|------|--------|
| CHANGELOG.md | Pass/Fail |
| README.md | Pass/Fail |

## Content
### [Name]
- Structure: Pass/Fail
- Quality: Pass/Fail
- Issues: [Severity] Desc (line X)

## Token Optimization
| File | Current | Savings |
|------|---------|---------|
| x.md | ~500 | 30% |

## Markdown
| File | Errors | Warnings |
|------|--------|----------|
| x.md | 0 | 2 |

## Verdict
[ ] APPROVE | [ ] REQUEST CHANGES | [ ] NEEDS DISCUSSION
```

## 9. Gemini Second Opinion

```bash
gemini -y "Review this Claude Code skills PR:
Files: <list>
My findings: <summary>
Provide: 1) Missed issues 2) Suggestions 3) Assessment"
```

Present both reviews.

## 10. Flow

1. Prerequisites check
2. `git diff --name-only` → files
3. Categorize (skill/cmd/mcp)
4. Check CHANGELOG
5. Check README
6. Review structure & quality
7. Tests & validation
8. Code quality checks
9. Token optimization (+ action)
10. Markdown validation
11. Generate report
12. Gemini review
13. Final verdict

## 11. Quick Ref

**Pass:** CHANGELOG + README updated, structure present, no critical issues, valid markdown

**Auto-Fail:** Missing CHANGELOG/README entry, secrets, broken markdown, missing frontmatter/When to Use

---
Integrates: token-formatter, md, gemini-cli | ~450 tokens