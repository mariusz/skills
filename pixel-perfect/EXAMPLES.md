# Pixel-perfect handoff examples

These examples show the output decisions that need care. They are illustrative; use measured values and selectors from the current audit.

## Equivalent values produce no issue

| Property | Design | Browser | Result |
|---|---|---|---|
| font-weight | Bold | 700 | Equivalent |
| line-height on 14px text | 1.5 | 21px | Equivalent |
| color | #FFFFFF | rgb(255, 255, 255) | Equivalent |
| width | 20px | 20.4px | Within default tolerance |

If rendered measurements confirm these are the only differences, deliver an empty diff table and JSON with `"total": 0`, `"verified_match": true`, `"coverage_limitations": []`, `"issues": []`, and `"systemic_groups": {}`. If only source inspection is available, set `verified_match` to `false` and name the browser coverage limit.

## A missing element

The design shows a verified badge next to the profile name. The rendered profile header exists, but the badge is absent after checking the expected variant and state.

| # | Element | Property | Expected | Actual | Severity | Selector |
|---|---|---|---|---|---|---|
| 1 | Profile header > Verified badge | presence | visible | missing | critical | [data-testid="profile-header"] |

The selector identifies the existing parent. The element path identifies the missing child. The fix instruction should identify the asset and placement from the design evidence.

## A systemic issue

Five settings rows all have `12px 16px` padding where the design specifies `16px 20px`. Source inspection confirms they share the same list-row rule.

| # | Element | Property | Expected | Actual | Severity | Selector |
|---|---|---|---|---|---|---|
| 1 | Settings rows (5) | padding | 16px 20px | 12px 16px | medium | .settings-row |

| Pattern | Affected selectors | Confirmed cause | Fix |
|---|---|---|---|
| Settings row padding | [data-testid="settings-row-account"], [data-testid="settings-row-notifications"], [data-testid="settings-row-privacy"], [data-testid="settings-row-billing"], [data-testid="settings-row-language"] | Shared list-row rule | Set its padding to 16px 20px |

```json
{
  "source_figma": "design-spec.md",
  "target_url": "target.html",
  "total": 1,
  "verified_match": false,
  "coverage_limitations": [],
  "issues": [
    {
      "id": 1,
      "severity": "medium",
      "element": "Settings rows (5)",
      "property": "padding",
      "expected": "16px 20px",
      "actual": "12px 16px",
      "selector": ".settings-row",
      "fix_instruction": "Set the shared list-row rule's padding to 16px 20px.",
      "systemic_group": "settings-row-padding"
    }
  ],
  "systemic_groups": {
    "settings-row-padding": {
      "description": "All five settings rows use the same undersized padding.",
      "affected_selectors": [
        "[data-testid=\"settings-row-account\"]",
        "[data-testid=\"settings-row-notifications\"]",
        "[data-testid=\"settings-row-privacy\"]",
        "[data-testid=\"settings-row-billing\"]",
        "[data-testid=\"settings-row-language\"]"
      ],
      "fix_instruction": "Set the shared list-row rule's padding to 16px 20px."
    }
  }
}
```

If the shared rule cannot be inspected, describe the common measured pattern and mark the proposed rule-level fix as a hypothesis.

## Responsive variants and states

Audit each supplied design viewport and state against the matching rendered viewport and state. A desktop default button and a tablet hover button are separate comparisons. Record the viewport and state in the report header; put the state in the element path, such as `Follow button:hover`. If the target cannot reach that state, list it under coverage limitations rather than inventing an actual value.
