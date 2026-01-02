# Phase 01: Core CRUD + Config

## Context
- [plan.md](./plan.md)

## Overview

| Key | Value |
|-----|-------|
| Date | 2024-12-25 |
| Priority | P0 |
| Status | ✅ Done |
| Est. Effort | 4-6h |
| Completed | 2024-12-25 |

## Requirements

### Functional
1. Config loader: đọc ENV + JSON config
2. Add/remove items từ project
3. Update single-select fields (status)
4. Update text fields (title)
5. Get item info

### Non-functional
- Zero token setup (dùng gh CLI auth)
- Cross-platform (Windows/Linux/Mac)
- Error handling rõ ràng

## Architecture

```
scripts/
├── project-cli.ts           # CLI entry point
├── lib/
│   ├── gh-executor.ts       # gh CLI wrapper
│   ├── config-loader.ts     # ENV + JSON loader
│   └── project-api.ts       # CRUD operations
└── __tests__/
    └── project-api.test.ts
```

## Config Structure

### .env.example
```env
PROJECT_OWNER=synkao
PROJECT_NUMBER=2
REPO=synkao/synkao
```

### config.example.json
```json
{
  "fields": {
    "status": "Status",
    "priority": "Priority"
  },
  "statusMap": {
    "backlog": "📥 Backlog",
    "todo": "📋 Todo",
    "progress": "🔧 In Progress",
    "done": "✅ Done"
  }
}
```

## Implementation Steps

### 1. Setup project structure
- [ ] Create skill directory
- [ ] Init package.json with TypeScript
- [ ] Create .env.example, config.example.json
- [ ] Write SKILL.md skeleton

### 2. Config Loader (`lib/config-loader.ts`)
- [ ] Load .env với fallback chain
- [ ] Parse JSON config
- [ ] Validate required fields
- [ ] Export merged config object

### 3. GH Executor (`lib/gh-executor.ts`)
- [ ] Wrapper cho `gh` CLI
- [ ] JSON parsing helper
- [ ] Error handling
- [ ] **Auth check**: `gh auth status` trước khi execute, hướng dẫn login nếu chưa

### 4. Project API (`lib/project-api.ts`)
- [ ] `getProjectInfo()` - lấy project metadata
- [ ] `listItems(limit)` - list items
- [ ] `findItem(issueNumber)` - tìm item by issue
- [ ] `addItem(url)` - add issue/PR
- [ ] `removeItem(itemId)` - remove item
- [ ] `updateField(itemId, fieldName, value)` - update field

### 5. CLI Entry Point (`project-cli.ts`)
- [ ] Parse args (commander/yargs)
- [ ] Commands: info, list, add, remove, set
- [ ] Help text

### 6. Tests
- [ ] Unit tests cho config-loader
- [ ] Unit tests cho project-api (mock gh)
- [ ] Integration test với real project

## CLI Commands

```bash
# Project info
node project-cli.ts info

# List items
node project-cli.ts list [--limit 20]

# Add item
node project-cli.ts add <url>
node project-cli.ts add https://github.com/synkao/synkao/issues/42

# Remove item
node project-cli.ts remove <item-id>

# Update field
node project-cli.ts set <issue> --status progress
node project-cli.ts set <issue> --status done --assignee @tuntran
```

## Success Criteria

- [ ] `project-cli.ts info` hiển thị project metadata
- [ ] `project-cli.ts list` list được items
- [ ] `project-cli.ts add <url>` add được issue
- [ ] `project-cli.ts set <issue> --status <x>` update được status
- [ ] Config hoạt động với bất kỳ project nào (không hardcode)
- [ ] Tests pass 100%

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| gh CLI không cài | High | Check và hướng dẫn install |
| Rate limit | Medium | Cache responses |
| Field ID thay đổi | Medium | Auto-detect từ field name |

## Security

- Không log sensitive data
- Không commit .env files
- Validate user input

## Next Steps

Sau khi hoàn thành Phase 01:
- → [Phase 02: Query & Reports](./phase-02-query-reports.md)
