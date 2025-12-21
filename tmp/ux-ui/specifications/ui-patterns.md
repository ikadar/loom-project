---
id: UI-PATTERNS
status: draft
derived-at: 2025-12-21T17:30:00Z
loom-version: "1.0"
structured-interview:
  decisions:
    CC-1: Skeleton
    CC-2: Hybrid (Toast + Inline + Banner)
    CC-3: On blur
    CC-4: Contextual
tags:
  - specification
  - ux-ui
  - patterns
---

# UI Patterns – Flux Scheduling UI

Cross-cutting UI patterns for this project. Components reference these patterns instead of defining their own loading, error, empty, and validation behaviors.

---

## Overview

This document defines reusable UI patterns that apply across all components:

| Category | Primary Pattern | Fallback |
|----------|-----------------|----------|
| Loading | Skeleton | Spinner for actions |
| Errors | Hybrid | Context-dependent |
| Validation | On blur | Real-time for format |
| Empty States | Contextual | First-use vs no-results |

---

## Loading States

### Pattern: PAT-LOADING-SKELETON {#pat-loading-skeleton}

**Use when:** Loading lists, grids, cards, or content-heavy areas

**Implementation:**
- Gray animated placeholder shapes matching content dimensions
- Pulse animation (1.5s cycle)
- Match approximate layout of loaded content

**CSS (Tailwind):**
```css
.skeleton {
  @apply bg-slate-700/50 animate-pulse rounded;
}

.skeleton-text {
  @apply h-4 bg-slate-700/50 animate-pulse rounded w-3/4;
}

.skeleton-tile {
  @apply h-20 w-full bg-slate-700/50 animate-pulse rounded-lg;
}
```

**Usage Examples:**

```tsx
// Loading job list
function JobsListSkeleton() {
  return (
    <div className="space-y-2">
      {[...Array(5)].map((_, i) => (
        <div key={i} className="skeleton h-16 w-full" />
      ))}
    </div>
  );
}

// Loading scheduling grid
function GridSkeleton() {
  return (
    <div className="flex gap-2">
      {[...Array(6)].map((_, i) => (
        <div key={i} className="skeleton-tile h-[1920px] w-60" />
      ))}
    </div>
  );
}
```

**When loading:**
- Jobs list: Show 5 skeleton cards
- Scheduling grid: Show skeleton columns
- Job details panel: Show skeleton task list
- Timeline: Show skeleton hour markers

---

### Pattern: PAT-LOADING-SPINNER {#pat-loading-spinner}

**Use when:** Loading buttons, inline actions, small areas, API operations

**Implementation:**
- Centered spinner icon
- Size variants: `sm` (16px), `md` (20px), `lg` (24px)
- Replace button text during loading
- Disable interaction during load

**CSS (Tailwind):**
```css
.spinner {
  @apply animate-spin rounded-full border-2 border-slate-600 border-t-white;
}

.spinner-sm { @apply h-4 w-4; }
.spinner-md { @apply h-5 w-5; }
.spinner-lg { @apply h-6 w-6; }
```

**Usage Examples:**

```tsx
// Button loading state
<Button disabled={isLoading}>
  {isLoading ? <Spinner className="spinner-sm" /> : 'Save'}
</Button>

// Inline loading
<div className="flex items-center gap-2">
  <Spinner className="spinner-sm" />
  <span className="text-slate-400">Saving...</span>
</div>
```

**When to use spinner over skeleton:**
- Save/submit actions
- API mutations (create, update, delete)
- Drag-drop operations (during API call)
- Compact areas (< 100px)

---

### Pattern: PAT-LOADING-OVERLAY {#pat-loading-overlay}

**Use when:** Blocking operations that require user to wait

**Implementation:**
- Semi-transparent overlay over affected area
- Centered spinner with optional message
- Disable all interactions in area
- Use sparingly (blocking is disruptive)

**CSS (Tailwind):**
```css
.loading-overlay {
  @apply absolute inset-0 bg-slate-900/80 flex items-center justify-center z-50;
}
```

**Usage Examples:**

```tsx
// Station column during compact operation
<StationColumn>
  {isCompacting && (
    <div className="loading-overlay">
      <Spinner className="spinner-lg" />
    </div>
  )}
  {children}
</StationColumn>
```

---

## Error States

### Pattern: PAT-ERROR-TOAST {#pat-error-toast}

**Use when:** Transient errors, action failures, network errors

**Implementation:**
- Position: bottom-center (matches dark UI better than top-right)
- Auto-dismiss: 5 seconds (or sticky for critical)
- Dismissible via X button or swipe
- Variant: `destructive` (red accent)

**Structure:**
```tsx
import { toast } from 'sonner'; // or your toast library

// API error
toast.error('Failed to save assignment', {
  description: 'Please try again or check your connection.',
  duration: 5000,
});

// Network error
toast.error('Connection lost', {
  description: 'Changes will sync when reconnected.',
  duration: Infinity, // sticky
});
```

**When to use:**
- API save/update failures
- Network errors
- Optimistic update rollbacks
- Rate limiting
- Session expiry warnings

---

### Pattern: PAT-ERROR-INLINE {#pat-error-inline}

**Use when:** Form validation errors, field-specific errors

**Implementation:**
- Position: below the input field
- Red text (`text-red-400`), small font (`text-xs`)
- Error icon optional (AlertCircle)
- Persist until error is corrected
- Animate in (fade + slide)

**CSS (Tailwind):**
```css
.field-error {
  @apply text-xs text-red-400 mt-1 flex items-center gap-1;
}

.input-error {
  @apply border-red-500 focus:ring-red-500;
}
```

**Structure:**
```tsx
<FormField>
  <Label>Scheduled Time</Label>
  <Input
    className={cn(error && 'input-error')}
    value={value}
    onChange={handleChange}
  />
  {error && (
    <p className="field-error">
      <AlertCircle className="h-3 w-3" />
      {error.message}
    </p>
  )}
</FormField>
```

**When to use:**
- Form field validation
- Input format errors
- Required field missing
- Value out of range

---

### Pattern: PAT-ERROR-BANNER {#pat-error-banner}

**Use when:** System-wide errors, degraded service, maintenance

**Implementation:**
- Position: top of page (below header) or top of section
- Persistent until resolved or dismissed
- Variants: `error` (red), `warning` (orange/amber)
- Include action if applicable (retry, learn more)

**CSS (Tailwind):**
```css
.banner-error {
  @apply bg-red-900/50 border border-red-700 text-red-200 p-3 rounded-lg;
}

.banner-warning {
  @apply bg-amber-900/50 border border-amber-700 text-amber-200 p-3 rounded-lg;
}
```

**Structure:**
```tsx
<Banner variant="error" dismissible onDismiss={handleDismiss}>
  <div className="flex items-center gap-3">
    <AlertCircle className="h-5 w-5 text-red-400" />
    <div>
      <p className="font-medium">Unable to connect to server</p>
      <p className="text-sm opacity-80">Changes may not be saved.</p>
    </div>
    <Button variant="outline" size="sm" onClick={retry}>
      Retry
    </Button>
  </div>
</Banner>
```

**When to use:**
- Server unreachable
- API degraded performance
- Scheduled maintenance
- Version update required

---

### Pattern: PAT-ERROR-PAGE {#pat-error-page}

**Use when:** Fatal errors, 404, 500, no permission

**Implementation:**
- Full page takeover
- Illustration or icon
- Clear message explaining what happened
- Action button (retry, go home, contact support)

**Structure:**
```tsx
<ErrorPage>
  <ErrorIllustration src="/error-500.svg" />
  <ErrorTitle>Something went wrong</ErrorTitle>
  <ErrorDescription>
    We couldn't load the scheduler. This has been reported automatically.
  </ErrorDescription>
  <ErrorActions>
    <Button onClick={retry}>Try Again</Button>
    <Button variant="ghost" onClick={goHome}>Go Home</Button>
  </ErrorActions>
</ErrorPage>
```

---

## Empty States

### Pattern: PAT-EMPTY-FIRST-USE {#pat-empty-first-use}

**Use when:** User has no data yet, first time using feature

**Implementation:**
- Friendly, encouraging tone
- Optional illustration (keep consistent with brand)
- Clear CTA to add first item
- Explain value proposition briefly

**Structure:**
```tsx
<EmptyState variant="first-use">
  <EmptyIcon icon={Calendar} className="h-12 w-12 text-slate-500" />
  <EmptyTitle>No jobs scheduled</EmptyTitle>
  <EmptyDescription>
    Select a job from the list to start scheduling tasks on the grid.
  </EmptyDescription>
  <EmptyAction onClick={selectFirstJob}>
    Select a Job
  </EmptyAction>
</EmptyState>
```

**Specific implementations:**
- **Jobs list empty**: "No jobs available. Jobs appear when orders are received."
- **Unscheduled tasks empty**: "All tasks scheduled! Great work."
- **Grid empty for day**: "No assignments for this day. Drag tasks to schedule."

---

### Pattern: PAT-EMPTY-NO-RESULTS {#pat-empty-no-results}

**Use when:** Search/filter returns no results

**Implementation:**
- Lighter treatment than first-use
- Explain no matches found
- Suggest adjusting search/filters
- Provide clear action to reset

**Structure:**
```tsx
<EmptyState variant="no-results">
  <EmptyIcon icon={SearchX} className="h-10 w-10 text-slate-600" />
  <EmptyTitle>No matching jobs</EmptyTitle>
  <EmptyDescription>
    No jobs match your current filters. Try adjusting your search.
  </EmptyDescription>
  <EmptyAction variant="ghost" onClick={clearFilters}>
    Clear Filters
  </EmptyAction>
</EmptyState>
```

---

### Pattern: PAT-EMPTY-ERROR {#pat-empty-error}

**Use when:** Data failed to load

**Implementation:**
- Error-styled empty state (not full error page)
- Retry action prominently displayed
- Brief explanation
- Consider showing cached/stale data if available

**Structure:**
```tsx
<EmptyState variant="error">
  <EmptyIcon icon={AlertCircle} className="h-10 w-10 text-red-400" />
  <EmptyTitle>Failed to load jobs</EmptyTitle>
  <EmptyDescription>
    There was a problem loading the job list.
  </EmptyDescription>
  <EmptyAction onClick={refetch}>
    Try Again
  </EmptyAction>
</EmptyState>
```

---

## Validation Feedback

### Pattern: PAT-VALIDATION-ONBLUR {#pat-validation-onblur}

**Primary validation strategy for this project.**

**Use when:** All form fields by default

**Implementation:**
- Validate when field loses focus
- Don't show errors while typing (unless previously invalid)
- Show loading indicator for async validation
- Persist error until field is re-validated

**Behavior:**
```typescript
const [touched, setTouched] = useState(false);
const [error, setError] = useState<string | null>(null);

const handleBlur = async () => {
  setTouched(true);
  const validationError = await validate(value);
  setError(validationError);
};

// Only show error if field has been touched
const showError = touched && error;
```

**Visual states:**
| State | Border | Message |
|-------|--------|---------|
| Pristine | Default | None |
| Touched + Valid | Default or green | None |
| Touched + Invalid | Red | Error message below |
| Validating | Default + spinner | "Checking..." |

---

### Pattern: PAT-VALIDATION-REALTIME {#pat-validation-realtime}

**Use when:** Simple format validation only

**Implementation:**
- Debounce: 300ms after typing stops
- Only for format checks (email, phone, required)
- Avoid for expensive or async validation
- Clear error immediately when format becomes valid

**Behavior:**
```typescript
const debouncedValidate = useMemo(
  () => debounce((value: string) => {
    const error = validateFormat(value);
    setError(error);
  }, 300),
  []
);

const handleChange = (value: string) => {
  setValue(value);
  debouncedValidate(value);
};
```

**When to use real-time:**
- Email format
- Phone number format
- Required field (show immediately if cleared)
- Character count limits

---

### Pattern: PAT-VALIDATION-SUBMIT {#pat-validation-submit}

**Use when:** Form-level validation, multi-field checks

**Implementation:**
- Validate all fields on submit attempt
- Scroll to first error
- Focus first error field
- Show all errors simultaneously
- Prevent submission until valid

**Behavior:**
```typescript
const handleSubmit = async (e: FormEvent) => {
  e.preventDefault();

  const errors = validateAll(formData);

  if (Object.keys(errors).length > 0) {
    setErrors(errors);
    // Focus first error field
    const firstErrorField = Object.keys(errors)[0];
    document.getElementById(firstErrorField)?.focus();
    return;
  }

  await submit(formData);
};
```

---

## Transitions & Animations

### Pattern: PAT-TRANSITION-FADE {#pat-transition-fade}

**Use when:** Content appearing/disappearing, page transitions

**Implementation:**
- Duration: 150ms
- Easing: ease-out (enter), ease-in (exit)
- Use for: modals, panels, overlays

**CSS:**
```css
.fade-enter {
  opacity: 0;
}
.fade-enter-active {
  opacity: 1;
  transition: opacity 150ms ease-out;
}
.fade-exit {
  opacity: 1;
}
.fade-exit-active {
  opacity: 0;
  transition: opacity 150ms ease-in;
}
```

---

### Pattern: PAT-TRANSITION-SLIDE {#pat-transition-slide}

**Use when:** Panels, drawers, list items

**Implementation:**
- Duration: 200ms
- Direction: based on context (up for new items, left/right for panels)
- Combine with fade for polish

**CSS:**
```css
.slide-up-enter {
  opacity: 0;
  transform: translateY(8px);
}
.slide-up-enter-active {
  opacity: 1;
  transform: translateY(0);
  transition: all 200ms ease-out;
}
```

---

### Pattern: PAT-TRANSITION-COLLAPSE {#pat-transition-collapse}

**Use when:** Expanding/collapsing sections, accordions, removed items

**Implementation:**
- Animate height from 0 to auto (use max-height trick)
- Duration: 200ms
- Exit: collapse + fade

**CSS:**
```css
.collapse-enter {
  max-height: 0;
  opacity: 0;
  overflow: hidden;
}
.collapse-enter-active {
  max-height: 500px; /* larger than content */
  opacity: 1;
  transition: all 200ms ease-out;
}
```

---

### Pattern: PAT-TRANSITION-SCALE {#pat-transition-scale}

**Use when:** Modals, dialogs, popovers, tooltips

**Implementation:**
- Scale from 95% to 100% + fade
- Duration: 200ms
- Origin: center or pointer position

**CSS:**
```css
.scale-enter {
  opacity: 0;
  transform: scale(0.95);
}
.scale-enter-active {
  opacity: 1;
  transform: scale(1);
  transition: all 200ms ease-out;
}
```

---

## Design Token Reference

Reference tokens from [design-tokens.md](design-tokens.md):

**Colors:**
| Token | Value | Usage |
|-------|-------|-------|
| Error | `red-500` (#ef4444) | Error states, destructive |
| Warning | `amber-500` (#f59e0b) | Warnings, bypass mode |
| Success | `green-500` (#22c55e) | Success, valid states |
| Loading | `slate-700` | Skeleton backgrounds |
| Muted | `slate-400` | Disabled, placeholder text |

**Timing:**
| Token | Value | Usage |
|-------|-------|-------|
| Fast | 100ms | Hover states, micro-interactions |
| Normal | 150ms | Standard transitions |
| Slow | 200ms | Complex animations, modals |
| Debounce | 300ms | Input validation debounce |

**Easing:**
| Token | Value | Usage |
|-------|-------|-------|
| Default | ease-out | Most transitions |
| Enter | ease-out | Elements appearing |
| Exit | ease-in | Elements leaving |

---

## Component Pattern References

Components should reference patterns in their specs:

```yaml
# Example from component-api.md
---
id: COMP-JOB-DETAILS-PANEL
---

Cross-cutting patterns:
  loading: → ui-patterns.md#pat-loading-skeleton
  error: → ui-patterns.md#pat-error-inline
  empty: → ui-patterns.md#pat-empty-first-use
  validation: → ui-patterns.md#pat-validation-onblur
```

### Quick Reference Table

| Component | Loading | Error | Empty | Validation |
|-----------|---------|-------|-------|------------|
| JobsList | Skeleton | Banner | First-use | - |
| JobDetailsPanel | Skeleton | Inline | First-use | - |
| SchedulingGrid | Skeleton | Banner | No-results | - |
| TaskTile | Spinner | Toast | - | - |
| StationColumn | Overlay | Toast | - | - |

---

## Accessibility Considerations

All patterns must respect:

1. **Reduced Motion** - Disable animations when `prefers-reduced-motion: reduce`
2. **Focus Management** - Move focus appropriately when content changes
3. **Announcements** - Use aria-live for dynamic error/loading states
4. **Color Independence** - Icons accompany color indicators

```css
@media (prefers-reduced-motion: reduce) {
  .skeleton,
  .spinner,
  [class*="transition"],
  [class*="animate"] {
    animation: none !important;
    transition: none !important;
  }
}
```

---

## Related Documents

- [Design Tokens](design-tokens.md)
- [Component API](component-api.md)
- [State Machines](state-machines.md)
- [Accessibility Audit](tests/accessibility-audit.md)
