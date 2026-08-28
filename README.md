# Gauntlet Loop

A Claude Code skill (with a Codex port — see [Use with Codex](#use-with-codex-openai)) that turns one big request into a small team of AI agents that build, check each other's work, and revise until the result actually passes its quality tests — instead of one agent doing one long pass and hoping for the best.

Think of it like a general contractor running a job site: one lead sets the plan, specialists each own their piece, inspectors check the work against a punch list, and nothing ships until the punch list is clear.

## Why it exists

Asking an AI to "make it amazing" usually gets you one long, unchecked draft. The gauntlet flips that: the quality bar is written down **before** the work starts, separate workers build in parallel without stepping on each other, and evaluators verify the result with real evidence (tests, screenshots, source checks) — not vibes. Failed checks go back to the responsible worker for another pass. The loop is bounded, so it converges instead of running forever.

## Install

Requires [Claude Code](https://claude.com/claude-code).

```bash
git clone https://github.com/cyog941/gauntlet-loop.git ~/.claude/skills/gauntlet-loop
```

That's it. Open a new Claude Code session and the skill is available as `/gauntlet-loop`.

## ⚠️ Before you run it: set a session budget

**This skill uses a lot of your 5-hour session usage limit.** Running several agents in parallel multiplies token spend — that's the whole point, and it's also the cost. So the skill is built around an explicit budget:

- **Best way: tell Claude your budget up front**, in the same message as your request:

  > `/gauntlet-loop build me a landing page for my product — use 50% of my session limit`

- **If you don't, the skill asks first.** Before spinning anything up, it asks how much of your remaining session limit the build may use and offers three tiers:

  | Tier | When to pick it |
  |------|-----------------|
  | **50%** — recommended starting point | Most builds. Leaves half your session for everything else. |
  | **75%** | Bigger builds, or a second pass after a 50% run got close. |
  | **95%** | Maximum-quality run. Assume the session is spent afterward. |

- **Start at 50%.** If the result isn't there yet when the budget runs out, the skill stops, shows you what the spend bought, and offers the next rung — 50% → 75% → 95%. It never escalates on its own, and it never plans past 95%.

You can check what's left of your session anytime with `/usage` inside Claude Code.

## How it works (plain-English version)

When you invoke it, the skill walks through seven steps:

1. **Understand the job.** It pins down what you want, what "done" means, and what materials it has to work from — without grilling you with questions.
2. **Get the direction right first.** If there's no existing draft or plan to build on, it does one strong solo pass to establish the target before bringing in the team. Polishing the wrong thing fast is still the wrong thing.
3. **Split the work by ownership.** Usually 2–4 workers, each with one clear job and exclusive ownership of their piece, so nobody overwrites anyone else.
4. **Write the tests before the build.** The quality checklist comes first: automated checks where possible (does it build? do the numbers add up?), then real-world evidence (run it, open it in a browser), then judgment calls against a written rubric.
5. **Right-size the brainpower.** The strongest model handles direction and final sign-off; cheaper, faster models handle the well-specified grunt work. Same quality, much less of your session limit burned.
6. **Loop with a leash.** Workers revise based on specific, evidence-backed feedback — up to a set number of rounds, within your chosen session budget. It stops early when everything passes, and stops honestly when it's stuck.
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

## Use with Codex (OpenAI)

Codex doesn't have skills, but it has custom prompts — any markdown file in `~/.codex/prompts/` becomes a slash command. A self-contained port lives in [`codex/gauntlet-loop.md`](codex/gauntlet-loop.md):

```bash
mkdir -p ~/.codex/prompts
curl -fsSL https://raw.githubusercontent.com/cyog941/gauntlet-loop/main/codex/gauntlet-loop.md \
  -o ~/.codex/prompts/gauntlet-loop.md
```

(Or copy the file from a clone.) Restart Codex and run `/gauntlet-loop <your request>`.

What's different in the Codex version:

- **Budget = your ChatGPT usage window.** Same 50% → 75% → 95% ladder, measured against the 5-hour usage window on ChatGPT plans. Check what's left with `/status` inside Codex.
- **Workers run as sequential passes by default** — Codex has no built-in parallel subagent tool. The quality comes from the same discipline (one owner per piece, tests written first, evidence-backed revisions), not from parallelism. When shell access allows, it can instead spawn real workers via `codex exec`, using `-m` to put cheaper models on the grunt-work tracks.

## Other agents

The technique itself is agent-agnostic: budget first, ground the direction, split by ownership, write the tests before the build, loop with a leash, verify the integrated whole. To port it to any other coding agent (Gemini CLI, Cursor, aider, …), take `codex/gauntlet-loop.md` — it's the self-contained version — and drop it wherever that agent takes custom instructions (its prompt/command folder, or its `AGENTS.md`-style instructions file), swapping the budget wording for whatever usage limit that agent has.

## What's in the repo

```
gauntlet-loop/
├── SKILL.md                        # the Claude Code skill
├── codex/
│   └── gauntlet-loop.md            # self-contained Codex custom-prompt port
└── references/
    ├── evaluator-patterns.md       # how work gets checked, by discipline
    └── examples.md                 # worked examples of good/bad gauntlet fits
```

## License

[MIT](LICENSE)
