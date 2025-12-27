# Phase 04: Testing

## Context

- [plan.md](plan.md)
- Skill: [phase-03-skill-update.md](phase-03-skill-update.md)

## Overview

| Key | Value |
|-----|-------|
| Ngày | 2024-12-24 |
| Priority | Medium |
| Status | pending |
| Estimate | 15 mins |

## Key Insights

- Test với real issue để verify end-to-end
- Check GitHub Project UI sau mỗi command
- Verify error handling

## Requirements

- Script đã implement
- SKILL.md đã update
- Có ít nhất 1 open issue để test

## Implementation Steps

### Step 1: Test add command

```bash
# Pick 1 open issue
gh issue list --repo synkao/synkao --state open --limit 1

# Add to project
node .claude/skills/synkao-github-skill/scripts/project-status.js add <issue>

# Verify trên https://github.com/orgs/synkao/projects/2
```

### Step 2: Test progress command

```bash
node .claude/skills/synkao-github-skill/scripts/project-status.js progress <issue>

# Verify status = "In Progress" trên project
```

### Step 3: Test done command

```bash
node .claude/skills/synkao-github-skill/scripts/project-status.js done <issue>

# Verify status = "Done" trên project
```

### Step 4: Test error cases

```bash
# Invalid issue number
node .claude/skills/synkao-github-skill/scripts/project-status.js add 99999

# Missing argument
node .claude/skills/synkao-github-skill/scripts/project-status.js add
```

### Step 5: Workflow integration test

1. Chạy `/brainstorm` cho 1 issue
2. Verify script được suggest và chạy
3. Check project updated

## Todo

- [ ] Test add command
- [ ] Test progress command
- [ ] Test done command
- [ ] Test error handling
- [ ] Workflow integration test

## Success Criteria

| Test | Expected |
|------|----------|
| `add <issue>` | Issue xuất hiện trong project, status = Todo |
| `progress <issue>` | Status = In Progress |
| `done <issue>` | Status = Done |
| Invalid issue | Error message rõ ràng |
| Workflow integration | Commands được suggest đúng context |

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Test fail | Medium | Debug script, check gh output |
| Status field mismatch | Medium | Update script với đúng field values |

## Security Considerations

- Không test với production issues quan trọng
- Use test/dummy issue nếu cần

## Next Steps

Sau khi test pass:
1. Update phase statuses trong plan.md → completed
2. Commit skill changes
3. Document trong CHANGELOG nếu có
