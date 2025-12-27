# Phase 02: Query & Reports

## Context
- [plan.md](./plan.md)
- Depends on: [Phase 01](./phase-01-core-crud.md)

## Overview

| Key | Value |
|-----|-------|
| Date | 2025-12-25 |
| Priority | P1 |
| Status | ✅ Done (Completed: 2025-12-25) |
| Est. Effort | 3-4h |

## Requirements

### Functional
1. Filter items by multiple criteria
2. Search items by text
3. Stats dashboard
4. Export to markdown/JSON

### Filters Supported
- `--status` (backlog, todo, progress, done)
- `--assignee` (@user hoặc "unassigned")
- `--label` (fe, be, P0, etc.)
- `--milestone` (milestone title)
- `--search` (text search in title/body)

## Architecture

```
scripts/lib/
├── query-builder.ts    # Filter & search logic
└── reporter.ts         # Stats & export
```

## Implementation Steps

### 1. Query Builder (`lib/query-builder.ts`)
- [ ] Parse filter options
- [ ] Filter in-memory (gh CLI không hỗ trợ server-side filter)
- [ ] Text search với fuzzy match
- [ ] Sort results (by status, date, etc.)

### 2. Reporter (`lib/reporter.ts`)
- [ ] Count by status
- [ ] Count by assignee
- [ ] Progress percentage
- [ ] Markdown table output
- [ ] JSON export

### 3. Extend CLI
- [ ] `list` command với filter options
- [ ] `report` command cho dashboard
- [ ] `export` command cho JSON/MD

## CLI Commands

```bash
# Filter by status
node project-cli.ts list --status todo
node project-cli.ts list --status "todo,progress"

# Filter by assignee
node project-cli.ts list --assignee @tuntran
node project-cli.ts list --assignee unassigned

# Filter by label
node project-cli.ts list --label fe
node project-cli.ts list --label "fe,P0"

# Combined filters
node project-cli.ts list --status todo --assignee @tuntran --label fe

# Search
node project-cli.ts list --search "auth"

# Reports
node project-cli.ts report              # Summary dashboard
node project-cli.ts report --by status  # Group by status
node project-cli.ts report --by assignee

# Export
node project-cli.ts export --format json > items.json
node project-cli.ts export --format md > items.md
```

## Report Output Example

```markdown
## Project: Plans (#2)

### Summary
| Status | Count | % |
|--------|-------|---|
| ✅ Done | 12 | 19% |
| 🔧 In Progress | 5 | 8% |
| 📋 Todo | 20 | 32% |
| 📥 Backlog | 25 | 40% |
| **Total** | **62** | **100%** |

### By Assignee
| Assignee | Todo | Progress | Done |
|----------|------|----------|------|
| @tuntran | 8 | 2 | 5 |
| Unassigned | 12 | 3 | 7 |
```

## Success Criteria

- [ ] Filter theo từng criteria hoạt động
- [ ] Combined filters hoạt động
- [ ] Search tìm được items
- [ ] Report hiển thị đúng stats
- [ ] Export ra JSON/MD đúng format
- [ ] Tests pass

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Large project (>100 items) | Medium | Pagination, cache |
| Filter performance | Low | In-memory filter nhanh |

## Next Steps

Sau khi hoàn thành Phase 02:
- → [Phase 03: Automation + Commands](./phase-03-automation-commands.md)
