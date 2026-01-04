# Phase 05: Comments Section

## Context

- Parent: [plan.md](plan.md)
- Dependencies: Phase 01 (types)
- Docs: [codebase-summary.md](../../docs/codebase-summary.md)

## Overview

| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P2 |
| Effort | 1h |
| Implementation Status | Pending |
| Review Status | Pending |

**Objective:** Create CommentsSection cho designer/reviewer conversation với optional version linking.

## Key Insights

1. Comments: Simple text conversation
2. Version link: Optional select to reference specific version
3. Display: Newest first hoặc oldest first (TBD)
4. Form: Text input + optional version dropdown + send button

## Requirements

### CommentsSection Component

- Comment list với avatar, name, time
- Optional "on Version X" badge if linked
- Input form at bottom
- Version selector dropdown

### Comment Item

- Author avatar + name
- Timestamp
- Content
- Optional version badge (link to version)

### Comment Form

- Text input (or textarea)
- Optional: Version selector dropdown
- Send button

## Related Code Files

| File | Action | Description |
|------|--------|-------------|
| `drawer/comments-section.tsx` | Create | Comment list + form |
| `task-detail-drawer.tsx` | Modify | Add section |

## Implementation Steps

### Step 1: Create comment item component

```tsx
function CommentItem({ comment }: { comment: MockComment }) {
  const author = mockUsers.find(u => u.id === comment.userId);
  const version = comment.designVersionId
    ? mockDesignVersions.find(v => v.id === comment.designVersionId)
    : null;

  return (
    <div className="flex gap-3">
      <Avatar>...</Avatar>
      <div>
        <div className="flex items-center gap-2">
          <span className="font-medium">{author?.name}</span>
          {version && <Badge>Version {version.version}</Badge>}
          <span className="text-muted-foreground text-xs">{formatTime}</span>
        </div>
        <p>{comment.content}</p>
      </div>
    </div>
  );
}
```

### Step 2: Create comment form

```tsx
function CommentForm({ taskId, versions }: Props) {
  const [content, setContent] = useState('');
  const [versionId, setVersionId] = useState<string | null>(null);

  const handleSubmit = () => {
    // Demo: just log or show toast
    console.log('New comment:', { content, versionId });
    setContent('');
  };

  return (
    <div className="flex gap-2">
      <Input value={content} onChange={...} placeholder="Add a comment..." />
      <Select value={versionId} onValueChange={...}>
        <option value="">No version</option>
        {versions.map(v => <option key={v.id}>Version {v.version}</option>)}
      </Select>
      <Button onClick={handleSubmit}>Send</Button>
    </div>
  );
}
```

### Step 3: Create main section

Combine list và form.

### Step 4: Add to drawer

Insert after DesignVersionsSection.

## Todo List

- [ ] Create CommentItem component
- [ ] Create CommentForm component
- [ ] Create CommentsSection container
- [ ] Style comment list
- [ ] Add version selector
- [ ] Add to drawer
- [ ] Test form interaction

## Success Criteria

- [ ] Comments display với avatar
- [ ] Version badge shows when linked
- [ ] Form accepts input
- [ ] Version selector works
- [ ] Styling matches design

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Long comments | Low | Truncate or wrap |
| Many comments | Low | ScrollArea với max height |

## Security Considerations

None - demo UI only.

## Next Steps

→ Phase 06: Quick Status & Links
