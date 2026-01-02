# Phase 03: Skill Update

## Context

- [plan.md](plan.md)
- Script: [phase-02-script-development.md](phase-02-script-development.md)

## Overview

| Key | Value |
|-----|-------|
| Ngày | 2024-12-24 |
| Priority | Medium |
| Status | pending |
| Estimate | 15 mins |

## Key Insights

- SKILL.md phải < 100 lines
- Workflow mới cần integrate với `/brainstorm`, `/plan`, commit
- Thêm references file cho chi tiết commands

## Requirements

- Script `project-status.js` đã hoạt động
- SKILL.md hiện tại: 109 lines (cần optimize)

## Architecture

```
synkao-github-skill/
├── SKILL.md                    # < 100 lines, core workflow
├── ISSUES.md                   # Giữ nguyên
├── scripts/
│   └── project-status.js
└── references/
    └── project-workflow.md     # Detailed project commands
```

## Related Code Files

- `.claude/skills/synkao-github-skill/SKILL.md` (update)
- `.claude/skills/synkao-github-skill/references/project-workflow.md` (new)

## Implementation Steps

### Step 1: Create references directory

```bash
mkdir -p .claude/skills/synkao-github-skill/references
```

### Step 2: Create project-workflow.md

Chi tiết về:
- Project commands
- Status values
- Troubleshooting

### Step 3: Update SKILL.md

Thêm workflow section:
```markdown
## Project Workflow

Khi làm việc với issue:
1. `/brainstorm` → `node scripts/project-status.js add <issue>`
2. `/plan` → `node scripts/project-status.js progress <issue>`
3. Commit xong → `node scripts/project-status.js done <issue>`

Chi tiết: `references/project-workflow.md`
```

### Step 4: Optimize SKILL.md

- Di chuyển commands chi tiết sang references
- Giữ < 100 lines

## Todo

- [ ] Create references directory
- [ ] Create project-workflow.md
- [ ] Update SKILL.md với workflow mới
- [ ] Ensure < 100 lines

## Success Criteria

- SKILL.md < 100 lines
- Workflow rõ ràng và dễ follow
- References có đủ chi tiết cho edge cases

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| SKILL.md quá dài | Low | Split thêm vào references |

## Security Considerations

- Không expose sensitive info trong references

## Next Steps

→ [phase-04-testing.md](phase-04-testing.md)
