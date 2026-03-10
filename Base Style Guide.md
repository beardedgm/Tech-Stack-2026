# Base Style Guide — Signature Dark Aesthetic

A complete, app-agnostic visual design system. Use this document as the master reference to recreate a consistent, refined dark-mode experience across all future applications, dashboards, and landing pages.

---

## 1. Design Philosophy

* **Dark-First:** Deep navy/charcoal backgrounds create a deep, immersive canvas.
* **Refined Accents:** Warm gold acts as the primary interactive thread.
* **Restrained Motion:** Subtle, polished animations (`150ms` - `200ms` eases) that guide the eye without overwhelming it.
* **Universal Accessibility:** Strict adherence to focus-visible rings on all interactive elements and respect for `prefers-reduced-motion`.
* **Systematic Scaling:** All spacing, sizing, and elevation rely on predictable mathematical scales.

---

## 2. Design Tokens (The DNA)

### Color Palette

**Backgrounds (Deepest → Lightest)**
* `--bg-primary` (`#0c0d12`): Application background, the deepest canvas layer.
* `--bg-secondary` (`#12131a`): Alternating page sections, secondary navs, dropdown menus.
* `--bg-surface` (`#161720`): Input fields, inner surface areas.
* `--bg-card` (`#1a1b24`): Standard cards, settings panels, and modals.
* `--bg-elevated` (`#22232e`): Hover states on cards, skeleton shimmer bases, popovers.

**Text & Content**
* `--text-primary` (`#e8e8ec`): Headings, main body text.
* `--text-secondary` (`#8e8fa1`): Labels, descriptions, subtitles.
* `--text-tertiary` (`#8b8c9e`): Metadata, counters, tertiary UI text.
* `--text-muted` (`#838495`): Hints, placeholders, disabled states.
* `--text-on-accent` (`#0c0d12`): High-contrast dark text used strictly on top of gold accent backgrounds.

**Signature Accent**
* `--accent` (`#c9a267`): Primary buttons, active tabs, main links.
* `--accent-hover` (`#dbb57e`): Hover state for accent elements.
* `--accent-muted` (`rgba(201, 162, 103, 0.12)`): Active chip backgrounds, subtle highlights, focus ring spreads.

**Semantic States**
* `--error` (`#e5484d`): Destructive actions, validation errors.
* `--success` (`#30a46c`): Success banners, verified states, positive trends.
* `--warning` (`#f5a623`): Warning states, usage alerts.
* `--warning-muted` (`#e8a838`): Non-critical warnings or secondary alerts.

**Status & Data Sequences**
*Mapped from previous tier colors to generic sequence scales for charts, badges, and levels.*
* `--status-1`: `#30a46c` (Green)
* `--status-2`: `#3498db` (Blue)
* `--status-3`: `#d4873f` (Amber)
* `--status-4`: `#bb7fe2` (Purple)
* `--status-5`: `#e5484d` (Red)
* `--status-premium`: `#FFD700` (Gold)

**Borders**
* `--border` (`#1f2029`): Default borders for cards, inputs, and dividers.
* `--border-hover` (`#2e2f3d`): Interactive border hover states.

---

### Typography System

**Font Stacks**
* **UI (Default):** `'Inter', -apple-system, sans-serif`. Weights: 400, 500, 600.
* **Display:** `'Cinzel', serif` or project-specific stylized font. Weight: 700.

**Typographic Scale**
| Role | Size | Weight | Color | Letter Spacing |
| :--- | :--- | :--- | :--- | :--- |
| **Display 1** (Hero) | `3.25rem` | 700 | `--text-primary` | `-0.015em` |
| **Heading 1** (Page Title) | `1.75rem` | 600 | `--text-primary` | `-0.01em` |
| **Heading 2** (Section) | `1.375rem` | 600 | `--text-primary` | `-0.01em` |
| **Heading 3** (Card Title) | `1rem` | 600 | `--text-primary` | — |
| **Subtitle** | `0.875rem` | 400 | `--text-secondary` | — |
| **Body Default** | `0.875rem` | 400 | `--text-primary` | — |
| **Label / Medium** | `0.8125rem` | 500 | `--text-secondary` | — |
| **Caption / Small** | `0.75rem` | 400 | `--text-tertiary` | — |
| **Overline** (Uppercase) | `0.6875rem` | 600 | `--text-tertiary` | `0.08em` |

---

### Global Scales (Spacing, Radii, Elevation)

**Spacing System (8-Point Grid)**
Used for all margins, paddings, and flex/grid gaps.
* `--space-xs`: `0.25rem` (4px)
* `--space-sm`: `0.5rem` (8px)
* `--space-md`: `0.75rem` (12px)
* `--space-lg`: `1rem` (16px)
* `--space-xl`: `1.5rem` (24px)
* `--space-2xl`: `2rem` (32px)
* `--space-3xl`: `3rem` (48px)

**Border Radii**
* `--radius-sm`: `6px` (Chips, small buttons, inner elements)
* `--radius-md`: `8px` (Inputs, standard buttons, menus)
* `--radius-lg`: `12px` (Cards, modals, large surfaces)
* `--radius-full`: `9999px` (Pills, avatars)

**Elevation & Z-Index**
* `--shadow-sm`: `0 1px 2px rgba(0,0,0,0.3)` (Buttons, resting UI)
* `--shadow-md`: `0 2px 8px rgba(0,0,0,0.4)` (Card hover, dropdowns)
* `--shadow-lg`: `0 8px 24px rgba(0,0,0,0.5)` (Modals, floating navs)
* `--z-dropdown`: `50`
* `--z-sticky`: `100`
* `--z-overlay`: `900`
* `--z-modal`: `1000`

---

## 3. UI Primitives (The Bricks)

### Interactive States
* **Hover:** `border-color` shifts to `--border-hover`, `background` shifts to `--bg-elevated`. Transition: `150ms ease`.
* **Focus-Visible (Global):** `outline: none`, `border-color: var(--accent)`, `box-shadow: 0 0 0 2px var(--accent-muted)`.
* **Disabled:** `opacity: 0.4`, `cursor: not-allowed`, `pointer-events: none`.

### Button System
* **Primary (CTA):** Gold background (`--accent`), dark text (`--text-on-accent`), no border. Hover: `--accent-hover`.
* **Default (Ghost):** Card background (`--bg-card`), border (`--border`), primary text (`--text-primary`). Hover: `--border-hover`, `--bg-elevated`.
* **Danger:** Transparent background, error text (`--error`), transparent border. Hover: error border, 8% red background opacity.

**Button Sizes:**
* `Large`: Padding `0.75rem`, Text `1rem`, Radius `8px`
* `Base`: Padding `0.5rem 1rem`, Text `0.875rem`, Radius `8px`
* `Small`: Padding `0.3rem 0.625rem`, Text `0.8125rem`, Radius `6px`

### Input Primitives
* **Text/Select:** Background `--bg-surface`, Border `--border`, Radius `--radius-md`, Padding `0.5rem 0.75rem`.
* **Validation Error:** Border shifts to `--error`, background shifts to 2% red opacity.

### Surface Primitives
* **Standard Card:** Background `--bg-card`, Border `--border`, Radius `--radius-lg`, Padding `1.25rem 1.5rem`.
* **Modal Container:** Max-width bounds, Background `--bg-card`, Border `--border`, Radius `--radius-lg`, Shadow `--shadow-lg`.
