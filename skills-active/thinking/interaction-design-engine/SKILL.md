---
name: interaction-design-engine
description: Defines precise interaction patterns for UI — transitions, motion, hover/active/focus states, and user feedback. Outputs implementable specs, not UX theory.
type: thinking
---

# Interaction Design Engine

## Purpose

Produce concrete interaction specifications that engineers can implement directly — exact durations, easing curves, state styles, and feedback patterns.

## When to Use

- Defining motion and animation standards for a product
- Specifying hover, active, focus, and disabled states for components
- Designing loading, success, and error feedback flows
- Auditing a UI for missing or inconsistent interaction states

---

## Process

### Step 1 — Gather Context

Ask (or infer):
- Product type (dashboard, e-commerce, consumer app, B2B SaaS)
- Tone (snappy and minimal, expressive and playful, calm and professional)
- Primary platform (web, iOS, Android)
- Performance constraints (low-end devices, slow network, accessibility needs)

---

### Step 2 — Transition System

#### Duration Scale

| Token          | Duration | Use |
|----------------|----------|-----|
| `duration-fast`  | 100ms  | Micro-interactions (checkbox toggle, tooltip appear) |
| `duration-base`  | 200ms  | Standard UI transitions (hover, state change) |
| `duration-slow`  | 300ms  | Larger element transitions (modal enter, panel slide) |
| `duration-enter` | 350ms  | Page transitions, drawers, complex enter animations |
| `duration-exit`  | 200ms  | Exits are always faster than enters |

**Rule:** Exit animations use `duration-exit` regardless of the enter duration. Fast exits feel responsive; slow exits feel sluggish.

#### Easing Curves

| Token           | Curve                        | Use |
|-----------------|------------------------------|-----|
| `ease-standard` | `cubic-bezier(0.4, 0, 0.2, 1)` | Default for most transitions |
| `ease-enter`    | `cubic-bezier(0, 0, 0.2, 1)`   | Elements entering the screen |
| `ease-exit`     | `cubic-bezier(0.4, 0, 1, 1)`   | Elements leaving the screen |
| `ease-spring`   | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Playful bounce (avatars, toggles) |
| `ease-linear`   | `linear`                       | Progress bars, spinners only |

**Rule:** Never use `ease-in` for enters or `ease-out` for exits — they feel wrong directionally.

---

### Step 3 — Interactive States

Define all 5 states for every interactive component.

#### Buttons

| State      | Style |
|------------|-------|
| Default    | `bg-primary-500`, white text |
| Hover      | `bg-primary-600`, box-shadow: `0 2px 8px rgba(0,0,0,0.12)` |
| Active     | `bg-primary-700`, scale: `0.97`, shadow: none |
| Focus      | Outline: `2px solid primary-500`, offset: `2px` |
| Disabled   | `opacity: 0.4`, `cursor: not-allowed`, no hover effect |

**Rules:**
- Active state scale (`0.97`) must use `transform: scale()` — not padding changes.
- Focus outline must be visible in both light and dark mode.
- Disabled elements must NOT show hover states (pointer-events: none is not sufficient — check CSS specificity).

#### Inputs

| State      | Style |
|------------|-------|
| Default    | Border: `neutral-300`, bg: `white` |
| Hover      | Border: `neutral-400` |
| Focus      | Border: `primary-500`, ring: `3px primary-100` |
| Error      | Border: `error`, ring: `3px error-light` |
| Disabled   | `bg-neutral-50`, `opacity: 0.6`, `cursor: not-allowed` |

#### Links & Text Interactions

- Underline on hover (not by default for navigation links)
- Color shift: `neutral-700` → `primary-600` on hover
- Transition: `color 150ms ease-standard`

---

### Step 4 — Motion Principles

**4 core principles:**

1. **Purposeful** — every animation communicates something (state change, relationship, direction). No decorative motion.
2. **Proportional** — small elements move fast, large elements move slower. A tooltip: 100ms. A full-page transition: 350ms.
3. **Directional** — motion implies spatial relationships. Drawers slide from their origin edge. Modals rise from center or bottom. Dropdowns expand from their anchor.
4. **Interruptible** — any in-progress animation must be safely cancellable when the user acts again. Never block input during transitions.

**Specific motion patterns:**

| Element       | Enter                              | Exit |
|---------------|------------------------------------|------|
| Modal         | Fade + scale from `0.95` → `1.0`  | Fade + scale to `0.95` |
| Drawer (right)| Slide from `translateX(100%)`     | Slide to `translateX(100%)` |
| Tooltip       | Fade in, no scale                 | Instant dismiss (no animation) |
| Dropdown      | `scaleY(0.9)` + fade from origin  | Fade only, 150ms |
| Toast         | Slide in from bottom/top          | Fade out, shrink height |
| Skeleton      | Shimmer: `background-position` linear loop, 1.5s |

---

### Step 5 — User Feedback Patterns

#### Loading States

| Pattern         | When to Use |
|-----------------|-------------|
| Skeleton screen | Initial page/section load — preserves layout, reduces perceived wait |
| Inline spinner  | Button-triggered async actions (form submit, save) |
| Progress bar    | File uploads, multi-step processes with known progress |
| Overlay spinner | Full-page blocking operations (use sparingly) |

**Button loading pattern:**
```
[Default] → [Loading: spinner + "Saving…"] → [Success: checkmark, 1.5s] → [Default]
```
- Disable button immediately on click (prevent double-submit)
- Show spinner inside button, not replace button
- On success: brief green checkmark before returning to default state

#### Success Feedback

- Inline confirmation preferred over toast for form submissions
- Toast for background operations (file saved, email sent)
- Toast duration: 4 seconds auto-dismiss, with manual close option
- Color: `color-success` background or border
- Icon: checkmark (✓), not text alone

#### Error Feedback

| Error Type     | Pattern |
|----------------|---------|
| Field validation | Inline below field, red border, error icon |
| Form submission | Summary banner at top of form + field highlights |
| Network error  | Toast with retry action |
| Destructive action failed | Modal with explanation, no auto-dismiss |

**Rules:**
- Never clear an error until the user fixes the cause — not on re-focus, not on keypress.
- Show error messages in plain language. "Something went wrong" is not an error message.
- Error state must be distinct from warning state visually (not just color — use icon + pattern).

#### Empty States

- Always explain why it's empty and what the user can do
- Include a clear primary action (CTA)
- Use illustration only if it adds clarity, not decoration

---

## Output Format

Produce specs in this order:
1. Transition token table (duration + easing)
2. State-by-state style table per component type
3. Motion pattern table (enter/exit per element)
4. Feedback flow per pattern (loading → success → error)

Use tables and code snippets. Skip prose where a table works.
