# Evaluator patterns

Select the cheapest reliable evidence first. Combine evaluator types only when each checks a distinct failure mode.

## Software and web applications

Use, in order:

1. build, type, lint, and unit/integration tests;
2. runtime and browser interaction checks;
3. visual comparison against supplied references;
4. accessibility and performance checks;
5. rubric review for maintainability and polish.

Good must-have criteria include successful build, required flows working, no console errors, keyboard access, and exact data behavior. Treat subjective visual preference as Polish unless the brief makes it essential.

## Research and analysis

Use, in order:

1. source existence and metadata checks;
2. claim-to-source traceability;
3. calculation or quotation verification;
4. coverage against the research questions;
5. synthesis review for relevance and clarity.

Separate source gathering from synthesis when possible. Require the evaluator to identify unsupported claims rather than rewrite the report from memory.

## Design and creative work

Ground evaluation in supplied references, brand rules, intended audience, format, and usage context.

Evaluate:

- brief and brand fidelity;
- required content and hierarchy;
- legibility and accessibility;
- technical/export constraints;
- composition and polish.

Add a human checkpoint before extended iteration when the direction is subjective. Do not let a model critic invent a new brand direction.

## Data and spreadsheets

Use schema checks, row-count reconciliations, formula tests, totals, null/duplicate checks, and known sample cases before visual review. Require reproducible calculations and record any transformations.

## Written content

Check factual grounding and required coverage before tone or elegance. Use explicit criteria for audience, structure, length, prohibited claims, and calls to action. Reserve multiple critics for genuinely distinct dimensions such as legal review and brand voice.

## Evaluator result format

Require each evaluator to return:

```text
Verdict: PASS | REVISE | BLOCKED
Criterion: [rubric item]
Evidence: [test output, source, screenshot, calculation, or exact artifact location]
Failure: [specific gap, if any]
Required fix: [smallest change that satisfies the criterion]
Confidence: high | medium | low
```

Reject feedback that is purely stylistic, expands scope, or cannot point to evidence.
