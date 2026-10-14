# Conversion examples

## Example 1: Product landing page

User prompt:

> Build a premium landing page for my electrolyte drink using a gauntlet loop.

Fit decision: Use a gauntlet only after establishing the brand brief and page strategy.

Suggested phases:

1. Baseline: confirm audience, positioning, claims, brand references, and required sections.
2. Parallel workers:
   - content/claims researcher owns source-backed product copy;
   - visual-system worker owns tokens and composition direction;
   - implementation worker owns the page code after receiving the approved baseline.
3. Evaluation:
   - build and browser checks;
   - required-section and claim-source checks;
   - accessibility/performance checks;
   - screenshot comparison and final visual rubric.
4. After each cycle, show the page and the failing must-have count, and stop when the user accepts it.

Avoid multiple workers editing the same page at once. Do not grade only on whether it “looks premium.”

## Example 2: Competitive research report

User prompt:

> Research five competitors and make the analysis extremely thorough.

Fit decision: Use a lighter gauntlet if the competitors can be researched independently.

Suggested workers: one or two source gatherers covering distinct competitor sets, plus a synthesis owner. Use one grounding evaluator to trace claims to sources and one coverage check against the research questions. Do not create five writers whose prose must later be reconciled.

## Example 3: Small code change

User prompt:

> Change the submit button label and use a gauntlet loop to verify it.

Fit decision: Use one agent. The work is small, tightly scoped, and directly verifiable.

Improved prompt:

```text
Change the submit button label to [text], run the relevant focused test or browser
check, and report the file changed and verification result. Do not modify anything else.
```

## Example 4: Existing MVP refinement

User prompt:

> Turn this working dashboard into a gauntlet prompt that improves quality without changing its product direction.

Fit decision: Use a gauntlet because a baseline exists and improvement tracks can be separated.

Suggested workers:

- interaction/accessibility audit;
- performance/data-loading audit;
- visual-system consistency audit.

Have the integrator approve proposed changes against the existing design system before implementation. Use deterministic tests and browser evidence first, then one final visual review. Ask the user after each cycle whether to continue, and list rejected changes that would alter product direction.

## Example 5: Client-ready slide deck

User prompt:

> Use a gauntlet loop to build a 10-slide deck on our Q3 results and get it client-ready.

Fit decision: Use a light gauntlet once the audience, the story and the source numbers are pinned down; ask for them before building rather than inventing figures.

Suggested phases:

1. Baseline: confirm the audience, the one message per slide, the source data and any brand template.
2. Parallel workers:
   - narrative worker owns the slide outline and speaker notes;
   - data worker owns every number and chart, each traced to its source;
   - design worker owns the template and layout once the outline is fixed.
3. Evaluation:
   - export every slide to an image and inspect each one: text overflow, contrast, legibility at presentation size, one idea per slide;
   - check every number and claim against its source;
   - blind comparison against an earlier client deck when one exists.
4. After each cycle, show the slides and the failing must-have count, and stop when the user accepts the deck.

Never grade a deck from its outline or its file alone; a slide that reads well as text can still overflow or be unreadable once rendered.
