---
title: "Convert Vietnamese to English"
description: "Replace all Vietnamese text in code and mock data with English for global market"
status: pending
priority: P2
effort: 2h
branch: main
tags: [i18n, localization, refactor]
created: 2026-01-02
---

# Convert Vietnamese Text to English

## Overview
Replace all Vietnamese text in codebase with English equivalents. App targets global market, needs English-only UI.

## Scope
- **In scope**: Code files, mock data, UI constants
- **Out of scope**: CLAUDE.md, README.md, docs/ (keep Vietnamese for internal communication)

## Affected Files Summary
- UI Constants: 1 file
- Mock Data: 4 files
- Components: ~10 files
- **Total**: ~16 files

## Implementation Phases

| Phase | Description | Status | Effort |
|-------|-------------|--------|--------|
| [Phase 1](phase-01-ui-constants.md) | Update UI constants (labels, status names) | pending | 30m |
| [Phase 2](phase-02-mock-data.md) | Update mock data files | pending | 45m |
| [Phase 3](phase-03-components.md) | Update component hardcoded strings | pending | 30m |
| [Phase 4](phase-04-verification.md) | Verify no VN text remains | pending | 15m |

## Key Translation Mapping

### Status/Priority Labels
| Vietnamese | English |
|------------|---------|
| Cần làm | To Do |
| Đang làm | In Progress |
| Cần sửa | Revision |
| Hoàn thành | Done |
| Gấp | Urgent |
| Cao | High |
| Bình thường | Normal |
| Thấp | Low |

### User Names
| Vietnamese | English |
|------------|---------|
| Trần Văn Admin | John Smith |
| Nguyễn Thị Hoa | Jane Doe |
| Lê Văn Dũng | Mike Johnson |
| Phạm Minh Tuấn | Tom Wilson |
| Hoàng Thị Lan | Emily Brown |
| Vũ Đức Anh | David Lee |

## Success Criteria
- [ ] No Vietnamese characters in apps/web/src/**
- [ ] App renders correctly with English text
- [ ] All mock data uses English names/descriptions

## References
- [Brainstorm Report](../reports/brainstorm-260102-1500-convert-vn-to-en.md)
