# Pixel-Perfect Verification Examples

## Example 1: Dashboard Page

### Input

- **Figma:** `https://figma.com/design/abc123/Dashboard?node-id=1-200`
- **Target:** `http://localhost:3000/dashboard`

### Phase 1: Design Extraction

After calling `get_design_context` and `get_screenshot`, the Design Spec Table:

```markdown
| Element (path)                  | Property         | Design Value         |
|---------------------------------|------------------|----------------------|
| Dashboard > Header > Title      | font-family      | Inter                |
| Dashboard > Header > Title      | font-size        | 28px                 |
| Dashboard > Header > Title      | font-weight      | 700                  |
| Dashboard > Header > Title      | line-height      | 36px                 |
| Dashboard > Header > Title      | color            | #111827              |
| Dashboard > Header > Subtitle   | font-size        | 14px                 |
| Dashboard > Header > Subtitle   | font-weight      | 400                  |
| Dashboard > Header > Subtitle   | color            | #6B7280              |
| Dashboard > Stats Cards         | gap              | 24px                 |
| Dashboard > Stats Cards         | display          | flex                 |
| Stats Card                      | width            | 280px                |
| Stats Card                      | padding          | 24px                 |
| Stats Card                      | border-radius    | 16px                 |
| Stats Card                      | background-color | #FFFFFF              |
| Stats Card                      | box-shadow       | 0 1px 3px #0000001A |
| Stats Card                      | border           | 1px solid #E5E7EB   |
| Stats Card > Label              | font-size        | 12px                 |
| Stats Card > Label              | font-weight      | 500                  |
| Stats Card > Label              | color            | #6B7280              |
| Stats Card > Label              | text-transform   | uppercase            |
| Stats Card > Label              | letter-spacing   | 0.05em               |
| Stats Card > Value              | font-size        | 32px                 |
| Stats Card > Value              | font-weight      | 700                  |
| Stats Card > Value              | color            | #111827              |
| Stats Card > Trend Badge        | padding          | 4px 8px              |
| Stats Card > Trend Badge        | border-radius    | 9999px               |
| Stats Card > Trend Badge        | font-size        | 12px                 |
| Stats Card > Trend Badge        | font-weight      | 500                  |
| Stats Card > Trend Badge (up)   | background-color | #ECFDF5              |
| Stats Card > Trend Badge (up)   | color            | #065F46              |
| Stats Card > Trend Badge (down) | background-color | #FEF2F2              |
| Stats Card > Trend Badge (down) | color            | #991B1B              |
| Table > Header Row              | background-color | #F9FAFB              |
| Table > Header Row              | border-bottom    | 1px solid #E5E7EB   |
| Table > Header Cell             | font-size        | 12px                 |
| Table > Header Cell             | font-weight      | 600                  |
| Table > Header Cell             | color            | #6B7280              |
| Table > Header Cell             | padding          | 12px 16px            |
| Table > Header Cell             | text-transform   | uppercase            |
| Table > Body Cell               | font-size        | 14px                 |
| Table > Body Cell               | font-weight      | 400                  |
| Table > Body Cell               | color            | #111827              |
| Table > Body Cell               | padding          | 16px                 |
| Table > Row                     | border-bottom    | 1px solid #F3F4F6   |
| Primary Button                  | height           | 40px                 |
| Primary Button                  | padding          | 0 16px               |
| Primary Button                  | border-radius    | 8px                  |
| Primary Button                  | background-color | #4F46E5              |
| Primary Button                  | color            | #FFFFFF              |
| Primary Button                  | font-size        | 14px                 |
| Primary Button                  | font-weight      | 600                  |
```

### Phase 2: Browser Measurement

JS measurement results (abbreviated):

```javascript
// Measured via getComputedStyle
{
  "h1.dashboard-title": {
    fontSize: "24px",      // ← differs from 28px
    fontWeight: "600",     // ← differs from 700
    lineHeight: "32px",    // ← differs from 36px
    color: "#111827"       // ✓ matches
  },
  ".stats-card": {
    padding: "20px",       // ← differs from 24px
    borderRadius: "12px",  // ← differs from 16px
    boxShadow: "none",    // ← missing entirely
    border: "1px solid #e5e7eb"  // ✓ matches
  },
  ".stats-card .label": {
    fontSize: "12px",      // ✓
    fontWeight: "500",     // ✓
    letterSpacing: "0.05em", // ✓
    textTransform: "uppercase" // ✓
  },
  ".stats-card .value": {
    fontSize: "28px",      // ← differs from 32px
    fontWeight: "700"      // ✓
  },
  ".btn-primary": {
    height: "36px",        // ← differs from 40px
    padding: "0px 12px",   // ← differs from 0 16px
    borderRadius: "6px",   // ← differs from 8px
    backgroundColor: "#4f46e5", // ✓
    fontSize: "14px",      // ✓
    fontWeight: "500"      // ← differs from 600
  }
}
```

### Phase 3: Diff Table Output

```markdown
## Pixel-Perfect Diff

**Source:** https://figma.com/design/abc123/Dashboard?node-id=1-200
**Target:** http://localhost:3000/dashboard
**Date:** 2025-01-15
**Total discrepancies:** 12

| # | Element | Property | Expected (Figma) | Actual (Browser) | Severity | Selector |
|---|---------|----------|------------------|------------------|----------|----------|
| 1 | Dashboard title | font-size | 28px | 24px | high | h1.dashboard-title |
| 2 | Dashboard title | font-weight | 700 | 600 | medium | h1.dashboard-title |
| 3 | Dashboard title | line-height | 36px | 32px | low | h1.dashboard-title |
| 4 | Stats Card | padding | 24px | 20px | high | .stats-card |
| 5 | Stats Card | border-radius | 16px | 12px | medium | .stats-card |
| 6 | Stats Card | box-shadow | 0 1px 3px rgba(0,0,0,0.1) | none | high | .stats-card |
| 7 | Stats Card value | font-size | 32px | 28px | high | .stats-card .value |
| 8 | Primary Button | height | 40px | 36px | medium | .btn-primary |
| 9 | Primary Button | padding-inline | 16px | 12px | medium | .btn-primary |
| 10 | Primary Button | border-radius | 8px | 6px | medium | .btn-primary |
| 11 | Primary Button | font-weight | 600 | 500 | medium | .btn-primary |
| 12 | Stats cards container | gap | 24px | 16px | medium | .stats-grid |

### Systemic Issues

| Pattern | Affected elements | Fix |
|---------|-------------------|-----|
| Border-radius consistently 4px smaller | Stats Card (16→12), Button (8→6) | Check `--radius-*` tokens: likely `--radius-lg: 12px` should be `16px`, `--radius-md: 6px` should be `8px` |
| Font weights one step lighter | Title (700→600), Button (600→500) | Heading and button weight tokens are one step too light |
| Padding consistently 4px smaller | Stats Card (24→20), Button (16→12) | Spacing scale may use `--space-5: 20px` where design expects `24px` |
```

---

## Example 2: Handling Design Tokens

When the implementation uses CSS variables/tokens, report the token that needs changing:

```markdown
| # | Element | Property | Expected (Figma) | Actual (Browser) | Severity | Selector |
|---|---------|----------|------------------|------------------|----------|----------|
| 1 | Card | border-radius | 16px | 12px | medium | .card |

### Systemic Issues

| Pattern | Affected elements | Fix |
|---------|-------------------|-----|
| `--radius-xl` is 12px, design shows 16px | Card, Modal, Sheet | Update token: `--radius-xl: 12px` → `--radius-xl: 16px` in tokens file |
```

The implementation agent should:
1. Grep for the CSS variable definition
2. Update the token value
3. All consumers automatically get the fix

---

## Example 3: Component with States

When a Figma component has hover/focus/active variants:

```markdown
| # | Element | Property | Expected (Figma) | Actual (Browser) | Severity | Selector |
|---|---------|----------|------------------|------------------|----------|----------|
| 1 | Button (default) | background-color | #4F46E5 | #4F46E5 | — | .btn |
| 2 | Button:hover | background-color | #4338CA | #3730A3 | medium | .btn:hover |
| 3 | Button:focus | outline | 2px solid #818CF8 | 2px solid #4F46E5 | medium | .btn:focus-visible |
| 4 | Button:active | background-color | #3730A3 | #4F46E5 | high | .btn:active |
| 5 | Button:disabled | opacity | 0.5 | 0.4 | low | .btn:disabled |
```

---

## Example 4: Missing Elements

When an element exists in Figma but is absent from the DOM:

```markdown
| # | Element | Property | Expected (Figma) | Actual (Browser) | Severity | Selector |
|---|---------|----------|------------------|------------------|----------|----------|
| 1 | Empty state illustration | presence | exists | MISSING | critical | — |
| 2 | Table footer pagination | presence | exists | MISSING | critical | — |
| 3 | Card hover shadow | box-shadow (hover) | 0 4px 12px rgba(0,0,0,0.15) | none (no hover state) | high | .card:hover |
```

---

## Example 5: Responsive Breakpoint Diff

When design has multiple breakpoints:

```markdown
## Pixel-Perfect Diff — Desktop (1440px)

| # | Element | Property | Expected | Actual | Severity | Selector |
|---|---------|----------|----------|--------|----------|----------|
| 1 | Grid | grid-template-columns | repeat(3, 1fr) | repeat(3, 1fr) | — | .grid |
| 2 | Sidebar | width | 280px | 260px | medium | aside.sidebar |

## Pixel-Perfect Diff — Tablet (768px)

| # | Element | Property | Expected | Actual | Severity | Selector |
|---|---------|----------|----------|--------|----------|----------|
| 1 | Grid | grid-template-columns | repeat(2, 1fr) | repeat(3, 1fr) | critical | .grid |
| 2 | Sidebar | display | none | block | critical | aside.sidebar |
```

---

## Anti-Patterns (what NOT to report)

### Not a bug: equivalent values
```
line-height: 1.5 (design) vs 24px (browser on 16px text) → equivalent, skip
font-weight: Bold (design) vs 700 (browser) → equivalent, skip
padding: 24 (design) vs 24px (browser) → equivalent, skip
color: #fff (design) vs #ffffff (browser) → equivalent, skip
```

### Not a bug: within tolerance
```
font-size: 14px (design) vs 13.6px (browser due to rem rounding) → within ±1px, skip
gap: 16px (design) vs 16.5px (browser) → within ±1px, skip
```

### Not a bug: browser defaults on non-specified properties
```
If Figma shows no shadow on an element and browser has none → don't report "none vs none"
If Figma auto-layout has no explicit align and browser has align-items: stretch → browser default, skip
```
