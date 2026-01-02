# Plan: Synkao GitHub Project Automation

> Tích hợp GitHub Project workflow tự động vào skill `synkao-github-skill`

## Thông tin

| Key | Value |
|-----|-------|
| Ngày tạo | 2024-12-24 |
| Skill target | `.claude/skills/synkao-github-skill/` |
| Project URL | https://github.com/orgs/synkao/projects/2 |
| Project Number | 2 |
| Owner | synkao (org) |

## Mục tiêu

Tự động update GitHub Project status theo workflow:

```
/brainstorm #issue → Todo
       ↓
    /plan        → In Progress
       ↓
commit + approve → Done
```

## Phases

| Phase | Tên | Status | File |
|-------|-----|--------|------|
| 01 | Prerequisites | ✅ completed | [phase-01-prerequisites.md](phase-01-prerequisites.md) |
| 02 | Script Development | ✅ completed | [phase-02-script-development.md](phase-02-script-development.md) |
| 03 | Skill Update | ✅ completed | [phase-03-skill-update.md](phase-03-skill-update.md) |
| 04 | Testing | ✅ completed | [phase-04-testing.md](phase-04-testing.md) |

## Deliverables

```
synkao-github-skill/
├── SKILL.md                    # Updated workflow
├── scripts/
│   ├── project-status.js       # Main script
│   └── package.json
└── references/
    └── project-workflow.md     # Detailed commands
```

## Script Commands

```bash
node scripts/project-status.js add <issue>      # Add + set Todo
node scripts/project-status.js progress <issue> # Set In Progress
node scripts/project-status.js done <issue>     # Set Done
```

## Dependencies

- `gh` CLI với scopes: `read:project`, `project`
- Node.js

## Notes

- Sử dụng `gh` CLI thay vì GraphQL trực tiếp để đơn giản
- Script detect Status field tự động từ project
