# Synkao Plans

Implementation plans và reports cho [synkao](https://github.com/synkao/synkao) project.

## Structure

```
plans/
├── {date}-{issue}-{slug}/   # Plan directories
│   ├── plan.md              # Main plan
│   ├── phase-01-*.md        # Phase details
│   └── reports/             # Phase-specific reports
│
├── reports/                 # Centralized reports
│   ├── brainstorm-*.md
│   ├── code-reviewer-*.md
│   ├── tester-*.md
│   └── docs-manager-*.md
│
└── templates/               # Plan templates
```

## Naming Convention

| Type | Format |
|------|--------|
| Plan dir | `{YYMMDD}-{HHMM}-{issue}-{slug}/` |
| Report | `{agent}-{YYMMDD}-{HHMM}-{slug}.md` |

**Examples:**
- `251224-0844-issue1-nextjs-setup/`
- `code-reviewer-251225-1553-zustand-tanstack-phase01.md`

## Plans

| Date | Plan | Description |
|------|------|-------------|
| 24/12 | issue1-nextjs-setup | Next.js 14 project setup |
| 24/12 | synkao-github-project-automation | GitHub Project automation |
| 24/12 | shadcn-glass-theme | Glass theme với shadcn/ui |
| 24/12 | msw-setup | Mock Service Worker setup |
| 24/12 | mock-data-structure | Mock data generation |
| 25/12 | issue-5-zustand-tanstack-query | State management setup |
| 25/12 | github-project-skill | GitHub Project skill |
| 27/12 | mock-data-update | Mock data improvements |

## Usage

```bash
# Clone main repo with submodules
git clone --recurse-submodules https://github.com/synkao/synkao.git

# Or init after clone
git submodule update --init --recursive
```

## Related

- Main repo: [synkao/synkao](https://github.com/synkao/synkao)
- Docs: [synkao/synkao-docs](https://github.com/synkao/synkao-docs)
