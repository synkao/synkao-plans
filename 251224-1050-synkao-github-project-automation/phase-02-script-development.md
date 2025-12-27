# Phase 02: Script Development

## Context

- [plan.md](plan.md)
- Prerequisite: [phase-01-prerequisites.md](phase-01-prerequisites.md)

## Overview

| Key | Value |
|-----|-------|
| Ngày | 2024-12-24 |
| Priority | High |
| Status | pending |
| Estimate | 30 mins |

## Key Insights

- Sử dụng `gh` CLI commands thay vì GraphQL trực tiếp → đơn giản hơn
- Script cần handle: add item, update status
- Detect Status field dynamically từ project

## Requirements

- Node.js runtime
- `gh` CLI với proper scopes
- Project number: 2, Owner: synkao

## Architecture

```
scripts/
├── project-status.js   # Main executable
└── package.json        # Dependencies (minimal)
```

### Script Flow

```
1. Parse args (action, issue-number)
2. Get project item ID từ issue
3. Get Status field ID từ project
4. Execute action (add/progress/done)
5. Output result
```

## Related Code Files

- `.claude/skills/synkao-github-skill/scripts/project-status.js` (new)
- `.claude/skills/synkao-github-skill/scripts/package.json` (new)

## Implementation Steps

### Step 1: Create scripts directory

```bash
mkdir -p .claude/skills/synkao-github-skill/scripts
```

### Step 2: Create package.json

Minimal, no external deps (use child_process for gh).

### Step 3: Create project-status.js

Commands:
- `add <issue>` - Add issue to project, set Todo
- `progress <issue>` - Set In Progress
- `done <issue>` - Set Done
- `status <issue>` - Get current status

### Step 4: Test script

```bash
node .claude/skills/synkao-github-skill/scripts/project-status.js add 1
node .claude/skills/synkao-github-skill/scripts/project-status.js status 1
```

## Todo

- [ ] Create scripts directory
- [ ] Create package.json
- [ ] Implement project-status.js
- [ ] Test với real issue

## Success Criteria

- Script chạy không lỗi
- `add` command thêm issue vào project
- `progress` và `done` update status đúng
- Error handling rõ ràng

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Status field name khác | Medium | Auto-detect từ field-list |
| Issue chưa trong project | Medium | `add` command handle case này |

## Security Considerations

- Không hardcode tokens
- Validate input issue numbers
- Escape shell arguments

## Next Steps

→ [phase-03-skill-update.md](phase-03-skill-update.md)
