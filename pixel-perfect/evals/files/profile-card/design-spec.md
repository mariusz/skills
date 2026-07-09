# Design Spec — Profile Card (extracted from Figma)

> Phase 0/1 already complete. Treat this table as the verified output of
> `get_design_context` + `get_metadata` + `get_screenshot` for node
> `figma.com/design/PC001/Profile-Card?node-id=4-88`. Do not re-fetch Figma —
> start at Phase 2 (browser measurement) using the target file provided.

| Element (path)                          | Property         | Design Value                  |
|------------------------------------------|------------------|--------------------------------|
| Profile Card                              | padding          | 24px                          |
| Profile Card                              | border-radius    | 20px                          |
| Profile Card                              | background-color | #FFFFFF                        |
| Profile Card                              | border           | 1px solid #E5E7EB              |
| Profile Card                              | box-shadow       | 0 4px 12px rgba(0,0,0,0.08)   |
| Profile Card > Avatar                     | width            | 64px                          |
| Profile Card > Avatar                     | height           | 64px                          |
| Profile Card > Avatar                     | border-radius    | 9999px                        |
| Profile Card > Name                       | font-size        | 20px                          |
| Profile Card > Name                       | font-weight      | 700                            |
| Profile Card > Name                       | color            | #111827                        |
| Profile Card > Name                       | line-height      | 28px                           |
| Profile Card > Verified Badge (icon)      | presence         | exists, 16x16, next to Name    |
| Profile Card > Role                       | font-size        | 14px                           |
| Profile Card > Role                       | font-weight      | 500                            |
| Profile Card > Role                       | color            | #6B7280                        |
| Profile Card > Bio                        | font-size        | 14px                           |
| Profile Card > Bio                        | font-weight      | 400                            |
| Profile Card > Bio                        | color            | #374151                        |
| Profile Card > Bio                        | line-height      | 1.5                            |
| Profile Card > Stats                      | display          | flex                           |
| Profile Card > Stats                      | gap              | 32px                           |
| Stat (Followers) > Value                  | font-size        | 18px                           |
| Stat (Followers) > Value                  | font-weight      | 700                            |
| Stat (Followers) > Value                  | color            | #111827                        |
| Stat (Followers) > Label                  | font-size        | 12px                           |
| Stat (Followers) > Label                  | font-weight      | 500                            |
| Stat (Followers) > Label                  | color            | #9CA3AF                        |
| Stat (Followers) > Label                  | text-transform   | uppercase                      |
| Stat (Followers) > Label                  | letter-spacing   | 0.05em                         |
| Stat (Following) > Label                  | text-transform   | uppercase                      |
| Stat (Following) > Label                  | letter-spacing   | 0.05em                         |
| Tag chip (Design / Product / Remote)      | padding          | 4px 12px                       |
| Tag chip                                  | border-radius    | 9999px                         |
| Tag chip                                  | background-color | #EEF2FF                        |
| Tag chip                                  | color            | #4338CA                        |
| Tag chip                                  | font-size        | 12px                           |
| Tag chip                                  | font-weight      | 500                            |
| Follow Button                             | height           | 44px                           |
| Follow Button                             | padding-inline   | 20px                           |
| Follow Button                             | border-radius    | 12px                           |
| Follow Button                             | background-color | #4F46E5                        |
| Follow Button                             | color            | #FFFFFF                        |
| Follow Button                             | font-weight      | 600                             |
| Follow Button                             | font-size        | 14px                            |
| Follow Button:hover                       | background-color | #4338CA                        |
| Message Button                            | height           | 44px                           |
| Message Button                            | padding-inline   | 20px                           |
| Message Button                            | border-radius    | 12px                           |
| Message Button                            | border           | 1px solid #E5E7EB              |
| Message Button                            | background-color | transparent                    |
| Message Button                            | color            | #111827                        |
| Message Button                            | font-weight      | 600                             |
