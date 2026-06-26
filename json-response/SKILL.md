---
name: json-response
description: >
  Emit a structured JSON blob as the agent's final output, ready to be consumed by an implementer agent in a loop.
  Use when: another skill (e.g. pixel-perfect) has produced a list of findings/issues and the next step is
  handing those findings to an implementer agent that processes them one by one.
  TRIGGER when: the user asks for "JSON output", "JSON blob", "push to implementer", "loop over issues",
  or any skill's handoff section says "save JSON to pixel-perfect-issues.json".
  DO NOT TRIGGER when: the user wants a general JSON formatter, schema validator, or data conversion.
---

# JSON Response

Serialize findings from the current skill run into a well-formed JSON blob and write it to disk.

---

## When to use

Invoke this skill at the end of any verification/audit skill (e.g. pixel-perfect) when the output
needs to feed an implementer agent loop rather than (or in addition to) a human-readable report.

---

## Output Rules

1. **Write JSON to the path specified by the calling skill** (e.g. `pixel-perfect-issues.json`).
   Default path if none specified: `{project-root}/issues.json`
2. **Always produce valid JSON** — no trailing commas, no comments.
3. **Sort issues by severity**: `critical → high → medium → low`.
4. **Every issue must be self-contained** — the implementer needs no other context to act on it.

---

## Schema

```json
{
  "source": "<audit source URL or file>",
  "target": "<implementation URL or path>",
  "total": 0,
  "issues": [
    {
      "id": 1,
      "severity": "critical | high | medium | low",
      "element": "<human-readable element name>",
      "property": "<CSS property or attribute name>",
      "expected": "<value from design/spec>",
      "actual": "<value in implementation>",
      "selector": "<CSS selector or file:line reference>",
      "fix_instruction": "<complete actionable sentence>",
      "systemic_group": "<slug | null>"
    }
  ],
  "systemic_groups": {
    "<slug>": {
      "description": "<what the systemic pattern is>",
      "affected_selectors": ["<sel1>", "<sel2>"],
      "fix_instruction": "<token/variable-level fix>"
    }
  }
}
```

### Field rules

| Field | Required | Notes |
|-------|----------|-------|
| `id` | yes | 1-indexed, sequential |
| `severity` | yes | `critical`, `high`, `medium`, or `low` only |
| `element` | yes | Human-readable (e.g. "Card header") |
| `property` | yes | CSS property or attribute name |
| `expected` | yes | Exact value from spec/design |
| `actual` | yes | Exact measured value |
| `selector` | yes | Stable CSS selector; never hashed class names |
| `fix_instruction` | yes | Full sentence; must be actionable without other context |
| `systemic_group` | yes | `null` if standalone; slug string if systemic |

---

## Systemic Groups

When 3+ issues share the same root cause (e.g. a wrong CSS variable), group them:

- Each issue in the group still gets its own entry in `issues[]`
- All share the same `systemic_group` slug
- The group entry in `systemic_groups` carries the one-time fix
- The implementer applies the group fix once, skips individual fixes for grouped issues

---

## Implementer Agent Loop Contract

The JSON is consumed like this:

```
for each issue in issues[]:
  if issue.systemic_group:
    apply systemic_groups[issue.systemic_group].fix_instruction (once per group)
  else:
    apply issue.fix_instruction to issue.selector
```

`fix_instruction` must therefore be:
- **Imperative** — starts with a verb ("Set", "Change", "Update", "Remove")
- **Complete** — includes the target selector/file and exact value to use
- **Idempotent** — applying it twice causes no harm
