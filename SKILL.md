---
name: gauntlet-mini
description: Build and test with the gauntlet-loop technique. Convert a substantial prompt into a bounded worker/evaluator fan-out, then set it up and execute it end-to-end at the highest quality the session budget allows, tiering models per role (top-tier director/verifier where judgment decides the outcome; cheaper workers elsewhere). Use only when the user mentions a gauntlet/guantlet loop or gauntlet-mini, or asks to iterate a task with critic review until it is production-ready. Do not use for general requests to fan out subagents or run parallel workers. Produce a prompt-only conversion when the user explicitly asks for just the prompt or spec.
---

# Gauntlet Mini

Turn a "fan out agents until the result is amazing" request into a build-and-test machine: workers with clear ownership, evaluators with real evidence, and revision loops that run until the quality checks pass, bounded so the loop converges instead of wandering.

The user invoked this skill because they want the technique applied. Default to running it. Fall back to a single agent only when the task is genuinely small or so tightly coupled that parallel workers would collide, and then just do the work well, noting the call in one line rather than lecturing about fit.

User and repository instructions (CLAUDE.md, AGENTS.md, or the agent's equivalent) override this skill, including when to stop and ask. Read the repository's own instruction file before setting up the gauntlet.

## Mode

- **Execute** (default): build the spec, run the gauntlet, deliver the tested result.
- **Convert**: only when the user explicitly asks for just the prompt or spec. Return the copy-ready prompt without running it.
- **Improve**: revise an existing gauntlet prompt, keeping useful constraints and cutting redundant agents or unbounded loops.

## Set the session budget first

A gauntlet spends the user's session usage limit (the 5-hour window on most plans) far faster than single-agent work, because fan-out multiplies every token. Before any fan-out, establish how much of the session the run may use:

- **If the user already stated a budget** ("use 75% of my session limit", "keep this under a third of my session"), honor it and move on. Never re-ask.
- **Otherwise, ask one question before building the spec** (use AskUserQuestion when available): *"How much of your remaining 5-hour session limit should this build use?"* with three options: **50% (recommended starting point)**, **75% (bigger builds)**, and **95% (maximum-quality run that leaves little session for anything else)**. Default to 50% if the user declines to choose or the session is non-interactive.
- **Treat the chosen share as the run's hard budget.** Size the roster and model tiers to fit inside it. Check remaining usage at phase boundaries using whatever signal the harness provides (a remaining-token counter in context, `/usage`, or context telemetry); when no signal exists, estimate conservatively and say so.
- **Escalate only up the ladder, only with consent.** If the budget is spent while must-have checks still fail, or the result passes but the user wants more polish, pause, report what the spend bought, and offer the next rung: 50% → 75% → 95%. Never escalate without the user's say-so, and never plan past 95%: the remaining headroom is the user's, not the gauntlet's.

## Set up the gauntlet

### 1. Normalize the brief

Extract the outcome, source-of-truth inputs, must-haves, deliverables, acceptance criteria, and any budget hints. Ask the user as many questions as it takes to stop guessing on anything that would change the result (use AskUserQuestion when available, batching related questions); a question costs far less than a fleet built on a wrong guess. Label whatever is still assumed after that.

### 2. Ground the direction

If a validated baseline, brief, design system, or MVP exists, treat it as the reference. If none exists, run Phase 0 first: research the existing product and materials, then one strong pass that establishes the brief or baseline the fleet will sharpen. When the research leaves the direction unclear, ask the user before fan-out instead of picking one. During the run, ask again whenever a decision comes up that the brief does not answer, rather than guessing and continuing. The gauntlet polishes a sound target; it can't rescue an accidental one, and every worker-hour spent on the wrong target is pure waste.

### 3. Decompose by ownership

2 to 4 workers by default; more when the task genuinely splits further. Each worker gets one non-overlapping objective, the minimum complete context, exclusive ownership of its files/sections/domains, an expected artifact, and local checks it must pass before handing off. Keep tightly coupled work sequential and never point two workers at the same artifact concurrently: merge conflicts burn more budget than parallelism saves.

### 4. Design the tests before the build

Quality comes from the evaluation design, so wire it before fan-out, strongest evidence first:

1. deterministic: builds, tests, type checks, schemas, linters, calculations;
2. execution evidence: run it, render it, browser-check it;
3. comparison against source-of-truth references;
4. rubric-based model critique for what only judgment can see.

Two mechanics make the critique layers honest:

- **Critics get fresh context.** A critic that watched the build inherits the builder's blind spots and grades leniently. Spawn evaluators with the criteria, the artifact, and the reference, and nothing about how the work was made or how hard it was.
- **When a real reference exists, compare blind.** Put the artifact and the reference side by side without labels and ask which wins and why: a binary pick with cited reasons, not a score. Blind comparison is the technique's sharpest tool, because a model that knows which one is "ours" roots for it. When no real reference exists, do not let the critic invent one; that is how open-ended gauntlets burn tokens chasing an arbitrary bar. Fall back to the observable criteria from the brief instead.

Model critics cite concrete failures against the rubric, never vibes, and never substitute for available ground truth. Critics also make findings up, so verify each one before anyone fixes it: open the file, line, source, or screenshot it cites and confirm the failure is really there. Drop a finding that points at something that does not exist, and count only verified findings toward the failing total. Share evaluators by discipline instead of pairing one critic to every worker; modern workers self-check, and mirrored critics multiply cost without moving quality. Read [references/evaluator-patterns.md](references/evaluator-patterns.md) when choosing checks for software, research, design, data, or written work.

### 5. Tier models to the work

Fan-out multiplies token spend against the shared session usage limit, so every agent running on a stronger model than its track needs is quality-neutral waste. Assign a model per role when delegating (the Agent tool and Workflow `agent()` both accept a `model` option; omitting it inherits the session's top model, which is exactly the expensive default to avoid on routine tracks):

- **Director / orchestrator and final verifier**: the strongest model the session offers when the role carries consequential judgment: ambiguous direction, integration verdicts, subjective quality gates where a wrong call wastes the whole fleet's work. When direction is validated and verification is mostly mechanical, one tier down is enough.
- **Workers**: a strong reasoning model (e.g. Opus) for tracks needing deep reasoning, architecture, or novel design; a fast mid-tier model (e.g. Sonnet) for well-specified implementation, research sweeps, data transforms, and content passes where the spec already did the thinking.
- **Critics / evaluators**: deterministic checks cost no model at all; prefer them. Rubric critics run well on a mid-tier model; reserve the top tiers for the final integration verdict where judgment genuinely decides the outcome.

Pick the cheapest tier that passes the role's own checks; upgrade a role when a failed check shows the model, not the spec or inputs, was the limit. Quality is protected by the evaluation design, not by putting the biggest model everywhere.

### 6. Bound the loop so it converges

The user decides when the result is good enough. The checks and the budget keep each cycle honest until then:

- revision cycles: no fixed count. After each integrated cycle, show the user the result, which checks pass and fail, and the failing count against the previous cycle, then ask whether to run another cycle or accept;
- measure progress as the number of failing Must and Should checks; a cycle moved the result only if that number dropped;
- when every Must and Should check passes, say so and ask whether to accept or spend the remaining budget on polish;
- stop when the user accepts the result;
- stop and report when a cycle fixes nothing (the failing count did not drop), needed evidence is unavailable, or a missing decision or dependency blocks the fleet, then ask the user how to proceed;
- stop when the session budget the user chose is reached. Report, and offer the next budget rung rather than silently overrunning;
- translate subjective quality into observable criteria. "Iterate until it's amazing" is a budget with no bottom.

Keep the run's state in a `gauntlet-state.md` file in the workspace: the Must/Should/Polish checklist with each item's status, the cycle number, the failing count per cycle, and the budget used so far. Update it after every cycle and re-read it after the conversation is summarized, so a long run never loses count.

### 7. Integrate and verify end-to-end

The orchestrator (or a named integrator) confirms every artifact arrived, resolves conflicts and coverage gaps, runs the full evaluation order on the integrated result, not just the pieces, and produces one coherent deliverable. Unresolved failures get reported, not hidden; a clean-looking bundle of unmerged outputs is not a result.

## Run it

1. Build the spec above compactly, write it to `gauntlet-state.md`, and show the setup in a few lines (session-budget share, roster with ownership and model tier, the Must/Should/Polish checklist the critics will grade against, test plan), then fan out. This is a heads-up, not a permission request.
2. Pass each worker its model tier explicitly when spawning it.
3. Run evaluators only once artifacts exist; feed concrete failures back to the owning worker for the next cycle.
4. Keep integration and final end-to-end verification with the primary agent unless a specialized integrator earns its seat.
5. Lead the final response with the tested outcome: what passed and the evidence, what changed across cycles, remaining risks or unverified criteria, and a one-line accounting of which roles ran on which tier and roughly how much of the chosen budget was used.

## Convert output (Convert and Improve modes)

Return: the fit decision in one sentence, material assumptions, the spec (outcome, references, constraints, Must/Should/Polish rubric, roster with ownership and model tier, phases, evaluation order, and stop rules), and a copy-ready prompt using this skeleton:

```text
Outcome
Build [artifact/outcome] for [user/purpose] using [references].

Constraints
- [must-haves]
- Do not [non-goals]
- Stay within [environment/budget, e.g. 50% of the remaining session limit]

Orchestration
- Establish or validate the baseline before fan-out.
- Delegate these independent tracks: [roster with ownership and model tier].
- Run direction-setting and final verification on [top-tier model]; run workers
  on the cheapest tier that passes their checks ([tier per track]).
- Integrate through [orchestrator/integrator].

Evaluation
Check in this order: [deterministic], [execution/reference evidence], then
[subjective rubric]. Critics get fresh context; use blind comparison when a
real reference exists. Must-have failures block completion.

Loop
Keep the checklist, cycle number and failing count in gauntlet-state.md. After
each cycle, show me the result and the failing Must/Should count, and ask whether
to run another cycle or accept. Stop when I accept. Stop and report if a cycle
stops moving the result, a decision is missing, or the session-budget share is
spent, then offer the next budget rung (50 → 75 → 95%).

Final response
The integrated artifact, evidence per must-have, changes made across cycles,
and unresolved issues.
```

Keep the conversion concise, roughly 500 to 900 words, and read [references/examples.md](references/examples.md) when a concrete pattern would remove ambiguity.

## Keep it honest

- Every loop bounded by the user's acceptance and the session budget; every delegated role carries an explicit model tier and the run carries an explicit session-budget share.
- No overlapping ownership, no evaluator claims without evidence, no hidden failures.
- Don't declare done while a must-have check fails, and don't keep polishing after the budget is spent.
