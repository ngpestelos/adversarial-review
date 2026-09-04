# adversarial-review

An adversarial review prompt for AI coding agents, packaged as a skill. Point it at a plan, a design doc, or a diff. It finds fault, cites a checked fact for every finding, and emits a verdict you can loop on until clean.

Read-only. It critiques. It never edits.

## Files

- `SKILL.md`: the prompt. Independence check, verify-first rule, ten lenses, output contract, parallel reviewers, loop, fix-phase discipline.
- `findings-format.md`: the output contract and verdict rules on their own, for pasting into a chat or wiring into a script.

## Install

**Claude Code**

```
mkdir -p ~/.claude/skills/adversarial-review
cp SKILL.md findings-format.md ~/.claude/skills/adversarial-review/
```

Run: `/adversarial-review <path>`

**Codex**

```
mkdir -p ~/.codex/skills/adversarial-review
cp SKILL.md findings-format.md ~/.codex/skills/adversarial-review/
```

Or paste the body of `SKILL.md` into `AGENTS.md` under its own heading. Verify against your Codex version; the skills directory layout may differ.

**Any chat**

Paste Steps 1 to 3 from `SKILL.md`. The parallel and loop sections need an agent that can spawn reviewers. The other three need nothing.

## Run

```
/adversarial-review plans/migration.md
```

Read the `VERDICT:` line. Fix or accept. Run again. Stop on `PROCEED`.

## Background

Why each part exists: ["'Tell Me What's Wrong With It' Isn't a Review"](https://ngpestelos.com/writing/tell-me-whats-wrong-with-it-isnt-review/).
The defect classes it hunts in agent systems: ["The Bugs That Hide Between Your Agents"](https://ngpestelos.com/writing/the-bugs-that-hide-between-your-agents/).
