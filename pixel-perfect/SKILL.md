---
name: pixel-perfect
description: Pixel-perfect design QA for a rendered page or component against a Figma frame or supplied spec. Measures visual differences and produces a verified diff and JSON handoff.
---

# Pixel-perfect verification

Compare a design with a rendered implementation at the same viewport and state. Report discrepancies that have a verified design value and a verified browser measurement. Use the user's stated scope; a small component is a complete audit when every applicable element in that component is covered.

## Phase 0: Establish the comparison

1. Identify the design source (Figma frame or supplied spec), running target URL or local HTML fixture, scope, viewport, state, and known intentional exceptions. Use available context before asking for missing inputs.
2. Match the design frame's dimensions to the browser viewport. Record any device scale, zoom, responsive variant, and content or data differences that affect comparison.
3. If the design or rendered target is inaccessible, report the exact blocker and any work already verified. Continue with accessible portions only when they support a real comparison.

**Done when:** the design and rendered target refer to the same component or page, viewport, content, and state, or the remaining mismatch is explicitly recorded as a limitation.

## Phase 1: Build the design spec

Use a supplied, verified design-spec table directly. For Figma, prefer structured design data through an available Figma tool. A configured REST API token is another route; use the frame's file key and node ID to request its node tree. Inspect the frame visually through a screenshot or equivalent view when available.

For each visible element in scope, record its path, state, property, value and unit, and evidence source. Cover every **specified, comparable** property that matters to the appearance:

- Text: family, size, weight, line height, letter spacing, alignment, case, decoration, wrapping.
- Paint: text and fill colors, gradients, opacity, borders, radii, shadows, effects.
- Geometry: frame dimensions, position, padding, spacing, sizing behavior, alignment, overflow.
- Assets: icon or image identity, dimensions, crop, and focal position.
- Interaction: variants or states shown in the design.

Keep authored values distinct from visual inferences. Figma's layout, fills, and effects do not always map to one CSS declaration; compare the resulting geometry or appearance when that is the meaningful contract. Mark an unavailable design value **unverified** and omit a numeric diff for it. A screenshot may establish presence, alignment, or asset identity without establishing an exact CSS value.

| Element path and state | Property | Design value | Evidence |
|---|---|---|---|
| Header > Title | font-size | 32px | Figma text style |
| Card | padding | 24px | Figma layout |
| Card > Badge | presence | visible | Figma screenshot |

**Done when:** every visible element in scope is inventoried, its applicable specified properties are recorded, and uncertain values are marked unverified. Coverage follows the requested scope, including small components.

## Phase 2: Measure the rendered target

1. Capture the target at the matched viewport and state. Compare its overall composition with the design view to locate omissions and wrong matches.
2. Match each design element to its rendered counterpart using content, role, and position. Record a stable selector for the element; prefer an existing `data-testid`, semantic selector, or stable class. For a missing element, record a stable selector for the expected parent and identify the missing child in the element path.
3. Read computed styles and geometry from the rendered DOM, including bounding boxes where size or position matters. Measure visible assets and text wrapping visually as needed. Batch measurements when it helps, but check that each selector matches the intended element.
4. Trigger and measure each state represented in the design, including hover, focus, active, disabled, or open overlays. A state that cannot be reached is a coverage limitation.

Measure design-system components like any other element. Computed CSS can differ from authored CSS through inheritance, resets, and component defaults. Record the actual text or other identifying content alongside measurements to catch wrong element matches.

**Done when:** each design element and specified property has a matching measurement, a confirmed missing-element finding, or an explicit coverage limitation. Retry selectors that fail before calling an element missing.

## Phase 3: Compare and classify

Compare only properties supported by design evidence. Normalize equivalent representations before diffing: color notation and case, named and numeric weights, zero values, and unitless line height resolved against font size. Compare individual sides of shorthand spacing and borders. Account for opacity and alpha. Use ±1px for geometry or spacing and ±1 RGB channel for color as a default tolerance; state a different tolerance when the design or rendering context requires it. Preserve a real, visible difference even when a numeric tolerance would hide it.

Classify by impact in the audited context:

| Severity | Meaning |
|---|---|
| critical | Missing or obscured content, broken layout, or wrong element |
| high | Prominent typography, color, spacing, or asset mismatch |
| medium | Clear local mismatch with limited page impact |
| low | Subtle visual mismatch worth fixing |

Group repeated discrepancies only after confirming a shared cause. If several elements use the same wrong token or rule, keep one table row for the pattern and list every affected selector in the systemic section and JSON group. A repeated value alone does not prove a shared token; inspect source before naming the fix. Keep separate rows when causes differ.

**Done when:** every candidate row has distinct expected and actual values after normalization, evidence on both sides, an impact-based severity, and a verified selector or parent selector.

## Phase 4: Verify findings

Recheck each finding against the exact Figma node or supplied spec row and the matched browser element. Re-measure all critical and high findings. Review the two views together for missing elements, wrong variants, content differences, assets, and layout context. Remove false positives caused by equivalent values, unspecified design properties, wrong elements, or intentional exceptions. Record unresolved coverage separately from discrepancies.

**Done when:** each reported row survives both source and target checks, repeated rows are consolidated by confirmed cause, and the limitations list names every part of the requested scope that could not be verified.

## Phase 5: Deliver the handoff

Write `pixel-perfect-diff.md` and `pixel-perfect-issues.json` in the target project or the user-specified output directory. A zero-diff audit still produces both files with `total: 0` and an empty `issues` array. Include the source, target, viewport/state, date, intentional exceptions, coverage limitations, and total. Sort findings critical to low. Show only discrepancies in the main table. Describe `total: 0` as a verified match only when the rendered target and every requested state were measured; otherwise identify the unverified scope.

```markdown
## Pixel-perfect diff

| # | Element | Property | Expected | Actual | Severity | Selector |
|---|---|---|---|---|---|---|
| 1 | Card | padding | 24px | 20px | high | [data-testid="card"] |

### Systemic issues

| Pattern | Affected selectors | Confirmed cause | Fix |
|---|---|---|---|

### Coverage limitations

- None.
```

Use this JSON shape. `total` counts rows in `issues`, including one row for each consolidated systemic pattern. Set `verified_match` to `true` only when `total` is zero and the full rendered scope has been measured without coverage limitations. For a missing element, `selector` identifies its existing parent and `element` identifies the absent child. An inferred fix must be labeled as a hypothesis in both artifacts; a confirmed fix names the actual rule or token.

```json
{
  "source_figma": "https://figma.com/design/...",
  "target_url": "https://example.com/page",
  "total": 1,
  "verified_match": false,
  "coverage_limitations": [],
  "issues": [
    {
      "id": 1,
      "severity": "high",
      "element": "Card",
      "property": "padding",
      "expected": "24px",
      "actual": "20px",
      "selector": "[data-testid=\"card\"]",
      "fix_instruction": "Set padding to 24px for [data-testid=\"card\"].",
      "systemic_group": null
    }
  ],
  "systemic_groups": {}
}
```

For a supplied design spec, put its path in `source_figma`; for a local fixture, put its path in `target_url`. A systemic group includes `description`, `affected_selectors`, and one `fix_instruction`; its issue references the group's slug. Validate the JSON, ensure `total` matches the issue count, and ensure every group reference resolves.

**Done when:** both artifacts agree, every finding is actionable and verified, and uncovered scope is explicit. See [EXAMPLES.md](EXAMPLES.md) for worked output and `evals/evals.json` for regression cases.

## Repair loop when fixes are in scope

When the user asks to bring the implementation in line with the design, or an implementation agent consumes the handoff, use the verified diff as a worklist:

1. Apply the confirmed fixes to the implementation. Check affected components before changing a shared token or rule.
2. Rebuild or reload the target at the same viewport, content, and states. Re-measure the full requested scope using Phases 2–4; a changed component can introduce new differences elsewhere.
3. Update both artifacts with the current findings and repeat the fix and verification pass until the measured discrepancy count is zero.

Stop with the remaining verified differences and the exact blocker if a fix is outside the authorized scope, the target cannot be measured, or another pass makes no progress. A zero count with coverage limitations is an incomplete audit, not a verified match. For an audit-only request, include these loop instructions in the handoff without editing the implementation.

**Done when:** the full requested scope has zero verified differences and no coverage limitations, or the handoff names every remaining difference and the reason the loop stopped.
