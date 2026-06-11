---
name: sopf-orchestrator
description: "Orchestrates the full SOPF (Save on Prop Firms) site maintenance pipeline. Use when updating prop firm pages, adding new firm pages, changing promo codes or deals, refreshing account data, updating the index or trading-tools pages, auditing the site for stale data, or any task involving the otakgemuk/sopf GitHub repo. Also triggers on: 'update sopf', 'add firm to sopf', 'refresh promo', 'new firm page', 'sopf audit', 'fix firm page', 'harness sopf', 'run sopf pipeline'."
---

# SOPF Orchestrator

Pipeline orchestrator for the Save on Prop Firms static site. Coordinates FirmResearch → CopyWriter → PageBuilder → BrandReviewer → GitCommit.

## Repo
otakgemuk/sopf (main branch)

## Agent Team
| Agent | File | Role |
|-------|------|------|
| FirmResearch | .claude/agents/firm-research.md | Data validation + affiliate codes |
| CopyWriter | .claude/agents/copy-writer.md | Page copy + CTAs |
| PageBuilder | .claude/agents/page-builder.md | HTML construction |
| BrandReviewer | .claude/agents/brand-reviewer.md | QA before commit |
| GitCommit | .claude/agents/git-commit.md | Push to GitHub |

## Architecture Pattern
Pipeline with Producer-Reviewer gate before commit.

```
FirmResearch → CopyWriter → PageBuilder → BrandReviewer → GitCommit
                                               ↓ (FAIL)
                                           PageBuilder (fix)
```

## Phase 0: Context Check
Before starting, determine run mode:
- `_workspace/` exists + partial task → **resume** from last checkpoint
- `_workspace/` exists + new task → rename to `_workspace_prev/`, start fresh
- No `_workspace/` → **initial run**

Read existing firm HTML file if updating (fetch from otakgemuk/sopf). Identify what changed vs what stays the same.

## Phase 1: Research
Spawn FirmResearch agent.
- Task: validate firm data for the requested firm(s)
- Input: firm name(s) + task type
- Output: structured data object → save to `_workspace/01_research_{firm}.md`
- If [NEEDS VERIFICATION] markers present: report to user before proceeding

## Phase 2: Copy
Spawn CopyWriter agent.
- Input: `_workspace/01_research_{firm}.md`
- Task: generate all copy blocks for the page
- Output: copy blocks → save to `_workspace/02_copy_{firm}.md`

## Phase 3: Build
Spawn PageBuilder agent.
- Input: `_workspace/01_research_{firm}.md` + `_workspace/02_copy_{firm}.md` + existing HTML (if update)
- Task: build or update the HTML file
- Output: complete HTML → save to `_workspace/03_html_{firm}.html`

## Phase 4: Review
Spawn BrandReviewer agent.
- Input: `_workspace/03_html_{firm}.html`
- Task: run full review checklist
- Output: PASS or FAIL report

On FAIL:
- Send issues back to PageBuilder
- PageBuilder fixes and re-saves to `_workspace/03_html_{firm}.html`
- Re-run BrandReviewer
- Max 2 correction cycles before escalating to user

## Phase 5: Commit
Spawn GitCommit agent.
- Input: PASS signal + `_workspace/03_html_{firm}.html` + target path
- Fetch current SHA if updating existing file
- Commit to otakgemuk/sopf/main
- Output: commit URL + SHA

## Phase 6: Confirm + Log
- Report commit URL to user
- Update CLAUDE.md change history
- Ask user for feedback on output quality

## Data Flow (file-based)
```
_workspace/
  01_research_{firm}.md     ← FirmResearch output
  02_copy_{firm}.md         ← CopyWriter output
  03_html_{firm}.html       ← PageBuilder output (reviewed + approved)
```

## Error Handling
- Research fails: mark [NEEDS VERIFICATION], ask user to confirm before proceeding
- Copy incomplete: PageBuilder uses TODO placeholders, BrandReviewer will catch
- Build fails: escalate to user with specific error
- Review fails twice: escalate to user with both FAIL reports
- Commit 409 (SHA mismatch): GitCommit re-fetches SHA and retries once
- Commit fails twice: escalate to user with error details

## Supported Task Types
| Task | Phases Run |
|------|------------|
| Update firm page (data/promo change) | All phases |
| New firm page | All phases |
| Promo-only update | Phase 1 (promo data only) → Phase 3 (surgical edit) → Phase 4 → Phase 5 |
| Site audit | Phase 1 (all firms) → report only, no commit |
| Copy refresh only | Phase 2 → Phase 3 → Phase 4 → Phase 5 |

## Test Scenarios

**Normal flow:**
> "Update the MFFU page with the new Builder Plan promo — 40% off code MOT"
1. FirmResearch validates MFFU data + confirms MOT code
2. CopyWriter generates updated promo block copy
3. PageBuilder updates .promo-box section surgically
4. BrandReviewer checks affiliate link + code case → PASS
5. GitCommit fetches SHA, commits to prop-firms/myfundedfutures.html

**Error flow:**
> "Add a new firm page for TopstepX"
1. FirmResearch pulls TopstepX data — some fields unverifiable → [NEEDS VERIFICATION] on CEO field
2. Reports to user: "CEO field could not be verified — proceed with N/A or hold?"
3. User confirms proceed → CopyWriter omits CEO, PageBuilder inserts <!-- TODO: CEO -->
4. BrandReviewer flags TODO → FAIL
5. PageBuilder removes TODO placeholder, replaces with N/A
6. BrandReviewer re-runs → PASS
7. GitCommit commits new file (no SHA needed for new file)
