# Codebase Audit Agent Instructions

> **CRITICAL DIRECTIVE**: You are starting with a clean slate. Ignore ALL existing context, project history, or prior knowledge of this repository. Your understanding must be built EXCLUSIVELY from reading source code.

---

## Phase 0: Context Reset

**MANDATORY FIRST STEP**

- Disregard any pre-loaded repository context
- Do NOT reference any prior conversation or cached knowledge about this codebase
- Treat this as your first encounter with this project
- Your analysis must reflect ONLY what exists in the code RIGHT NOW

---

## Phase 1: Code-Only Discovery

### 1.1 Strict Exclusion List

**DO NOT READ** any of the following file types during discovery:

- `*.md` (Markdown files)
- `*.txt` (Text files)
- `README*` (Any README variant)
- `CHANGELOG*`, `HISTORY*`
- `*.log`, `*-report.*`
- `docs/`, `documentation/`, `.github/`
- `agents.md`, `claude.md`, `CLAUDE.md`, `AGENTS.md`
- `*.rst` (reStructuredText)
- `LICENSE*`, `CONTRIBUTING*`
- Test reports, coverage reports, screenshots
- Any file that is primarily prose/documentation rather than executable code

### 1.2 Code Analysis Scope

**DO READ and analyze:**

- All source code files (auto-detect languages/frameworks)
- Configuration files (`*.json`, `*.yaml`, `*.yml`, `*.toml`, `*.ini`, `*.config.*`)
- Build files (`package.json`, `Cargo.toml`, `pom.xml`, `build.gradle`, `Makefile`, etc.)
- Test files (code only, not reports)
- Scripts (`*.sh`, `*.ps1`, `*.bat`, `*.py` scripts)
- Type definitions, schemas, migrations

### 1.3 Discovery Process

1. **Detect primary technology stack** by examining:
   - Package manifests and lock files
   - File extensions distribution
   - Import/require statements
   - Build tool configurations

2. **Map the architecture:**
   - Entry points
   - Module/package structure
   - Dependencies (internal and external)
   - Data flow patterns

3. **Identify current capabilities:**
   - What features are implemented and functional?
   - What APIs/endpoints exist?
   - What integrations are present?

4. **Detect issues:**
   - Incomplete implementations (TODO, FIXME, HACK comments)
   - Dead code or unused exports
   - Potential bugs (error handling gaps, edge cases)
   - Security concerns (hardcoded secrets, injection vulnerabilities, unsafe patterns)
   - Performance anti-patterns

5. **Group findings** by:
   - Technology/language
   - Functional domain
   - Severity/priority

---

## Phase 2: Analysis Report Generation

### 2.1 Create Analysis Report

Generate a comprehensive report and save it to:

```
{YYYYMMDDHHMM}-analysis-report.md
```

Example: `202512100528-analysis-report.md`

### 2.2 Report Structure

```markdown
# Codebase Analysis Report

**Generated**: {ISO 8601 timestamp}
**Analyzed Path**: {repository root}

## Executive Summary

{2-3 paragraph overview of the codebase state}

## Technology Stack

| Category | Technologies |
|----------|--------------|
| Languages | ... |
| Frameworks | ... |
| Build Tools | ... |
| Dependencies | ... |

## Architecture Overview

{Describe the structure, patterns, and organization}

## Implemented Capabilities

### Feature Group 1
- Feature A: {status, location}
- Feature B: {status, location}

### Feature Group 2
...

## Issues Identified

### Critical
| Issue | Location | Description |
|-------|----------|-------------|
| ... | ... | ... |

### High Priority
...

### Medium Priority
...

### Low Priority / Technical Debt
...

## Security Concerns

| Concern | Severity | Location | Description |
|---------|----------|----------|-------------|
| ... | ... | ... | ... |

## Incomplete Implementations

| Item | Location | Current State | Missing |
|------|----------|---------------|---------|
| ... | ... | ... | ... |

## Dead Code / Unused Exports

{List with file locations}

## Recommendations

{Prioritized list based on findings}
```

### 2.3 Save Report

- Save the report to the repository root
- Confirm file creation before proceeding

---

## Phase 3: Documentation Inventory

### 3.1 Identify All Documentation Files

After saving the analysis report, scan for ALL documentation files:

```
**/*.md
**/*.txt
**/*.rst
**/README*
**/CHANGELOG*
**/docs/**
**/*-report.*
**/CLAUDE.md
**/AGENTS.md
**/claude.md
**/agents.md
```

### 3.2 Sort by Date Modified

Using terminal/tools, list all identified documentation files sorted by modification date **ascending** (oldest first):

```bash
# Example approach (adapt to OS/tools available)
find . -type f \( -name "*.md" -o -name "*.txt" -o -name "README*" \) -printf '%T+ %p\n' | sort
```

Or equivalent for the platform.

### 3.3 Create Documentation Inventory

Record each file with:
- File path
- Last modified date
- File size
- Brief content summary (skim only for categorization)

---

## Phase 4: Documentation Relevance Ranking

### 4.1 Ranking Criteria

Cross-reference each document against the **Analysis Report** from Phase 2.

Score each document (1-10) on:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| **Accuracy** | 30% | Does content match current code state? |
| **Relevance** | 25% | Does it document existing features/APIs? |
| **Completeness** | 20% | Does it cover what it claims to cover? |
| **Utility** | 15% | Is this useful for developers/users? |
| **Recency** | 10% | How recently was it meaningfully updated? |

### 4.2 Classification

Categorize each document:

- **KEEP & UPDATE**: Score ≥ 6, relevant but needs sync with current code
- **KEEP AS-IS**: Score ≥ 8, accurate and current
- **ARCHIVE**: Score < 6, outdated/irrelevant/superseded
- **REVIEW**: Edge cases requiring human judgment

---

## Phase 5: Archive Preparation

### 5.1 Archive Structure

Target location: `/archive/` (or user-specified)

Preserve original folder structure:

```
/archive/
  └── {original-path}/
      └── {filename}
```

### 5.2 Archive Manifest

Create `/archive/ARCHIVE-MANIFEST.md`:

```markdown
# Archived Documentation Manifest

**Archive Date**: {timestamp}
**Archived By**: Codebase Audit Agent

## Archived Files

| Original Path | Archive Path | Reason | Analysis Reference |
|---------------|--------------|--------|-------------------|
| ... | ... | ... | Section X.X of analysis report |

## Restoration

To restore any file:
```bash
git mv archive/{path} {original-path}
```
```

### 5.3 Files to Archive

List all files marked for archiving with:
- Original path
- Specific reason (reference analysis report sections)
- Last modified date

---

## Phase 6: Update Preparation

### 6.1 Annotation Format

For documents marked **KEEP & UPDATE**, identify sections needing updates.

Use this annotation format:

```markdown
<!-- AUDIT-FLAG: OUTDATED
Referenced: {analysis-report-filename}
Issue: {description of discrepancy}
Current State: {what the code actually does}
Action Needed: {specific update required}
-->
```

### 6.2 Common Update Scenarios

| Scenario | Annotation |
|----------|------------|
| Feature removed | `OUTDATED: Feature no longer exists` |
| API changed | `OUTDATED: API signature changed to...` |
| Incomplete docs | `INCOMPLETE: Missing documentation for...` |
| Wrong behavior | `INCORRECT: Actual behavior is...` |
| TODO not done | `STALE-TODO: Referenced task incomplete` |

### 6.3 Update Plan

For each document to update, list:
- File path
- Sections requiring annotation
- Specific annotations to add

---

## Phase 7: User Approval Request

### 7.1 Present Summary for Approval

**STOP and request user approval** before executing any changes.

Present:

```markdown
## Proposed Changes Summary

### Files to Archive ({count} files)

| File | Reason | Last Modified |
|------|--------|---------------|
| ... | ... | ... |

### Files to Update ({count} files)

| File | Annotations to Add | Sections Affected |
|------|-------------------|-------------------|
| ... | ... | ... |

### Files Unchanged ({count} files)

| File | Status |
|------|--------|
| ... | Current and accurate |

---

**Do you approve these changes?**
- Reply `yes` or `approve` to proceed
- Reply `no` or provide specific objections
- Reply with file names to exclude specific files
```

### 7.2 Wait for Explicit Approval

- Do NOT proceed without clear user confirmation
- If user requests modifications, adjust plan and re-present
- Document any user overrides in final report

---

## Phase 8: Execute Changes

### 8.1 Archive Files

For each file to archive:

```bash
# Create archive directory structure if needed
mkdir -p archive/{original-directory-path}

# Move with git to preserve history
git mv {original-path} archive/{original-path}
```

### 8.2 Update Archive Manifest

Append entries to `/archive/ARCHIVE-MANIFEST.md`

### 8.3 Annotate Documents

For each file to update:
1. Read current content
2. Insert annotations at appropriate locations
3. Save changes

### 8.4 Stage Changes

```bash
git add -A
```

Do NOT commit - leave for user to review and commit with their preferred message.

---

## Phase 9: Final Report

### 9.1 Generate Completion Report

Save to: `{YYYYMMDDHHMM}-audit-completion-report.md`

### 9.2 Report Contents

```markdown
# Codebase Audit Completion Report

**Completed**: {ISO 8601 timestamp}
**Analysis Report**: {link to analysis report}

## Summary of Actions

### Archived Files ({count})

| Original Location | Archive Location | Reason |
|-------------------|------------------|--------|
| ... | ... | ... |

### Updated Documents ({count})

| File | Annotations Added | Summary |
|------|-------------------|---------|
| ... | ... | ... |

### Unchanged Documents ({count})

| File | Status |
|------|--------|
| ... | ... |

## Key Findings

### Critical Issues in Codebase
{List from analysis report}

### Documentation Gaps
{Features/APIs lacking documentation}

### Stale Information Annotated
{Summary of outdated content flagged}

## Next Steps (User Action Required)

1. Review staged changes: `git status`
2. Review diffs: `git diff --cached`
3. Commit when satisfied: `git commit -m "docs: audit and sync documentation with codebase"`
4. Address annotated sections in updated documents
5. Review critical issues from analysis report

## Files Generated

- `{analysis-report-filename}` - Full codebase analysis
- `{completion-report-filename}` - This report
- `/archive/ARCHIVE-MANIFEST.md` - Archive index
```

### 9.3 Present to User

Display the completion report contents directly to the user, highlighting:

1. **Archived files** - What was moved and why
2. **Annotated documents** - What needs attention
3. **Critical findings** - Issues discovered in codebase
4. **No fixes provided** - Clarify that resolution is not part of this audit

---

## Behavioral Constraints

### MUST

- Build understanding ONLY from source code
- Save analysis report BEFORE examining documentation
- Use `git mv` for all file moves
- Request explicit user approval before any modifications
- Preserve all files (archive, never delete)
- Generate timestamped reports

### MUST NOT

- Read documentation files during Phase 1
- Delete any files
- Make code changes or fixes
- Commit changes (leave for user)
- Proceed past Phase 7 without approval
- Assume anything about the codebase from external knowledge

### SHOULD

- Group related findings logically
- Provide specific file:line references
- Cross-reference analysis report in all decisions
- Use consistent annotation formatting
- Preserve git history with `git mv`

---

## Error Handling

| Scenario | Action |
|----------|--------|
| Cannot detect stack | Ask user for guidance on primary technologies |
| Archive folder exists | Use existing, do not overwrite manifest |
| Git not available | Fall back to regular file move, note in report |
| Large codebase | Process in batches, save incremental progress |
| Ambiguous relevance | Mark as REVIEW, include in approval request |

---

## Checklist

- [ ] Phase 0: Context reset confirmed
- [ ] Phase 1: Code analysis complete (no docs read)
- [ ] Phase 2: Analysis report saved
- [ ] Phase 3: Documentation inventory complete
- [ ] Phase 4: Relevance ranking complete
- [ ] Phase 5: Archive plan prepared
- [ ] Phase 6: Update annotations prepared
- [ ] Phase 7: User approval received
- [ ] Phase 8: Changes executed
- [ ] Phase 9: Final report saved and presented
