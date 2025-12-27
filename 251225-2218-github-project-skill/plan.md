# GitHub Project Skill - Implementation Plan

> Generic skill để tương tác với bất kỳ GitHub Project V2 nào

## Overview

| Key | Value |
|-----|-------|
| Skill Name | `github-project` |
| Location | `.claude/skills/github-project/` |
| API Method | gh CLI |
| Config | ENV + JSON |
| Language | TypeScript/Node.js |

## Phases

| Phase | Focus | Status | Link |
|-------|-------|--------|------|
| 01 | Core CRUD + Config | ✅ Done (2024-12-25) | [phase-01](./phase-01-core-crud.md) |
| 02 | Query & Reports | ✅ Done (2025-12-25) | [phase-02](./phase-02-query-reports.md) |
| 03 | Automation + Commands | ✅ Done (2025-12-25) | [phase-03](./phase-03-automation-commands.md) |

## Deliverables

### Skill Structure
```
.claude/skills/github-project/
├── SKILL.md
├── .env.example
├── config.example.json
├── scripts/
│   ├── project-cli.ts
│   └── lib/
└── references/
```

### Commands (8 total)
```
.claude/commands/project/
├── list.md      # /project:list - filter items
├── add.md       # /project:add - add single/from context
├── sync.md      # /project:sync - bulk add issues
├── status.md    # /project:status - update status
├── next.md      # /project:next - suggest next issue
├── close.md     # /project:close - close issue + done
├── info.md      # /project:info - project info
└── report.md    # /project:report - stats dashboard
```

## Key Features

**Phase 1 - Core:**
- Add/remove items from project
- Update fields (status, assignee, labels)
- Config-driven field mapping

**Phase 2 - Query:**
- Filter by status/assignee/label/milestone
- Stats dashboard (progress, workload)
- Export to markdown/json

**Phase 3 - Automation:**
- Slash commands integration
- Auto-sync với issues
- Event triggers

## Tech Stack

- **Runtime:** Node.js + TypeScript
- **API:** gh CLI (no token setup needed)
- **Config:** dotenv + JSON
- **Test:** Jest

## Notes

- Skill generic, có thể reuse cho bất kỳ org/project nào
- Config-driven, không hardcode IDs
- Commands cho phép user interact trực tiếp qua slash commands
