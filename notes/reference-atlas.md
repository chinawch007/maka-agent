# Reference Layout — UI Atlas

Systematic reverse-engineering pass of the external reference layout used as the
visual north star for Maka. Built from the extracted bundle at
`/tmp/qoder-extracted/out/renderer/`.

This document is the source of truth for Phase 2 reapplication. Every section is
backed by direct evidence pulled from `assets/globals-UfMzAdiO.css` (265KB
minified, beautified to `/tmp/reference-globals.css` = 8032 lines) and
`index.html`. Where an entry says "from JS chunk", it's a string literal extracted
from `index-C02cm0ok.js`.

**Use the reference's mechanics — never copy product names.** Naming hygiene
rule (memory `project_reference_layout_lane`) requires every artifact in Maka's
codebase use neutral terms ("reference layout", "upstream design"). The product
name appears in this atlas only because it's needed to identify what we're
analyzing; it must not leak into commit messages, JSDoc, or shipped CSS comments.

---

## 1. Theme System

`<html data-theme="...">` carries 8 themes:

| Theme              | Loading bg  | Loading fg  | Notes                                   |
|--------------------|-------------|-------------|-----------------------------------------|
| `light`            | `#ffffff`   | `#09090b`   | Default on Windows/Linux                |
| `dark`             | `#09090b`   | `#fafafa`   |                                         |
| `light-glass`      | transparent | `#09090b`   | macOS default; uses Electron vibrancy   |
| `dark-glass`       | `#171716`   | `#eeeeeb`   |                                         |
| `classic-light`    | `#fafaf9`   | `#171716`   | Warmer neutral                           |
| `classic-dark`     | `#09090b`   | `#fafafa`   |                                         |
| `light-parchment`  | `#FAF9F6`   | `#202116`   | Warm bone canvas                        |
| `dark-parchment`   | `#1E1C18`   | `#FAF9F6`   |                                         |

Initial body fallback (before React mounts): `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto`.

macOS default is `light-glass` (transparent + native vibrancy). Maka does **not**
currently use Electron vibrancy — adding it is a separate platform concern.

### Glass theme implication

`light-glass`/`dark-glass` make the window content render against the system
material (translucent panels with desktop bleed-through). The whole page chrome
is calibrated to look correct on a vibrancy substrate. If we adopt `light-glass`
later, all the colors below need an alpha pass too.

---

## 2. Font System

CSS custom props in light-parchment:

| Token              | Value                                                                  |
|--------------------|------------------------------------------------------------------------|
| `--font-ui`        | `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, 'PingFang SC', 'Microsoft YaHei', 'Noto Sans CJK SC'` |
| `--font-sans`      | `ui-sans-serif, system-ui, sans-serif, …emoji fallbacks`                |
| `--font-mono`      | `ui-monospace, 'Cascadia Code', 'SF Mono', Menlo, Consolas, …` (light) / `ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, …` (dark) |
| `--font-body`      | `var(--font-ui)` in sans mode; `'Libre Baskerville', Georgia, 'Songti SC', …` in serif mode |

Plus a downloaded variable: `InstrumentSans-VariableFont_wdth_wght-BjF37L9x.ttf` —
not declared via `@font-face` in the snippet I checked; investigate further if a
specific surface uses Instrument Sans explicitly.

### Font weight scale
- `--font-weight-normal: 400`
- `--font-weight-medium: 500`
- `--font-weight-semibold: 600`
- `--font-weight-bold: 700`

### Type scale (Tailwind v4 defaults)
| Token         | Value      | Px equivalent | Line height                  |
|---------------|------------|---------------|------------------------------|
| `--text-xs`   | `.75rem`   | 12px          | `calc(1 / .75)` = 16px       |
| `--text-sm`   | `.875rem`  | 14px          | `calc(1.25 / .875)` = 20px   |
| `--text-base` | `1rem`     | 16px          | `1.5`                        |
| `--text-lg`   | `1.125rem` | 18px          | `calc(1.75 / 1.125)` = 28px  |
| `--text-xl`   | `1.25rem`  | 20px          | `calc(1.75 / 1.25)` = 28px   |
| `--text-2xl`  | `1.5rem`   | 24px          | `calc(2 / 1.5)` = 32px       |
| `--text-3xl`  | `1.875rem` | 30px          | `1.2`                        |

### Runtime size controller — `data-font-size`

Set on `<html>` via `localStorage.getItem('qoder-work-ui:fontStyle')` / `'…:fontSize'`. Three steps:

| Mode                                              | Chat input  | Prose body  | Code in prose |
|---------------------------------------------------|-------------|-------------|---------------|
| `[data-font-size=small]`                          | 13 / 24     | 13          | 12            |
| `[data-font-size=medium]` (default; `:not([…])`)  | 14 / 24     | (default)   | 13            |
| `[data-font-size=large]`                          | 16 / 28     | 16          | (larger)      |

Default font-style: `sans`. `[data-font-style=serif]` swaps body to `Libre
Baskerville` (Songti SC for CJK).

Maka currently has neither `data-font-size` nor `data-font-style` runtime
controls — those are an unimplemented feature, not a styling miss.

---

## 3. Color Tokens (light-parchment as canonical reference)

Reference uses an Ant-Design-style semantic naming with `--color-bg-*`,
`--color-fill-*`, `--color-text-*`, `--color-border-*`, `--color-link-*` families.
Below is the light-parchment family — the warm-canvas theme that maka's
"reference-layout lane" targets.

### Backgrounds
- `--color-bg-base: #faf9f6` — canvas / page background (warm bone)
- `--color-bg-container: #faf9f6` — primary card background
- `--color-bg-elevated: #f7f4ef` — slightly deeper for elevated surfaces
- `--color-bg-highlight: #c9c4b8` — selected row fill
- `--color-bg-highlight-hover: #d4c9bc` — hover fill (warm taupe)
- `--color-bg-layout: #fdfcfa` — outer shell (sidebar / chrome plate; lightest)

### Fill scale (subtle tints used for chips, sub-panels)
- Tertiary fill: `var(--color-fill-tertiary)` — most subtle (think 4-5% black)

### Borders
- `--color-border` — primary
- `--color-border-secondary` — slightly lighter
- `--color-border-tertiary` — the lightest (used on the workbench card border)

### Light (default) family condensed
- `--color-bg-base: #fff`
- `--color-bg-container: #fff`
- `--color-bg-elevated: #f9f9f9`
- `--color-bg-highlight-hover: #e0e0e0` (or `#eeeeeb`)

### Dark family condensed
- `--color-bg-base: #171716`
- `--color-bg-container: #171716`
- `--color-bg-elevated: #22221f`

Maka's tokens (`maka-tokens.css`) use `oklch()` — semantically equivalent. The
gap is in **the chrome variables** (`--agents-*`) below — those don't exist in
maka.

---

## 4. Page Architecture Tokens (THE critical findings)

### Content area tokens
- `--agents-content-area-radius: 6px` — the universal corner radius for cards,
  panels, content area outer ring. **Six pixels, not 10-12.**
- `--agents-content-area-gap: 4px` — the gap between sibling cards / panels.
  **Four pixels.** Cards sit very close, almost touching.
- `--agents-content-area-bg` — `var(--color-bg-container)` in default/dark,
  `#fff` in light, `transparent` in glass themes
- `--agents-layout-bg` — `var(--color-bg-layout)` for shell; in glass:
  `#0006` (dark) / `#ffffff80` (light); in parchment: a 172° linear gradient
  from `--color-fill-tertiary` to `--color-bg-container`

### The page layout DOM (inferred from CSS selectors)

```
[data-agents-page]                              ← root, theme-scoped
  └── .agents-layout-root
       └── .agents-layout-body
            ├── (left sidebar, sometimes glass material)
            └── .agents-content-area            ← right column / main pane
                 └── [data-agents-view=cron|skills|chat|im_hub|new_chat]
                      └── .agents-inner-view-clamp   ← THE page card itself
                           ├── (page header / hero)
                           └── content (workbench-split-root, dual-card-row, etc.)
```

### The signature card chrome

When **`light-parchment` / `dark-parchment` theme** + view is `skills` / `cron` /
`im_hub` (the three "wide" views, vs chat which is a centered conversation
column), `.agents-inner-view-clamp` paints as the lifted page card:

```css
border: 1px solid var(--color-border-tertiary);   /* often 50-60% via color-mix */
border-radius: var(--agents-content-area-radius); /* 6px */
min-width: 350px;
```

### Layout primitives inside the page card

- `.workbench-split-root` / `.workbench-split-view` — split-pane container
  (uses resizable panels; `gap: var(--agents-content-area-gap)` = 4px)
- `.agents-dual-card-row` — alternative dual-card row
- `.workbench-card` — content card inside the split
- `.aux-panel` — sibling card / sub-panel
- `.task-monitor-right-panel` — special right-side monitor variant (no border /
  no shadow / `--color-fill-tertiary` background to recede)

### The universal card recipe

```css
border: 1px solid var(--color-border-tertiary);
border-radius: var(--agents-content-area-radius);  /* 6px */
background: var(--color-bg-base);
box-shadow: 0 1px 2px #0c0c0d0a;                   /* black @ ~4% */
```

This is the formula for `.workbench-card`, `.aux-panel`, and
`.agents-chat-panel`. It is the **single most-reused look** in the bundle and
the gap with maka — maka's cards have 8-10px radii, 0.04 foreground shadows,
and use `var(--border)` (more like `--color-border-secondary`). Visually maka's
cards read as "bigger / softer / more isolated" than reference's tight 6px /
hairline-tertiary cards.

### The `--agents-layout-bg` (sidebar / chrome) tinted gradient

```css
background: linear-gradient(
  172deg,
  var(--color-fill-tertiary, #f9f9f9) 19.61%,
  var(--color-bg-container, #fff)     81.35%
);
```

Subtle warm gradient on the outer shell that maka doesn't replicate.

---

## 5. Shadow Vocabulary (curated; 5 patterns)

The bundle has hundreds of `box-shadow:` declarations, but they reduce to these
5 patterns:

| Pattern                                            | Use                                |
|----------------------------------------------------|-----------------------------------|
| `box-shadow: 0 1px 2px #0c0c0d0a`                  | **Universal card lift** (the only shadow on default cards) |
| `box-shadow: 0 6px 20px -12px #0c0c0d33`           | Medium float (popovers, sticky panels) |
| `box-shadow: 0 8px 24px -16px #0000006b`           | Deep float (dark mode equivalent) |
| `box-shadow: 0 4px 12px #00000026`                 | Anchored anchored alerts |
| 3-layer triple-glow (10px wing-out variants)       | Composer hero glow (multiple tints) |

Notes:
- `#0c0c0d0a` = `rgba(12, 12, 13, 0.04)` — about 4% black.
- All non-glow shadows use plain hex with alpha suffix (not `rgba(…)`), keeping
  the declarations short.

Maka uses `oklch(from var(--foreground) l c h / 0.04)` style alpha shadows
mostly — semantically the same lift, just expressed via oklch. The numeric gap:
maka's lift cards use 0.10 in the second shadow layer (per `da1a40f1` removal).
Reference would have used the `0 1px 2px` 4% only and let the 1px border do
the visual separation.

---

## 6. Keyframes (all 13)

| Keyframe                          | Spec (from earlier samples)                                                   |
|-----------------------------------|------------------------------------------------------------------------------|
| `agents-fade-in`                  | 200ms ease-out opacity 0→1                                                   |
| `agents-slide-up`                 | 200ms translateY 8px → 0 with opacity 0→1                                   |
| `streaming-fade-in`               | `.18s ease-out` opacity 0→1 (per-token fade on streaming Markdown)          |
| `workbench-card-scale-in`         | `.35s cubic-bezier(.16,1,.3,1) .15s both` scale .97→1, opacity 0→1         |
| `chat-input-glow-pulse`           | `4.8s ease-in-out infinite` (composer ambient breathing — already ported)    |
| `indeterminate-slide`             | `1.4s ease-in-out infinite` `left -40% → 100%` (indeterminate progress bar) |
| `pointer-poke`                    | `4s linear infinite`, micro-shake to draw cursor attention                    |
| `pulse`                           | `2s cubic-bezier(.4,0,.6,1) infinite` (Tailwind built-in)                    |
| `spin`                            | `1s linear infinite` (Tailwind built-in)                                     |
| `chroma-slide`                    | Color shift on tiled chrome (not yet sampled)                                |
| `connectors-lightning-pulse`      | Connector banner emphasis                                                    |
| `industry-guide-marquee`          | Horizontal marquee for the industry-guide page                               |
| `agents-chat-design-launch-narrow`| Chat hero launch (likely the empty-chat slide-up sequence)                   |

**Already ported into maka:**
- `agents-fade-in` → kept as ambient utility
- `agents-slide-up` → `maka-message-row-enter` (rounds 24/30) and `maka-hero-enter` (round 29/30)
- `workbench-card-scale-in` → `maka-tool-card-enter` (round 23/30)
- `chat-input-glow-pulse` → `maka-composer-glow-pulse`

**Not yet ported (good Phase 2 candidates):**
- `streaming-fade-in` — would be a real per-token fade in streamed Markdown
- `indeterminate-slide` — loading bar for retry / pending states
- `chroma-slide` / `connectors-lightning-pulse` — non-core, skip
- `industry-guide-marquee` — non-applicable (no Maka counterpart)

---

## 7. Page-Level Component Names (from JS chunks)

Strings literal-grep'd from `index-C02cm0ok.js`. These are React component or
className tokens — useful for naming Maka equivalents.

| Reference string                            | Likely surface                                  |
|---------------------------------------------|------------------------------------------------|
| `agents-settings`                           | Settings page root                              |
| `agents-settings-page-heading`              | Settings page H1                                |
| `agents-settings-page-title-narrow`         | Settings page H1 (narrow viewport variant)      |
| `agents-settings-page-description`          | Settings page lede                              |
| `workbench-aux-stack-from-panel-layout`     | Aux panel stack adapter                         |
| `workbench-root-from-panel-layout`          | Main workbench wrapper                          |
| `workbench-tab`                             | Tab control variant                             |
| `cron-tasks`                                | Cron page tasks list                            |
| `skills-navigation`                         | Skills page nav                                 |
| `agents-rename-subchat`                     | Inline rename overlay                           |

The settings page has a dedicated heading/description block (per the H1 + lede
hierarchy below). Cron page has a `cron-tasks` content area for the actual
schedule list.

---

## 8. Data attribute selectors (state vocabulary)

Reference uses `[data-*]` heavily for state. Subset relevant to Maka:

- `[data-agents-page]` — root scope guard (theme-aware blocks)
- `[data-agents-view=cron]` / `[data-agents-view=skills]` / `[data-agents-view=chat]` / `[data-agents-view=new_chat]` / `[data-agents-view=im_hub]` — current route
- `[data-active=true]` — list selection
- `[data-disabled=true]` / `[data-disabled]` — interaction disabled
- `[data-focused=true]` — focus-within (used on the composer)
- `[data-font-size=small|medium|large]` / `[data-font-style=sans|serif]` — runtime typography
- `[data-glass-effects=off]` — disables vibrancy / backdrop blur
- `[data-composer-message]` — composer rich-text shell
- `[data-user-bubble]` — user message bubble (in chat)
- `[data-chat-launch-transition=design]` — initial chat launch animation flag
- `[data-skills-banner]` / `[data-skill-filter-menu]` — skills page chrome
- `[data-canvas-dialog]` / `[data-close-button]` — modal chrome
- `[data-gradual-blur-layer]` — blur scrim
- `[data-banner-logo]` — Settings banner logo

Maka mostly uses `[data-active="true"]` on list rows. There's no `[data-agents-view]` route attribute — module switching uses local React state, not a documented DOM contract. Adopting `[data-agents-view]` would make CSS targeting easier.

---

## 9. Specific gap analysis: Maka vs reference

| Surface              | Reference recipe                                            | Maka current                                                | Gap to close in Phase 2                              |
|----------------------|-------------------------------------------------------------|-------------------------------------------------------------|------------------------------------------------------|
| Page card outer      | 1px `--color-border-tertiary` + 6px radius + `0 1px 2px #0c0c0d0a` | flat (no border / no shadow after da1a40f1)                 | Restore the lift, but tight 6px radius (was 10)      |
| Inter-card gap       | 4px (`--agents-content-area-gap`)                           | 10-14px gap                                                 | Tighten to 4-6px                                     |
| Card shadow          | `0 1px 2px` 4% only — no second softer layer                | Used to have 2-layer (0 1px 2px + 0 8px 24px -16px 10%)     | Single 4% lift; let border do the separation         |
| Sidebar / chrome bg  | 172° linear-gradient warm                                   | Flat `--surface-canvas`                                     | Optional: add the gradient                            |
| Cron page heading    | (Need to extract — JS chunk has `cron-tasks` block)         | h2 24px / 650                                               | Likely larger / display weight                       |
| Settings page        | `agents-settings-page-heading` + lede + `narrow` variant    | Modal popup (still!)                                        | Restructure as inline page in module-main pane       |
| Active list fill     | `var(--color-bg-highlight)` = `#c9c4b8` (warm taupe)        | `oklch(from --foreground l c h / 0.07)` (neutral gray)      | Optional: warm taupe in parchment theme              |
| Glass theme          | Vibrancy + `data-glass-effects` toggle                     | Not implemented                                             | Out of UI scope (Electron platform work)             |
| `data-font-size` ctl | Runtime small / medium / large                              | Not implemented                                             | Future settings surface                              |
| Streaming tokens     | `streaming-fade-in` per-token                               | Bubble-level fade only                                      | Would need ChatMarkdown renderer wrap                |

---

## 10. Phase 2 plan (atlas-driven)

Goal: rewrite Maka's **Plan/Reminder** page and **Settings** surface to match
reference's signature card chrome + heading hierarchy + tight 4-6px geometry.

### 10.1 Plan/Reminder page (PlanReminderPanel)
- Wrap content in a `.maka-page-card` matching the universal recipe (1px
  border-tertiary + 6px radius + 4% lift), with `--agents-content-area-gap`
  equivalent at 4-6px between internal sections.
- Heading: change `.maka-plan-heading h2` from 24/650 to 30px / weight-600
  (reference uses `--text-3xl` for page headings).
- Internal sections render as workbench-card-style sub-cards: 1px border, 6px
  radius, soft lift. Card-vs-card gap = 4-6px.
- Task list = aux-panel pattern: per-row 1px divider (`border-bottom` on
  `:not(:last-child)`), no per-row border or radius (cockpit-mode density per
  taste skills).

### 10.2 Settings (currently a modal!)
- Convert from Base UI Dialog overlay to an inline `.maka-module-main` view.
- Header: `.maka-page-heading` (30px display) + lede paragraph + narrow variant
  (`-title-narrow`) for tighter sidebar widths.
- Nav: left rail (current modal nav) → keep, but as part of a 2-column
  workbench-split-root inside the page card.
- Content panel: right side workbench-card; sub-sections divided by
  `border-bottom` lines, not nested cards.

### 10.3 Shared primitives (CSS-only first)
- Add `.maka-card-base` mixin via custom property: a single rule that defines
  the recipe (1px border-tertiary, 6px radius, 4% lift). Compose into
  `.maka-skill-library`, `.maka-plan-shell`, future settings page.
- Add `.maka-page-heading` typography class (30px / 1.2 line / weight 600).
- Optionally add `[data-maka-view=plan|skills|settings|...]` on a wrapper to
  match reference's `[data-agents-view]` pattern.

### 10.4 Out of Phase 2 scope (file follow-ups)
- Glass theme + Electron vibrancy
- `data-font-size` / `data-font-style` runtime controls
- Streaming-fade-in per-token (requires Markdown renderer surgery)
- The warm taupe `--color-bg-highlight` (parchment-only)

---

## 11. Source pointers (where evidence lives)

- `/tmp/qoder-extracted/out/renderer/index.html` — DOM bootstrap, theme bootstrap, font preload
- `/tmp/reference-globals.css` — beautified CSS (multi-line)
- `/tmp/qoder-extracted/out/renderer/assets/globals-UfMzAdiO.css` — original minified CSS
- `/tmp/qoder-extracted/out/renderer/assets/index-C02cm0ok.js` — main JS bundle (component names live here)
- `/tmp/qoder-extracted/out/renderer/assets/chat-view-inner-Dgjr-1Tu.js` — chat view chunk
- `/tmp/qoder-extracted/out/renderer/assets/chat-markdown-renderer-C-R6Fd8_.js` — Markdown chunk

Grep patterns used for this pass (re-runnable):
```sh
# CSS custom props
grep -E '^[[:space:]]*--[a-z][a-z0-9-]+:' /tmp/reference-globals.css | sort -u
# All keyframes
grep -oE '^[[:space:]]*@keyframes [a-zA-Z][a-zA-Z0-9-]+' /tmp/reference-globals.css | sort -u
# All shadow patterns
grep -oE 'box-shadow:[^;}]+' /tmp/reference-globals.css | sort -u
# Page-level class strings from JS
grep -oE '"(agents|workbench|aux|cron|skills|settings)-[a-z-]+"' /tmp/qoder-extracted/out/renderer/assets/index-C02cm0ok.js | sort -u
```
