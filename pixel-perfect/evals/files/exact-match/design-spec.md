# Design Spec — Alert Banner (extracted from Figma)

> Phase 0/1 already complete. Treat this table as the verified output of
> `get_design_context` for node `figma.com/design/AB001/Alert-Banner?node-id=2-14`.
> Do not re-fetch Figma — start at Phase 2 (browser measurement) using the target file provided.
>
> This design is a small, deliberately tricky component: the live build matches the
> design intent exactly, but several values are authored differently between Figma
> (ratios, named weights) and the browser (resolved px, numeric weights). A correct
> pixel-perfect pass reports **zero discrepancies**.

| Element (path)         | Property         | Design Value              |
|--------------------------|------------------|----------------------------|
| Alert                    | display          | flex                       |
| Alert                    | align-items      | center                     |
| Alert                    | gap              | 12px                       |
| Alert                    | padding          | 16px                       |
| Alert                    | border-radius    | 8px                        |
| Alert                    | background-color | #FEF2F2                    |
| Alert                    | border           | 1px solid #FCA5A5          |
| Alert > Icon             | width            | 20px                       |
| Alert > Icon             | height           | 20px                       |
| Alert > Title            | font-size        | 14px                       |
| Alert > Title            | font-weight      | Bold                       |
| Alert > Title            | color            | #991B1B                    |
| Alert > Title            | line-height      | 1.4                        |
| Alert > Message          | font-size        | 14px                       |
| Alert > Message          | font-weight      | 400                        |
| Alert > Message          | color            | #7F1D1D                    |
| Alert > Message          | line-height      | 1.5                        |
| Dismiss Button           | width            | 24px                       |
| Dismiss Button           | height           | 24px                       |
| Dismiss Button           | border-radius    | 9999px                     |
