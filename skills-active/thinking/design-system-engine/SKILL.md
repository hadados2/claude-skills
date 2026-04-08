---
name: design-system-engine
description: Generates a complete, production-ready design system spec — spacing, typography, color, and layout — from a product brief or existing UI. Outputs concrete tokens and rules, not theory.
type: thinking
---

# Design System Engine

## Purpose

Produce a full design system specification that a developer can implement directly — no vague guidance, only concrete values and rules.

## When to Use

- Starting a new product and need a consistent design foundation
- Auditing an existing UI for inconsistency
- Defining design tokens before or during component development
- Syncing design and engineering on shared language (spacing, type scale, colors)

---

## Process

### Step 1 — Gather Context

Ask (or infer from the brief):
- Product type (dashboard, marketing, mobile-first, SaaS, consumer app)
- Brand tone (minimal, bold, playful, enterprise)
- Existing colors or brand assets (if any)
- Target platform (web, iOS, Android, cross-platform)

---

### Step 2 — Spacing System

Use a **4pt base grid**. All spacing values are multiples of 4.

| Token       | Value | Use |
|-------------|-------|-----|
| `space-1`   | 4px   | Micro gaps (icon padding, input insets) |
| `space-2`   | 8px   | Tight spacing (label-to-input, badge padding) |
| `space-3`   | 12px  | Small internal padding |
| `space-4`   | 16px  | Standard component padding |
| `space-6`   | 24px  | Section breathing room |
| `space-8`   | 32px  | Card/container padding |
| `space-12`  | 48px  | Major section gaps |
| `space-16`  | 64px  | Page-level vertical rhythm |

**Rules:**
- Never use arbitrary pixel values. All spacing must map to a token.
- Between sibling elements: use `space-2` to `space-4`.
- Between sections: use `space-8` to `space-16`.
- Internal component padding: `space-3` or `space-4`.

---

### Step 3 — Typography Hierarchy

Use a **modular scale** (ratio 1.25 recommended for web; 1.333 for display-heavy).

| Token       | Size  | Weight | Line Height | Use |
|-------------|-------|--------|-------------|-----|
| `text-xs`   | 11px  | 400    | 1.4         | Labels, metadata |
| `text-sm`   | 13px  | 400    | 1.5         | Supporting text, captions |
| `text-base` | 16px  | 400    | 1.6         | Body copy |
| `text-md`   | 18px  | 500    | 1.5         | Emphasized body, card titles |
| `text-lg`   | 22px  | 600    | 1.4         | Section headings (H3) |
| `text-xl`   | 28px  | 700    | 1.3         | Page headings (H2) |
| `text-2xl`  | 36px  | 700    | 1.2         | Hero headings (H1) |
| `text-3xl`  | 48px+ | 800    | 1.1         | Display / marketing |

**Rules:**
- Max 3 font sizes visible at once in any single view.
- Heading hierarchy must be sequential (H1 → H2 → H3 — no skipping).
- Line height decreases as size increases (large text needs tighter leading).
- Font family: one sans-serif for UI, optionally one serif/display for marketing.

---

### Step 4 — Color System

Define 3 categories of colors with semantic tokens.

#### Primary (Brand)
| Token              | Value (example) | Use |
|--------------------|-----------------|-----|
| `color-primary-50` | #EFF6FF         | Light tints, hover backgrounds |
| `color-primary-500`| #3B82F6         | Default interactive elements |
| `color-primary-700`| #1D4ED8         | Hover/active states |
| `color-primary-900`| #1E3A8A         | Dark text on light bg |

#### Neutral (UI Chrome)
| Token               | Value (example) | Use |
|---------------------|-----------------|-----|
| `color-neutral-50`  | #F9FAFB         | Page background |
| `color-neutral-100` | #F3F4F6         | Card/surface background |
| `color-neutral-300` | #D1D5DB         | Borders, dividers |
| `color-neutral-500` | #6B7280         | Placeholder, secondary text |
| `color-neutral-700` | #374151         | Body text |
| `color-neutral-900` | #111827         | Headings, high-contrast text |

#### Semantic (Feedback)
| Token             | Value (example) | Use |
|-------------------|-----------------|-----|
| `color-success`   | #10B981         | Confirmations, positive states |
| `color-warning`   | #F59E0B         | Caution states |
| `color-error`     | #EF4444         | Errors, destructive actions |
| `color-info`      | #3B82F6         | Informational messages |

**Rules:**
- Never hardcode hex values in components — always use tokens.
- Ensure 4.5:1 contrast ratio minimum for text on backgrounds (WCAG AA).
- Dark mode: map semantic tokens to dark equivalents, don't invert arbitrarily.

---

### Step 5 — Layout Consistency Rules

**Grid:**
- Web: 12-column grid, `24px` gutters, `16px`–`32px` outer margins.
- Mobile: 4-column grid, `16px` gutters, `16px` margins.
- Dashboard: use CSS Grid with named areas; avoid deeply nested flexbox.

**Container widths:**
| Name     | Max Width | Use |
|----------|-----------|-----|
| `narrow` | 640px     | Forms, auth screens |
| `prose`  | 720px     | Long-form reading |
| `default`| 1024px    | Standard pages |
| `wide`   | 1280px    | Dashboards, data tables |
| `full`   | 100%      | Hero sections, media |

**Component sizing:**
- Touch targets: minimum 44×44px (mobile), 32×32px (desktop).
- Input heights: 36px (compact), 40px (default), 48px (large).
- Button heights: match input heights for alignment.
- Icons: 16px (inline), 20px (default), 24px (prominent).

---

## Output Format

Produce a structured spec in this order:
1. Design token table (spacing, type, color) — copy-pasteable
2. Layout grid summary
3. Component sizing reference
4. 3–5 specific rules the team must follow

Keep it scannable. Use tables. Avoid paragraphs where a table works.
