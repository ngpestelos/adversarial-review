# Adversarial Review: Findings Format

The output contract for `adversarial-review`. Every field exists to make a silent failure loud.

## Structure

```
VERDICT: <one of: PROCEED | PROCEED WITH CHANGES | REWORK>
Lenses applied: <e.g. "1-10" or "1-9; 10 skipped: no security surface">

Gets right: <one line>

Load-bearing assumptions:
  - <assumption> → holds? [yes/no/unverified]
  - ...

Findings (severity-ranked):
  [BLOCKER] <one line> — scenario: <concrete inputs/state → wrong outcome> — <file:line or source>
  [MAJOR]   <one line> — scenario: ... — <source>
  [MINOR]   <one line> — scenario: ... — <source>
  [NIT]     <one line>

Accepted (won't fix):
  - <finding> — accepted by user on <pass N>

Not inspected: <paths or sections, or "none">
```

## Field rules

| Field | Rule |
|---|---|
| `VERDICT` | Whole token. Consumers match the full value, never a substring. `PROCEED` is a prefix of `PROCEED WITH CHANGES`. |
| `Lenses applied` | Lists every lens. A skipped lens names a structural reason. Silent skips are not allowed. |
| `Gets right` | One line. A pure attack piece loses calibration. |
| `Load-bearing assumptions` | What the target stands on. Each marked holds, fails, or unverified. |
| Finding line | One sentence, a concrete failure scenario, a cited source. No scenario, no finding. |
| `Accepted (won't fix)` | Populated only by the user. Carried between passes. Accepted items do not re-report. |
| `Not inspected` | Required. A partial review never issues a clean verdict. |

## Severity

| Level | Meaning |
|---|---|
| BLOCKER | Do not commit. |
| MAJOR | Fix, or the user accepts it explicitly. |
| MINOR | Fix soon. |
| NIT | Optional. |

## Verdict mapping

First match wins:

1. Any BLOCKER → `REWORK`
2. One or more MAJOR not in `Accepted` → `PROCEED WITH CHANGES`
3. Otherwise → `PROCEED`

## Loop

The model reads `VERDICT:` and decides; the harness does not parse it. Review, fix or accept, re-run. Stop at `PROCEED`. The same BLOCKER or MAJOR surviving three passes un-accepted stops the loop and escalates.

## Malformed output

A reviewer that returns prose instead of this structure leaves its lenses UNREVIEWED. Do not synthesize around the gap. Re-run the reviewer or flag `[cluster N unreviewed]` in the synthesized report. Never `PROCEED` over an unreviewed cluster.
