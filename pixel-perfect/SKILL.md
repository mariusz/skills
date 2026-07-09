---
name: pixel-perfect
description: >
  Systematic pixel-perfect design verification — extracts design specs from Figma (via MCP API + screenshots)
  and compares every CSS property against the live browser implementation using computer-use.
  Outputs a structured diff table consumable by an implementation agent.
  TRIGGER when: user mentions "pixel-perfect", "design diff", "design QA", "check against Figma",
  "verify implementation", "design verification", "CSS diff", "Figma vs browser",
  or wants to compare a live build against a Figma design.
  DO NOT TRIGGER when: user wants to CREATE a Figma design, wants visual regression testing (screenshot diffing),
  wants CSS linting/optimization, or wants functional testing.
---

# Pixel-Perfect Design Verification

Extract design specs from Figma → measure live implementation → produce a structured diff table for an implementation agent to fix.

**Output:** A markdown table with columns: Element | Property | Expected | Actual | Severity | Selector

---

## Phase 0: Inputs

Collect from user:

1. **Figma URL** — with node-id pointing to the frame/page to verify (e.g. `figma.com/design/FILE_KEY/Name?node-id=X-Y`)
2. **Target URL** — the running build to audit (localhost or deployed)
3. **Scope** — full page or specific section? Component-level or page-level?
4. **Known exceptions** — intentional deviations to skip

> **HARD STOP:** Must have both a Figma URL and a running target. No guessing values.

---

## Phase 1: Design Extraction (Figma)

### Step 1A: Get structured data via Figma MCP

```
1. get_design_context(url=FIGMA_URL)
   → Returns: node tree with properties (fills, typography, spacing, effects, layout)
   → Extract ALL values into the Design Spec Table (see format below)

2. get_metadata(url=FIGMA_URL)
   → Returns: hierarchy, dimensions, positions
   → Use for: absolute sizes, padding inference, gap values
```

### Step 1B: Visual verification via screenshot

```
3. get_screenshot(url=FIGMA_URL)
   → Visual reference for layout, alignment, relative sizing
   → Cross-check against structured data (catches auto-layout gaps that API may not surface clearly)
```

### Step 1C: Build the Design Spec Table

Extract EVERY property for EVERY visible element. Be exhaustive:

```markdown
| Element (path)              | Property         | Design Value        |
|-----------------------------|------------------|---------------------|
| Header > Title              | font-size        | 32px                |
| Header > Title              | font-weight      | 700                 |
| Header > Title              | line-height      | 40px                |
| Header > Title              | letter-spacing   | -0.02em             |
| Header > Title              | color            | #1A1A1A             |
| Header > Subtitle           | font-size        | 16px                |
| Card                        | padding          | 24px                |
| Card                        | border-radius    | 12px                |
| Card                        | background       | #FFFFFF             |
| Card                        | box-shadow       | 0 2px 8px #0000001A |
| Card > Items                | gap              | 16px                |
| Button                      | height           | 40px                |
| Button                      | padding-inline   | 16px                |
| Button                      | border-radius    | 8px                 |
| Button                      | font-weight      | 600                 |
```

### Properties Checklist (extract ALL that apply per element)

**Typography:**
- [ ] font-family
- [ ] font-size
- [ ] font-weight
- [ ] line-height
- [ ] letter-spacing
- [ ] text-transform
- [ ] text-decoration
- [ ] text-align

**Colors:**
- [ ] color (text)
- [ ] background-color / background (gradients)
- [ ] border-color
- [ ] opacity

**Spacing:**
- [ ] padding (all sides)
- [ ] margin (all sides)
- [ ] gap (flex/grid)
- [ ] width / height (fixed dimensions)
- [ ] min-width / min-height / max-width / max-height

**Layout:**
- [ ] display (flex/grid/block)
- [ ] flex-direction
- [ ] align-items
- [ ] justify-content
- [ ] flex-wrap
- [ ] grid-template-columns / rows

**Borders & Corners:**
- [ ] border-width
- [ ] border-style
- [ ] border-radius (all corners if different)

**Effects:**
- [ ] box-shadow
- [ ] backdrop-filter
- [ ] filter
- [ ] overflow

**Alignment & Position:**
- [ ] position (relative/absolute/sticky)
- [ ] top/right/bottom/left
- [ ] z-index

### Guardrail: Minimum Extraction Threshold

**HARD STOP — do NOT proceed to Phase 2 until:**

- [ ] Design Spec Table has **≥10 distinct elements** (not 10 rows — 10 different elements)
- [ ] At least **5 colors** identified (text, background, border, accent, secondary)
- [ ] At least **3 typography styles** captured (heading, body, label/caption)
- [ ] At least **2 spacing values** documented (padding, gap)
- [ ] At least **1 component** fully specified (all properties: radius, padding, colors, typography)

If `get_design_context` returns insufficient data, supplement by:
1. Calling `get_metadata` for dimensions/positions
2. Taking `get_screenshot` and visually identifying elements the API missed
3. Asking the user to confirm values you can't extract programmatically

> **NEVER guess or interpolate design values.** If a value isn't extractable, flag it and ask the user.

---

## Phase 2: Browser Measurement

### Step 2A: Navigate and screenshot

```
1. Navigate to target URL
2. Take screenshot → compare visually against Figma screenshot from Phase 1
3. Note obvious layout discrepancies before measuring
```

### Step 2B: Measure with JS (via computer-use)

For each element in the Design Spec Table, measure the corresponding browser element:

```javascript
const rgbToHex = (v) => {
  if (!v || v === 'transparent' || !v.startsWith('rgb')) return v;
  const m = v.match(/[\d.]+/g);
  if (!m || m.length < 3) return v;
  const hex = '#' + m.slice(0,3).map(x => Math.round(+x).toString(16).padStart(2,'0')).join('');
  const a = m.length >= 4 ? parseFloat(m[3]) : 1;
  return a < 1 ? hex + ` (opacity: ${a})` : hex;
};

const measureAll = (sel) => {
  const el = document.querySelector(sel);
  if (!el) return { error: 'NOT FOUND', selector: sel };
  const s = getComputedStyle(el);
  return {
    selector: sel,
    text: el.textContent?.trim().slice(0, 40),
    // Typography
    fontFamily: s.fontFamily.split(',')[0].trim().replace(/['"]/g, ''),
    fontSize: s.fontSize,
    fontWeight: s.fontWeight,
    lineHeight: s.lineHeight,
    letterSpacing: s.letterSpacing,
    textTransform: s.textTransform,
    textDecoration: s.textDecoration,
    textAlign: s.textAlign,
    // Colors
    color: rgbToHex(s.color),
    backgroundColor: rgbToHex(s.backgroundColor),
    borderColor: rgbToHex(s.borderColor),
    opacity: s.opacity,
    // Spacing
    padding: `${s.paddingTop} ${s.paddingRight} ${s.paddingBottom} ${s.paddingLeft}`,
    margin: `${s.marginTop} ${s.marginRight} ${s.marginBottom} ${s.marginLeft}`,
    gap: s.gap,
    width: s.width, height: s.height,
    // Layout
    display: s.display, flexDirection: s.flexDirection,
    alignItems: s.alignItems, justifyContent: s.justifyContent,
    // Borders
    borderWidth: s.borderWidth, borderStyle: s.borderStyle,
    borderRadius: s.borderRadius,
    // Effects
    boxShadow: s.boxShadow,
    overflow: s.overflow
  };
};

JSON.stringify([
  measureAll('SELECTOR_1'),
  measureAll('SELECTOR_2'),
  // ... batch 5-10 per call
]);
```

**Selector strategy (prefer stable selectors):**
1. `[data-testid="x"]` — best
2. `[role="x"]`, semantic tags (`h1`, `nav`, `button`)
3. Stable BEM classes (`.card__header`)
4. Structural (`.container > :nth-child(2)`)
5. Never use hashed classes (`.css-1abc`, `.sc-xyz`)

### Step 2C: Handle interactive states

For hover/focus/active states referenced in the Figma design:
```
1. Trigger state (focus via JS, hover via computer-use)
2. Immediately measure with getComputedStyle
3. Record as separate row: "Button:hover | background-color | ..."
```

### Guardrail: Measurement Coverage

**HARD STOP — do NOT proceed to Phase 3 until:**

- [ ] Every element in the Design Spec Table has a corresponding browser measurement
- [ ] Elements marked `NOT FOUND` have been retried with alternate selectors
- [ ] At least **15 distinct elements** measured (if page has fewer, measure all)
- [ ] All interactive states shown in Figma (hover, focus, active, disabled) have been triggered and measured
- [ ] Dropdowns, modals, tooltips visible in the Figma design have been opened and measured

> **If an element cannot be found in the DOM**, record it as `MISSING` with severity `critical` — do not silently skip it.

---

## Phase 3: Diff Generation

Compare Design Spec Table against Browser Measurements. Produce the **Diff Table**.

### Rules

1. **Only report actual differences.** If expected === actual → skip.
2. **Normalize values before comparing:**
   - Colors: both to lowercase hex (`#ffffff` = `#FFFFFF`)
   - Spacing: round to nearest px (14.4px ≈ 14px is NOT a bug, 14px vs 16px IS)
   - Weights: numeric (400 = normal, 700 = bold)
   - line-height: if design says `1.5` and browser says `24px` on 16px text → equivalent, skip
   - `0px` = `0` = `none` for border-radius, margin, padding
3. **Tolerance:** ±1px for spacing, ±1 for RGB channels. Anything within tolerance = skip.
4. **Severity classification:**

| Severity | Criteria |
|----------|----------|
| critical | Layout broken, content hidden/clipped, wrong element entirely |
| high     | Wrong color on primary element, wrong font-size on headings, wrong padding causing visual imbalance |
| medium   | Wrong weight, wrong letter-spacing, wrong border-radius, secondary color off |
| low      | 2px spacing diff, minor opacity difference, subtle shadow difference |

### Output: The Diff Table

```markdown
## Pixel-Perfect Diff

**Source:** [Figma URL]
**Target:** [Target URL]
**Date:** YYYY-MM-DD
**Total discrepancies:** N

| # | Element | Property | Expected (Figma) | Actual (Browser) | Severity | Selector |
|---|---------|----------|------------------|------------------|----------|----------|
| 1 | Page title | font-size | 32px | 28px | high | h1.page-title |
| 2 | Page title | font-weight | 700 | 600 | medium | h1.page-title |
| 3 | Card | border-radius | 12px | 8px | medium | .card |
| 4 | Card | padding | 24px | 16px | high | .card |
| 5 | Card | box-shadow | 0 2px 8px rgba(0,0,0,0.1) | none | high | .card |
| 6 | Button | height | 40px | 36px | medium | .btn-primary |
| 7 | Nav items | gap | 24px | 16px | medium | nav > ul |
| 8 | Badge | background-color | #EEF2FF | #E5E7EB | low | .badge |

### Systemic Issues

| Pattern | Affected elements | Fix |
|---------|-------------------|-----|
| All border-radius 12→8 | Card, Modal, Dropdown | Global token `--radius-lg` is `8px`, should be `12px` |
| Heading weights 700→600 | H1, H2, Card title | Font weight token or CSS variable |
```

---

## Phase 4: Verification Pass

Before finalizing, verify the diff:

1. **Screenshot comparison** — take side-by-side screenshots, confirm reported issues are visually apparent
2. **No false positives** — for each "high" or "critical" item, re-measure to confirm
3. **Systemic deduplication** — if 10 elements have the same border-radius diff, report once as systemic + list affected elements
4. **Check for missing elements** — elements in Figma but absent from DOM (severity: critical)

### False Positive Prevention (MUST follow)

| # | Rule | What to do |
|---|------|------------|
| 1 | **Value exists in design but you missed it** | Before reporting "off-spec", go back to Figma and verify the specific element. Your extraction may be incomplete. |
| 2 | **Equivalent values** | `line-height: 1.5` = `24px` on 16px text. `700` = `bold`. `#fff` = `#ffffff`. Do NOT report these. |
| 3 | **Browser defaults on unspecified properties** | If Figma doesn't explicitly set a value, the browser default is not a bug. |
| 4 | **Design intent: different roles = different styles** | Admin badge blue, member badge green is NOT inconsistency — it's by design. |
| 5 | **Imperceptible color diff** | RGB channels each ≤3 apart → dismiss. |
| 6 | **Computed vs authored values** | `getComputedStyle` returns resolved px even if authored in rem/em. Convert before comparing. |

### Cross-Verification Loop

For any value flagged as a discrepancy:

```
1. Is this value actually specified in the Figma design? → Re-check get_design_context
2. Could this be a different variant/state? → Check Figma component variants
3. Is the browser element the CORRECT match for the Figma element? → Verify by text content + position
4. Still a real diff after all checks? → Keep in the table
```

> **GOLDEN RULE: Never report a diff without verifying BOTH the expected value (from Figma) and the actual value (re-measured) are correct.**

### Guardrail: Diff Table Quality

**HARD STOP — do NOT deliver until:**

- [ ] Zero rows where Expected === Actual (after normalization)
- [ ] Every row has a working CSS selector (not a guess)
- [ ] No duplicate rows (same element + same property)
- [ ] Systemic patterns extracted (3+ elements with same diff → systemic issue)
- [ ] All `critical` and `high` items have been re-measured to confirm

---

## Phase 5: Handoff Format

Produce **both** artifacts:

1. The markdown Diff Table (Phase 3 format) — human-readable reference
2. A JSON blob — machine-readable handoff for the implementer agent loop

Save markdown to: `{project}/pixel-perfect-diff.md`  
Save JSON to: `{project}/pixel-perfect-issues.json`

### JSON Schema

```json
{
  "source_figma": "https://figma.com/...",
  "target_url": "https://...",
  "total": 8,
  "issues": [
    {
      "id": 1,
      "severity": "high",
      "element": "Page title",
      "property": "font-size",
      "expected": "32px",
      "actual": "28px",
      "selector": "h1.page-title",
      "fix_instruction": "Set font-size to 32px on h1.page-title",
      "systemic_group": null
    },
    {
      "id": 3,
      "severity": "medium",
      "element": "Card",
      "property": "border-radius",
      "expected": "12px",
      "actual": "8px",
      "selector": ".card",
      "fix_instruction": "Update --radius-lg token from 8px to 12px (affects Card, Modal, Dropdown)",
      "systemic_group": "radius-lg-mismatch"
    }
  ],
  "systemic_groups": {
    "radius-lg-mismatch": {
      "description": "--radius-lg token is 8px, should be 12px",
      "affected_selectors": [".card", ".modal", ".dropdown"],
      "fix_instruction": "Change --radius-lg CSS variable to 12px in the design token file"
    }
  }
}
```

**Rules for JSON output:**

- `severity`: one of `critical | high | medium | low`
- `systemic_group`: `null` if standalone; a slug string if part of a systemic pattern
- Issues in a `systemic_group` share the same `fix_instruction` pointing to the token/variable fix
- Sort `issues` array by severity: critical first, low last
- `fix_instruction` must be a complete, actionable sentence an agent can execute without other context

### For the implementation agent

Include this preamble in the markdown output:

```markdown
> **Instructions for implementation agent:**
> Use pixel-perfect-issues.json. Each issue is one CSS fix.
> Apply `expected` value to the element at `selector`.
> For issues with a `systemic_group`, apply the group-level fix once instead of per-element.
> After applying all fixes, re-run this verification to confirm zero remaining diffs.
```

---

## Critical Rules

### MUST

- Extract design specs BEFORE measuring the browser — never measure first and guess the expected value
- Design Spec Table must have ≥10 elements and ≥5 colors before proceeding
- For every "not in design" finding → go back to Figma and verify that specific element
- Capture text content with every measurement (confirms you're measuring the right element)
- Normalize values before comparing (px/rem, hex case, weight names vs numbers)
- Re-measure every `critical` and `high` finding before including in final diff
- Open every dropdown, modal, tooltip visible in the Figma design
- Report MISSING elements as `critical` — don't silently skip
- Include a working CSS selector for every row

### NEVER

- Never report a diff where Expected === Actual (after normalization)
- Never guess or interpolate design values — if you can't extract it, ask the user
- Never start measuring before the Design Spec Table passes the minimum threshold
- Never flag different categories/roles having different styles as "inconsistency"
- Never use hashed/generated class names as selectors (they break on rebuild)
- Never skip the cross-verification loop — every diff gets verified both sides
- Never deliver a table with duplicate rows (same element + same property)
- Never treat browser defaults on unspecified properties as bugs

---

## Examples

See [EXAMPLES.md](EXAMPLES.md) for full worked examples showing:
- Figma extraction → diff table generation
- Handling design tokens vs hardcoded values
- Systemic issue detection patterns

---

## Evals

`evals/evals.json` has 3 self-contained regression cases (schema per the `skill-creator`
skill's `evals.json` format). Each pairs a pre-extracted Design Spec Table
(`evals/files/<case>/design-spec.md`, standing in for a completed Phase 1 Figma
extraction) with a static `target.html` fixture that has known, deliberate deviations —
so the eval is deterministic and needs no live Figma/network access, only a browser
that can open a `file://` URL.

- **`profile-card-mixed-deviations`** — the core case: high/medium/low/critical severity
  classification, a missing element, a `:hover` state check, and two systemic patterns
  (shared radius-token drift, identical drift across 3 repeated tag chips), alongside
  several exact-match properties that must NOT be reported.
- **`exact-match-anti-hallucination`** — a build that matches the design exactly but is
  authored differently (ratio vs resolved line-height, named vs numeric weight, sub-pixel
  rem rounding, hex case). The correct output is **zero** reported discrepancies — this
  is the guardrail against inventing diffs.
- **`settings-list-systemic-token-drift`** — one spacing token wrong across 5 repeated
  rows, testing that the dedup rule collapses it into a single systemic issue instead of
  5 duplicate rows.

Run these with the `skill-creator` skill's eval workflow (with-skill vs. baseline
subagents, grade against each eval's `expectations`, aggregate into `benchmark.json`).
All numeric/color claims in `evals/files/**` were verified with `getComputedStyle`
against real Chromium — treat any fixture edit as needing the same re-verification.
