# Phase 01: Prerequisites

## Context

- [plan.md](plan.md)
- Skill: `.claude/skills/synkao-github-skill/`

## Overview

| Key | Value |
|-----|-------|
| Ngày | 2024-12-24 |
| Priority | High |
| Status | pending |
| Estimate | 5 mins |

## Key Insights

- Token hiện tại thiếu scopes: `read:project`, `project`
- Cần refresh token để có quyền access GitHub Projects API
- Không cần tạo project mới (đã có project #2)

## Requirements

1. `gh` CLI đã cài đặt
2. Đã login với `gh auth login`
3. Token có scopes: `read:project`, `project`

## Implementation Steps

### Step 1: Refresh gh token

```bash
gh auth refresh -s read:project -s project
```

Browser sẽ mở để authorize scopes mới.

### Step 2: Verify access

```bash
# List projects
gh project list --owner synkao

# View project details
gh project view 2 --owner synkao
```

### Step 3: Get Status field info

```bash
gh project field-list 2 --owner synkao --format json
```

Ghi lại tên field Status (thường là "Status") và các option values.

## Todo

- [ ] Run `gh auth refresh -s read:project -s project`
- [ ] Verify `gh project list --owner synkao` works
- [ ] Get Status field name và options

## Success Criteria

- `gh project list --owner synkao` trả về project #2
- `gh project view 2 --owner synkao` hiển thị chi tiết
- Biết được Status field ID và option values

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Token refresh fail | High | Re-login với `gh auth login` |
| Project private | Medium | Ensure user là member của org |

## Security Considerations

- Không commit token vào repo
- Chỉ request scopes cần thiết

## Next Steps

Sau khi hoàn thành → [phase-02-script-development.md](phase-02-script-development.md)
