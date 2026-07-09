# Design Spec — Settings List (extracted from Figma)

> Phase 0/1 already complete. Treat this table as the verified output of
> `get_design_context` for node `figma.com/design/SL001/Settings-List?node-id=9-40`.
> Do not re-fetch Figma — start at Phase 2 (browser measurement) using the target file provided.

| Element (path)                | Property         | Design Value        |
|---------------------------------|------------------|----------------------|
| Settings List > Heading         | font-size        | 18px                 |
| Settings List > Heading         | font-weight      | 700                   |
| Settings List > Heading         | color            | #111827               |
| Settings Row (all 5)            | padding          | 16px 20px            |
| Settings Row (all 5)            | display          | flex                 |
| Settings Row (all 5)            | align-items      | center               |
| Settings Row (all 5)            | gap              | 12px                 |
| Settings Row (all 5)            | border-bottom    | 1px solid #E5E7EB    |
| Settings Row > Icon             | width            | 20px                 |
| Settings Row > Icon             | height           | 20px                 |
| Settings Row > Label            | font-size        | 14px                 |
| Settings Row > Label            | font-weight      | 500                   |
| Settings Row > Label            | color            | #111827               |
| Settings Row > Chevron          | width            | 16px                 |
| Settings Row > Chevron          | height           | 16px                  |
| Settings Row > Chevron          | color            | #9CA3AF               |

**Rows (5):** Account, Notifications, Privacy, Billing, Language — all share the exact
same padding, layout, and border-bottom per the design system's list-row pattern.
