---
name: adversarial-review
description: "Adversarial stress-test of a plan, design, document, or code change before you commit to it. Read-only: it critiques, never edits. Runs inline for a quick pass or fans out parallel read-only reviewers for high-stakes or self-authored targets. Emits a verdict (PROCEED / PROCEED WITH CHANGES / REWORK) that drives a loop until clean. Trigger phrases: adversarial review, red-team this, stress-test, poke holes, what's wrong with this, before I commit."
---

# Adversarial Review

**Read-only.** Findings only. Never edit the target.

## Invocation

- `/adversarial-review` reviews the artifact under discussion.
- `/adversarial-review <path-or-content>` reviews a named target.

Resolve the target first. No target: ask, then stop. Do not invent one. One target: proceed. Several targets, interactive: ask once. Several targets, unattended: pick the most recent or salient, state the assumption, proceed.

## Step 0: Independence check

Self-review is the weakest mode. You cannot reliably attack a target you just authored. Classify before reviewing:

| Signal | Route |
|---|---|
| Low stakes, reversible, not self-authored | Inline single pass (Steps 1 to 3). |
| High stakes, hard to reverse, drafted this session, or self-strategy | Parallel Mode: spawn read-only reviewers so the critique is not the author's. |

State the route and why, in one line. The user can override.

If you cannot spawn sub-agents, do not fall silently to the weakest mode. Run the lens clusters as sequential independent passes with a fresh adversary each time. Do not reuse prior conclusions. Flag: `[reduced independence: sequential self-review, no parallel reviewers]`.

## Step 1: Verify, do not speculate

Every finding rests on a checked fact.

- Code: open the file. Cite `file:line`. Never assert a line exists without reading it. A finding built on an imagined line is itself the bug.
- Non-code: cite the source of truth (linked doc, config, log, primary source). Unconfirmed claims carry `[Inference]` or `[Unverified]`.

Verify the facts your critique depends on, not only the target's claims. A finding on a stale fact discredits the whole pass.

**Coverage honesty.** A large target may get a partial review. Never issue a clean verdict on a partial pass. State what was inspected and what was not.

## Step 2: The ten lenses

Apply each lens. Skip one only when it is structurally not applicable, and say so.

1. **Goal.** What is this trying to achieve? Does the artifact serve that goal or a proxy of it?
2. **Assumptions.** List the load-bearing assumptions. Which one, if false, collapses the whole thing?
3. **Failure modes.** How does this break in practice? Concrete scenarios, not vibes.
4. **Gaps.** What is unhandled: missing branch, unscoped case, silent-failure path, undefined behavior?
5. **Reversibility.** If this is wrong, what is the blast radius and the rollback path? Flag irreversible steps.
6. **Verification.** How would we know it worked or failed? Is there a test, a metric, an observation? Unfalsifiable is the highest-value finding.
7. **Simpler alternative.** Is there a smaller path to the same goal? Does any component solve an unmeasured problem? Necessity, not only correctness.
8. **Evidence.** Are the factual and quantitative claims externally grounded, or internal logic asserting itself?
9. **Second-order effects.** What does this cause downstream: next week, on the shared module, for the maintainer?
10. **Security.** Who is the adversary and what is the attack surface? Untrusted input handled unsafely, secrets exposed in code or logs or docs, auth gaps, supply-chain trust (`curl | bash`, new dependencies), data-exfiltration paths. For a plan: does executing it create any of these? When a finding is an exposed secret, cite its location and never quote the value.

## Step 3: Output

Emit exactly the structure in `findings-format.md`:

```
VERDICT: <PROCEED | PROCEED WITH CHANGES | REWORK>
Lenses applied: <"1-10" or "1-9; 10 skipped: no security surface">

Load-bearing assumptions:
  - <assumption> → holds? [yes/no/unverified]

Findings (severity-ranked):
  [BLOCKER] <one line> — scenario: <inputs/state → wrong outcome> — <file:line or source>
  [MAJOR]   ...
  [MINOR]   ...
  [NIT]     ...
```

Rules:

- `Lenses applied:` makes a skipped lens visible. A skipped lens must name a structural reason.
- Every finding is a concrete failure scenario. *Fragile* is not a finding. *Empty `orders` divides by zero at line 42* is.
- Severity: Blocker means do not commit. Major means fix or have the user accept. Minor means soon. Nit is optional.
- Verdict, first match wins: any Blocker gives REWORK. One or more un-accepted Majors give PROCEED WITH CHANGES. Otherwise PROCEED.
- Verdicts are whole tokens. Never substring-match PROCEED; it is a prefix of PROCEED WITH CHANGES.
- Lead with one line the target gets right. A clean verdict is a real result, not a sign of under-looking.

## Parallel Mode

For high-stakes or self-authored targets, spawn independent reviewers:

1. **Read-only reviewers.** No edit or write tools. Prefer an analytical planner type over a search type; a search agent locates, it does not critique.
2. **Pass the target.** A spawned agent shares no context. The prompt carries the target path or content, the lens cluster, and the Step 3 format. Lenses without a target review nothing.
3. **Distinct clusters.** A: lenses 1 to 3 and 10. B: lenses 4 to 6. C: lenses 7 to 9. Different priors give different coverage.
4. **One message.** Launch all reviewers in one message. Rotate models when the host allows.
5. **Two-sided reviewer for contracts.** When the target has a producer or consumer (a script and its docs, an API and its caller, two skills that hand off), dedicate one reviewer to both sides' files. It arbitrates every cross-side claim and trusts neither. Contract drift is invisible to single-file review.

Synthesize: dedupe, keep the highest severity, one verdict. Every sub-agent finding stays `[Agent Report — Unverified]` until you confirm the fact yourself (Step 1).

A reviewer that dies or returns prose leaves its cluster UNREVIEWED. Do not synthesize around the gap. Re-run it or flag `[cluster N unreviewed]`. Never PROCEED over an unreviewed cluster.

## Loop Mode

The harness does not parse the verdict. The model reads `VERDICT:` and decides. Drive a loop: review, resolve, re-run. Stop at PROCEED.

- **Accepting a Major is the user's act.** Not the reviewer's, not the agent's. Carry an `Accepted (won't fix):` list between passes. Accepted Majors do not re-report and do not force PROCEED WITH CHANGES.
- Repeat each pass: *Do not re-report resolved or Accepted findings.*
- **Convergence guard.** The same Blocker or Major surviving three passes un-accepted stops the loop and escalates. The disagreement is the signal.

## Fix phase (author side, between passes)

Fixes happen outside the read-only review. Fix-introduced regressions are the loop's main cost.

Ask necessity first (lens 7). A target that has never run and duplicates existing work is a delete candidate. Propose removal, not repair.

Author discipline for Blockers, Majors, or contract-coupled targets:

1. **Contract fixes are two-sided.** Name the mirror clause on the other side before editing. *No mirror* is an explicit placement.
2. **New state gets a lifecycle table first.** Writers, readers, removers, coexistence.
3. **New constraints get a literal-obedience test.** Grep for the text the new rule outlaws. The happy path must survive literal obedience.
4. **The author never closes the loop.** A fix commit is a checkpoint. *Fixed* is claimed only after a clean delta pass by a reviewer over the diff. Until then: fixes applied, unverified.
