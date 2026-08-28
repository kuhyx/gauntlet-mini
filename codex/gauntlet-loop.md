---
description: Build and test with the gauntlet-loop technique — bounded worker/evaluator passes with an explicit usage budget
argument-hint: [what to build, and optionally a usage budget like "use 50% of my limit"]
---

# Gauntlet Loop

Apply the gauntlet-loop technique to the request that follows (or, if none follows, to the task already in progress): turn one substantial build request into a bounded set of worker passes with clear ownership, evaluators with real evidence, and revision loops that run until the quality checks pass — bounded so the loop converges instead of wandering.

Default to running it. Fall back to a single direct pass only when the task is genuinely small or so tightly coupled that splitting it would cause collisions — then just do the work well, noting the call in one line.

If the user asks for just the prompt or spec ("convert this"), return the copy-ready gauntlet prompt without executing it.

## Set the usage budget first

A gauntlet spends the user's usage limits (the 5-hour and weekly windows on ChatGPT plans) far faster than a single pass — every extra worker and revision cycle multiplies spend.

- If the user already stated a budget ("use 75% of my limit"), honor it and move on — never re-ask.
- Otherwise, ask one question before building the spec: **"How much of your remaining 5-hour usage window should this build use — 50% (recommended starting point), 75% (bigger builds), or 95% (maximum-quality run)?"** Default to 50% if they decline to choose. They can check remaining usage with `/status`.
- Treat the chosen share as the run's hard budget: size the number of tracks, the model choices, and the cycle count to fit inside it.
- Escalate only up the ladder, only with consent: if the budget is spent while must-have checks still fail — or the result passes but the user wants more polish — stop, report what the spend bought, and offer the next rung: 50% → 75% → 95%. Never plan past 95%; the remaining headroom is the user's.

## Set up the gauntlet

1. **Normalize the brief.** Pin down the outcome, source-of-truth inputs, must-haves, deliverables, and acceptance criteria. Don't interrogate the user — proceed on sensible assumptions and label the material ones.
2. **Ground the direction.** If a validated baseline, brief, or draft exists, treat it as the reference. If none exists, run one strong solo pass first to establish the target. The gauntlet polishes a sound target; it can't rescue an accidental one.
3. **Decompose by ownership.** 2–4 tracks by default, each with one non-overlapping objective and exclusive ownership of its files/sections/domains, an expected artifact, and local checks it must pass before handing off. Never let two tracks touch the same artifact in the same cycle.
4. **Design the tests before the build**, strongest evidence first:
   1. deterministic — builds, tests, type checks, schemas, linters, calculations;
   2. execution evidence — run it, render it, browser-check it;
   3. comparison against source-of-truth references;
   4. rubric-based critique for what only judgment can see — citing concrete failures against the rubric, never vibes.
5. **Bound the loop.** Default 3 revision cycles; stop early when every must-have and should-level check passes; stop and report when a cycle fixes nothing, evidence is unavailable, a needed decision is missing, or the budget share is spent.

## Execute (Codex adaptation)

Codex has no built-in parallel subagent tool, so run the fan-out one of two ways:

- **Sequential passes (default):** run each track as its own focused pass inside this session, in dependency order, holding each to its ownership boundary and local checks before moving on. The discipline — one owner per artifact, tests designed up front, evidence-backed revision — is what delivers the quality; parallelism is only a speed optimization.
- **Subprocess workers (when shell access allows):** spawn workers with `codex exec "<track brief>"` from this session, writing each track's brief to give it the minimum complete context and its ownership boundary. Tier models to the work with `-m`: keep direction-setting, integration, and the final verdict on the strongest available model; run well-specified implementation and research tracks on a faster/cheaper model. Every agent on a stronger model than its track needs is quality-neutral waste.

Either way:

1. Show the setup in a few lines before starting — budget share, track roster with ownership, test plan, cycle budget. This is a heads-up, not a permission request.
2. Run evaluators only once artifacts exist. Each evaluator verdict must cite evidence: test output, a rendered result, a source, a calculation, or an exact artifact location. Feed concrete failures back to the owning track for the next cycle. Reject feedback that is purely stylistic, expands scope, or can't point to evidence.
3. Integrate at the end: confirm every artifact arrived, resolve conflicts and coverage gaps, and re-run the full evaluation order on the integrated result — not just the pieces. Unresolved failures get reported, not hidden.

## Final response

Lead with the tested outcome: what passed and the evidence, what changed across cycles, remaining risks or unverified criteria, and a one-line accounting of roughly how much of the chosen budget was used.

Don't declare done while a must-have check fails — and don't keep polishing after the budget is spent.
