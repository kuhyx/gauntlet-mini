# Gauntlet Loop

An agent skill that turns one big request into a small team of AI agents that build, check each other's work, and revise until the result actually passes its quality tests — instead of one agent doing one long pass and hoping for the best.

It's agent-agnostic: it ships in the standard `SKILL.md` agent-skill format, plus a self-contained prompt port for agents that don't load skills.

Think of it like a general contractor running a job site: one lead sets the plan, specialists each own their piece, inspectors check the work against a punch list, and nothing ships until the punch list is clear.

## Why it exists

Asking an AI to "make it amazing" usually gets you one long, unchecked draft. The gauntlet flips that: the quality bar is written down **before** the work starts, separate workers build in parallel without stepping on each other, and evaluators verify the result with real evidence (tests, screenshots, source checks) — not vibes. Failed checks go back to the responsible worker for another pass. The loop is bounded, so it converges instead of running forever.

## Install

**Agents that load `SKILL.md` skills** — clone this repo into your agent's skills folder. For example:

```bash
git clone https://github.com/cyog941/gauntlet-loop.git ~/.claude/skills/gauntlet-loop
```

Open a new session and it's available as `/gauntlet-loop`.

**Codex** — no skill loader, but any markdown file in `~/.codex/prompts/` becomes a slash command. A self-contained port lives in [`codex/gauntlet-loop.md`](codex/gauntlet-loop.md):

```bash
mkdir -p ~/.codex/prompts
curl -fsSL https://raw.githubusercontent.com/cyog941/gauntlet-loop/main/codex/gauntlet-loop.md \
  -o ~/.codex/prompts/gauntlet-loop.md
```

Restart Codex and run `/gauntlet-loop <your request>`. One difference: with no parallel subagent tool, workers run as sequential ownership-bounded passes by default — or as real workers via `codex exec -m <model>` when shell access allows.

**Any other agent** — [`codex/gauntlet-loop.md`](codex/gauntlet-loop.md) doubles as the portable, self-contained version. Drop it wherever your agent takes custom prompts or instructions (its prompt/command folder, or its `AGENTS.md`-style instructions file) and adjust the budget wording to that agent's usage limits. The technique itself — budget first, ground the direction, split by ownership, write the tests before the build, loop with a leash, verify the integrated whole — ports anywhere.

## ⚠️ Before you run it: set a usage budget

**This skill spends your usage limit fast.** Running several agents in parallel multiplies token spend — that's the whole point, and it's also the cost. So the skill is built around an explicit budget, expressed as a share of your remaining usage window (the 5-hour rolling limits most plans have):

- **Best way: tell the agent your budget up front**, in the same message as your request:

  > `/gauntlet-loop build me a landing page for my product — use 50% of my limit`

- **If you don't, the skill asks first.** Before spinning anything up, it asks how much of your remaining usage window the build may use and offers three tiers:

  | Tier | When to pick it |
  |------|-----------------|
  | **50%** — recommended starting point | Most builds. Leaves half your window for everything else. |
  | **75%** | Bigger builds, or a second pass after a 50% run got close. |
  | **95%** | Maximum-quality run. Assume the window is spent afterward. |

- **Start at 50%.** If the result isn't there yet when the budget runs out, the skill stops, shows you what the spend bought, and offers the next rung — 50% → 75% → 95%. It never escalates on its own, and it never plans past 95%.

Check what's left of your window anytime with your agent's usage command (typically `/usage` or `/status`).

## How it works (plain-English version)

When you invoke it, the skill walks through seven steps:

1. **Understand the job.** It pins down what you want, what "done" means, and what materials it has to work from — without grilling you with questions.
2. **Get the direction right first.** If there's no existing draft or plan to build on, it does one strong solo pass to establish the target before bringing in the team. Polishing the wrong thing fast is still the wrong thing.
3. **Split the work by ownership.** Usually 2–4 workers, each with one clear job and exclusive ownership of their piece, so nobody overwrites anyone else.
4. **Write the tests before the build.** The quality checklist comes first: automated checks where possible (does it build? do the numbers add up?), then real-world evidence (run it, open it in a browser), then judgment calls against a written rubric.
5. **Right-size the brainpower.** The strongest model handles direction and final sign-off; cheaper, faster models handle the well-specified grunt work. Same quality, much less of your usage limit burned.
6. **Loop with a leash.** Workers revise based on specific, evidence-backed feedback — up to a set number of rounds, within your chosen budget. It stops early when everything passes, and stops honestly when it's stuck.
7. **Assemble and verify the whole.** One agent puts the pieces together and re-checks the final result end-to-end, then reports what passed, what changed, and anything still unresolved.

## Best ways to use it

**Great fits:**
- Building something substantial from scratch — a landing page, an app feature, a report, a deck — where "good" is checkable.
- Improving an existing thing without changing its direction (audits + fixes across design, performance, and accessibility, for example).
- Research that splits naturally — several competitors, several markets, several papers — with one synthesis at the end.

**Bad fits (the skill will just do these directly with one agent):**
- Small, tightly-scoped changes. A one-line fix doesn't need a committee.
- Work so intertwined that parallel workers would collide on the same file.

**Tips:**
- Give it your raw materials — briefs, examples, links, an existing draft. The gauntlet sharpens a target; it doesn't guess one.
- Say what "must be true" for you to accept the result. Concrete must-haves become the tests.
- Want just the plan, not the run? Say so: *"convert this to a gauntlet prompt"* returns a copy-ready prompt instead of executing it.

## What's in the repo

```
gauntlet-loop/
├── SKILL.md                        # the agent skill (standard SKILL.md format)
├── codex/
│   └── gauntlet-loop.md            # self-contained portable port (Codex custom prompt)
└── references/
    ├── evaluator-patterns.md       # how work gets checked, by discipline
    └── examples.md                 # worked examples of good/bad gauntlet fits
```

## License

[MIT](LICENSE)
