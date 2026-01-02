# Phase 03: Automation + Commands

## Context
- [plan.md](./plan.md)
- Depends on: [Phase 01](./phase-01-core-crud.md), [Phase 02](./phase-02-query-reports.md)

## Overview

| Key | Value |
|-----|-------|
| Date | 2024-12-25 |
| Priority | P2 |
| Status | ✅ Done (Completed: 2025-12-25) |
| Est. Effort | 2-3h |

## Requirements

### Slash Commands
Tạo commands trong `.claude/commands/project/` để user interact qua Claude Code.

### Automation
- Auto-sync status khi commit
- Trigger events khi status change

## Slash Commands

### Directory Structure
```
.claude/commands/project/
├── list.md      # /project:list [filters]
├── add.md       # /project:add <url> hoặc từ context
├── sync.md      # /project:sync - Bulk add issues
├── status.md    # /project:status <issue> <status>
├── next.md      # /project:next - Suggest next issue
├── close.md     # /project:close <issue> - Close issue
├── info.md      # /project:info
└── report.md    # /project:report
```

## Implementation Steps

### 1. Create Commands

#### `/project:list`
```markdown
---
description: List GitHub Project items với filters
argument-hint: [--status <status>] [--assignee <user>]
---

Activate `github-project` skill.

Run: `node .claude/skills/github-project/scripts/project-cli.ts list $ARGUMENTS`

Display results in table format.
```

#### `/project:add`
```markdown
---
description: Add issue to GitHub Project (from URL, issue number, or brainstorm context)
argument-hint: [issue-number-or-url]
---

Activate `github-project` skill.

## Modes

### Mode 1: Add existing issue (có args)
Run: `node .claude/skills/github-project/scripts/project-cli.ts add $ARGUMENTS`

### Mode 2: Create new issue từ context (không args)
Khi user vừa chạy /brainstorm xong:
1. Extract từ conversation context:
   - Title: từ problem statement
   - Body: requirements, solution, checklist
   - Labels: từ tech stack mentioned
2. Confirm với user trước khi tạo
3. Tạo issue: `gh issue create --title "..." --body "..."`
4. Add vào project: `node project-cli.ts add <new-issue-url>`
```

#### `/project:sync`
```markdown
---
description: Bulk add issues to GitHub Project
argument-hint: [--all] [--label <label>] [--milestone <milestone>]
---

Activate `github-project` skill.

## Modes

### Mode 1: Add all open issues
/project:sync --all
→ Add tất cả open issues chưa có trong project

### Mode 2: Add by filter
/project:sync --label fe
/project:sync --label "fe,P0"
/project:sync --milestone "FE-1: Setup"
/project:sync --assignee @tuntran

### Mode 3: Combine filters
/project:sync --label fe --milestone "FE-1: Setup"

## Process
1. List issues từ repo theo filter
2. Check existing items trong project
3. Add missing issues
4. Report: "Added X issues, Y already in project"
```

#### `/project:status`
```markdown
---
description: Update item status in GitHub Project
argument-hint: <issue> <status>
---

Activate `github-project` skill.

Arguments:
- $1: Issue number
- $2: Status (backlog, todo, progress, done)

Run: `node .claude/skills/github-project/scripts/project-cli.ts set $1 --status $2`
```

#### `/project:next`
```markdown
---
description: Suggest next issue to work on (highest priority Todo)
---

Activate `github-project` skill.

1. Query items với status=Todo, sort by priority (P0 > P1 > P2)
2. Output: "Next: #42 - Setup dark mode (P0, fe)"
3. Option: --start để auto set In Progress
```

#### `/project:close`
```markdown
---
description: Close issue and set Done in project
argument-hint: <issue>
---

Activate `github-project` skill.

1. Set status = Done trong project
2. Close issue: `gh issue close <issue>`
3. Output: "✅ #42 closed and marked Done"
```

#### `/project:info`
```markdown
---
description: Show GitHub Project info
---

Activate `github-project` skill.

Run: `node .claude/skills/github-project/scripts/project-cli.ts info`
```

#### `/project:report`
```markdown
---
description: Show GitHub Project stats dashboard
argument-hint: [--by status|assignee]
---

Activate `github-project` skill.

Run: `node .claude/skills/github-project/scripts/project-cli.ts report $ARGUMENTS`
```

### 2. Update SKILL.md
- [x] Add commands reference
- [x] Document automation features
- [x] Add usage examples

### 3. Automation Features (Optional)
- [x] Hook vào git commit để auto-update status
- [x] Detect issue number từ branch name
- [x] Suggest next issue khi done

## Usage Examples

### Example 1: List items
```bash
/project:list --status todo

# Output:
| # | Title | Status | Assignee |
|---|-------|--------|----------|
| 5 | Setup Zustand | 📋 Todo | @tuntran |
| 6 | Add TanStack Query | 📋 Todo | - |
```

### Example 2: Brainstorm → Create Issue
```bash
# Step 1: Brainstorm
/brainstorm "Add dark mode support"

# Claude outputs: problem, solution, requirements...

# Step 2: Create issue từ context
/project:add

# Claude:
# → Đọc context từ brainstorm
# → Tạo draft issue:
#   Title: "Add dark mode support"
#   Body: [extracted from brainstorm]
#   Labels: fe, enhancement
# → Confirm với user
# → gh issue create...
# → Add vào project
# → Output: "✅ Created #63, added to project"
```

### Example 3: Add existing issue
```bash
/project:add 42
# hoặc
/project:add https://github.com/synkao/synkao/issues/42
```

## Success Criteria

- [x] Tất cả 8 commands hoạt động (list, add, sync, status, next, close, info, report)
- [x] Commands có help text rõ ràng
- [x] Error messages user-friendly
- [x] SKILL.md document đầy đủ

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Command syntax lỗi | Medium | Validation + help |
| Skill không activate | Medium | Clear description |

## Final Deliverables

Sau Phase 03, skill hoàn chỉnh với:

```
.claude/skills/github-project/
├── SKILL.md
├── .env.example
├── config.example.json
├── package.json
├── tsconfig.json
├── scripts/
│   ├── project-cli.ts
│   ├── lib/
│   │   ├── gh-executor.ts
│   │   ├── config-loader.ts
│   │   ├── project-api.ts
│   │   ├── query-builder.ts
│   │   └── reporter.ts
│   └── __tests__/
└── references/
    ├── commands-usage.md
    └── api-reference.md

.claude/commands/project/
├── list.md
├── add.md
├── sync.md
├── status.md
├── next.md
├── close.md
├── info.md
└── report.md
```

## Post-Implementation

Sau khi skill hoàn thành:
1. Setup config cho project synkao (#2)
2. Test tất cả commands
3. Document trong SKILL.md
