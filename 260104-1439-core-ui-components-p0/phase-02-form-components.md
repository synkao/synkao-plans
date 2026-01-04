# Phase 2: Form Components

## Context
- **Issue:** #9
- **Parent Plan:** [plan.md](./plan.md)
- **Reference:** `apps/web/src/components/ui/input.tsx`, `select.tsx`

## Overview
| Field | Value |
|-------|-------|
| Date | 2026-01-04 |
| Priority | P0 |
| Effort | 4h |
| Status | completed |
| Dependencies | cmdk, react-dropzone, date-fns, react-day-picker |

## Key Insights

Từ codebase analysis:
- Input đã có: focus-visible, aria-invalid states
- Select dùng Radix UI primitives
- Pattern: `data-slot` cho all elements
- Validation: react-hook-form + zod (đã installed)

## Decisions (from user)
- **FileUploader preview:** Chỉ images (không preview PDF, docs, etc.)
- **Form inputs glass effect:** Thử apply glass effect cho form inputs

## Requirements

1. **Combobox** - Searchable single select
2. **Multi-select** - Multiple selection với tags
3. **FileUploader** - Drag-drop file upload
4. **DatePicker** - Calendar date selection
5. **Calendar** - Calendar primitive (cho DatePicker)

## Dependencies Installation

```bash
pnpm add -F @synkao/web cmdk react-dropzone date-fns react-day-picker
```

## Architecture

### Combobox
```tsx
interface ComboboxProps {
  options: { value: string; label: string }[];
  value?: string;
  onValueChange?: (value: string) => void;
  placeholder?: string;
  searchPlaceholder?: string;
  emptyMessage?: string;
}
```

### Multi-select
```tsx
interface MultiSelectProps {
  options: { value: string; label: string }[];
  value?: string[];
  onValueChange?: (value: string[]) => void;
  placeholder?: string;
  maxSelected?: number;
}
```

### FileUploader
```tsx
interface FileUploaderProps {
  value?: File[];
  onValueChange?: (files: File[]) => void;
  accept?: Record<string, string[]>;
  maxFiles?: number;
  maxSize?: number; // bytes
  showPreview?: boolean;
}
```

### DatePicker
```tsx
interface DatePickerProps {
  value?: Date;
  onValueChange?: (date: Date | undefined) => void;
  placeholder?: string;
  disabled?: boolean;
}
```

## Related Code Files

- `apps/web/src/components/ui/input.tsx` - Input styles reference
- `apps/web/src/components/ui/select.tsx` - Select pattern reference
- `apps/web/src/components/ui/popover.tsx` - Popover for dropdowns
- `apps/web/src/components/ui/button.tsx` - Button styles

## Implementation Steps

### Step 1: Install dependencies (5m)

```bash
cd apps/web && pnpm add cmdk react-dropzone date-fns react-day-picker
```

### Step 2: Create calendar.tsx (30m)

Based on shadcn/ui calendar pattern:
- Use react-day-picker
- Custom styling với Tailwind
- Support single/range mode

```tsx
// Key structure
<DayPicker
  showOutsideDays
  className={cn("p-3", className)}
  classNames={{
    months: "flex flex-col sm:flex-row space-y-4 sm:space-x-4 sm:space-y-0",
    month: "space-y-4",
    // ... more classNames
  }}
  components={{
    IconLeft: () => <ChevronLeftIcon className="size-4" />,
    IconRight: () => <ChevronRightIcon className="size-4" />,
  }}
  {...props}
/>
```

### Step 3: Create date-picker.tsx (30m)

Combine Popover + Calendar:

```tsx
function DatePicker({ value, onValueChange, placeholder = "Pick a date" }) {
  return (
    <Popover>
      <PopoverTrigger asChild>
        <Button variant="outline">
          <CalendarIcon className="mr-2 size-4" />
          {value ? format(value, "PPP") : placeholder}
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-auto p-0">
        <Calendar mode="single" selected={value} onSelect={onValueChange} />
      </PopoverContent>
    </Popover>
  );
}
```

### Step 4: Create combobox.tsx (1h)

Use cmdk + Popover:

```tsx
function Combobox({ options, value, onValueChange, placeholder }) {
  const [open, setOpen] = useState(false);

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button variant="outline" role="combobox" aria-expanded={open}>
          {value ? options.find(o => o.value === value)?.label : placeholder}
          <ChevronsUpDownIcon className="ml-2 size-4" />
        </Button>
      </PopoverTrigger>
      <PopoverContent>
        <Command>
          <CommandInput placeholder="Search..." />
          <CommandList>
            <CommandEmpty>No results found.</CommandEmpty>
            <CommandGroup>
              {options.map((option) => (
                <CommandItem
                  key={option.value}
                  onSelect={() => {
                    onValueChange(option.value);
                    setOpen(false);
                  }}
                >
                  {option.label}
                  <CheckIcon className={cn("ml-auto", value === option.value ? "opacity-100" : "opacity-0")} />
                </CommandItem>
              ))}
            </CommandGroup>
          </CommandList>
        </Command>
      </PopoverContent>
    </Popover>
  );
}
```

### Step 5: Create multi-select.tsx (1h)

Similar to combobox but với tags:

```tsx
function MultiSelect({ options, value = [], onValueChange, maxSelected }) {
  // Render tags for selected items
  // Popover với CommandList for options
  // Support remove individual tags
  // Keyboard navigation
}
```

### Step 6: Create file-uploader.tsx (1h)

Use react-dropzone:

```tsx
function FileUploader({ value = [], onValueChange, accept, maxFiles, maxSize }) {
  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    accept,
    maxFiles,
    maxSize,
    onDrop: (acceptedFiles) => onValueChange([...value, ...acceptedFiles]),
  });

  return (
    <div
      {...getRootProps()}
      className={cn(
        "border-2 border-dashed rounded-lg p-8 text-center cursor-pointer",
        isDragActive && "border-primary bg-primary/5"
      )}
    >
      <input {...getInputProps()} />
      <UploadCloudIcon className="mx-auto size-12 text-muted-foreground" />
      <p>Drag & drop files here, or click to select</p>
    </div>
  );
}
```

## Todo List

- [x] Install dependencies (cmdk, react-dropzone, date-fns, react-day-picker)
- [x] Create calendar.tsx
- [x] Create date-picker.tsx
- [x] Create combobox.tsx
- [x] Create multi-select.tsx
- [x] Create file-uploader.tsx
- [ ] Add to design-system showcase (Phase 4)
- [ ] Test keyboard navigation (Phase 4)
- [ ] Test form integration (Phase 4)

## Success Criteria

- [x] Combobox opens/closes, filters options
- [x] Multi-select tags render, removable
- [x] FileUploader accepts drag-drop
- [x] FileUploader shows file list
- [x] DatePicker opens calendar
- [x] DatePicker selects date correctly
- [x] All components keyboard accessible (built on Radix/cmdk primitives)

## Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| cmdk styling conflicts | Medium | Medium | Scope styles |
| Large bundle from date-fns | Low | Low | Tree-shaking |
| FileUploader file type edge cases | Medium | Low | Validate accept config |

## Security Considerations

- FileUploader: Validate file types client-side (server validation required)
- FileUploader: maxSize limit để prevent large uploads
- No actual upload logic - just UI preparation

## Next Steps

Sau khi complete Phase 2:
1. Proceed to Phase 3: Data Table
2. Create form examples với react-hook-form
