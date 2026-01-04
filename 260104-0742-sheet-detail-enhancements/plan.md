---
title: "Sheet Detail Enhancements"
description: "Enhance TaskDetailDrawer với print specs, reference files, comments, thumbnails và quick actions"
status: done
priority: P2
effort: 7h
branch: main
tags: [design-module, drawer, ui-enhancement, phase-02]
created: 2026-01-04
---

# Sheet Detail Enhancements Plan

## Overview

Enhance TaskDetailDrawer component với các features mới:
- Print Specs section (key-value display + edit modal)
- Reference Files section (grid thumbnails + lightbox)
- Comments section (conversation + version linking)
- Enhanced Design Versions (thumbnails + lightbox)
- Quick Status Actions (buttons thay dropdown)
- Enhanced Task Info (order/item links, deadline warnings)

## Current State

```
TaskDetailDrawer
├── Header (task code, product, customer)
├── TaskInfoSection (status, priority, source, assignee, deadline, workspace)
├── DesignNotesSection (read-only notes)
├── DesignVersionsSection (version list without thumbnails)
└── TimelineSection (activity timeline)
```

## Target State

```
TaskDetailDrawer
├── Header + QuickStatusActions ← NEW
├── TaskInfoSection (enhanced: order/item links, deadline warning)
├── PrintSpecsSection ← NEW (replaces DesignNotes)
├── ReferenceFilesSection ← NEW
├── DesignVersionsSection (enhanced: thumbnails + lightbox)
├── CommentsSection ← NEW
└── TimelineSection (existing)
```

## Implementation Phases

| Phase | Name | Effort | Status | Progress |
|-------|------|--------|--------|----------|
| 01 | [Types & Mock Data](phase-01-types-mock-data.md) | 1h | Done | 100% |
| 02 | [Print Specs Section](phase-02-print-specs-section.md) | 1h | Done | 100% |
| 03 | [Reference Files Section](phase-03-reference-files-section.md) | 1.5h | Done | 100% |
| 04 | [Design Versions Enhancement](phase-04-design-versions-enhancement.md) | 1h | Done | 100% |
| 05 | [Comments Section](phase-05-comments-section.md) | 1h | Done | 100% |
| 06 | [Quick Status & Links](phase-06-quick-status-links.md) | 0.5h | Done | 100% |
| 07 | [Remove Design Notes](phase-07-remove-design-notes.md) | 5m | Done | 100% |
| 08 | [Reject/Approve Design Versions](phase-08-reject-approve-versions.md) | 1h | Done | 100% |

## Dependencies

- shadcn/ui: dialog, sheet, scroll-area, badge, button, input
- react-dropzone (new): upload UI
- picsum.photos: mock placeholder images

## Success Criteria

- [x] Print specs hiển thị key-value + edit modal
- [x] Reference files grid 3 columns + lightbox
- [x] Design versions có thumbnail preview
- [x] Comments section với list và input
- [x] Quick status action buttons
- [x] Order/item links clickable
- [x] Deadline warning badges
- [x] Upload zone UI (demo only)
- [x] DesignNotesSection removed (replaced by PrintSpecs)
- [x] Reject/Approve buttons cho SUBMITTED versions
- [x] Reject dialog với reason input

## Reports

- [Brainstorm Report](brainstorm-report.md) - Initial scope definition
