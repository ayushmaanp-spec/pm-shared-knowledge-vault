---
name: prd-writer
description: Write a fresh PRD from scratch — given raw input context (source docs, notes, tickets, design files, reference PRDs) rather than an existing draft — in the exact structure and voice of Varun's reference "Detailed PRD" (Auto Voice Evals). Trigger this when Varun hands over input material and asks to "write a PRD," "draft a PRD," or "turn this into a PRD" and there is no existing draft PRD yet to build on — only raw context. This is the earlier stage than the `prd-buildout` skill: prd-buildout takes an *existing* draft PRD + a Claude design-prototype zip and hardens it into a build-ready v0; prd-writer authors the draft PRD itself, before that design zip necessarily exists. If Varun already has a rough draft PRD and just wants it fleshed out against a design export, use prd-buildout instead. Trigger even without these exact words — e.g. "here's a doc dump and some Figma screens, write me a PRD for this" or "I have context on this problem, help me structure a PRD."
---

# PRD Writer

## What this produces and why it's shaped this way

The reference document for this skill is Varun's Auto Voice Evals "Detailed PRD." Every section below exists because it does a specific job in that document — reproduce the shape, not just the headings:

1. **Context** — the scale/problem stated in concrete numbers (call volume, GMV, time windows), never in the abstract. If you don't have a real number for something, say so as an open question rather than writing a vague sentence to paper over it.
2. **Problem Statement** — one plain-language framing of the pain ("if someone wants to do X today, they have to do Y — and that doesn't scale because Z"), then decomposed into the separate, compounding sub-problems (usually 2-4) that make it worse than it looks at first glance. Each sub-problem gets its own short label + one-sentence explanation.
3. **Summary** — one tight paragraph, bolded lead clause, naming the core mechanism of the solution in plain terms, with one concrete example.
4. **Right to win** (only if there's a real business case) — quantified, not asserted. Cite the source data (a sheet, a dashboard), show the actual formula/math (e.g. "GMV at risk ≈ (days undetected) × (conversion dip %) × (GMV/day) ≈ ₹X"), and separate the quantified case from any qualitative upside you can't yet size.
5. **Goals** — phased (Phase 1, Phase 2, ...), each goal an imperative statement tied to a concrete guarantee or capability, not a vague aspiration ("give X continuous visibility into Y" / "guarantee no Z happens until W").
6. **Who uses it** — one role, one paragraph: what they know, what they don't (e.g. "not an engineer"), and the one thing the product must let them do without depending on someone else.
7. **Success Metrics** — a table with **Type (Input/Output/Outcome) | Metric | Why it belongs here**. Every metric needs its one-line justification — a metric without a "why it belongs here" is a placeholder, not a finished thought. Input = enablement/rollout levers, Output = usage of the built thing, Outcome = whether the underlying problem actually got better.
8. **Definitions** — a running Term/Meaning glossary of every domain noun introduced. Once a term is defined, use that exact word everywhere else in the document — never introduce a synonym for something already named.
9. **User Stories & Acceptance Criteria** — the bulk of the document:
   - `### USn: <short name>` (append `(Phase 2)` etc. to the heading when relevant) — one line "As a `<role>`, I want to `<action>`, so that `<outcome>`."
   - An `| AC | Description |` table beneath it. **Number ACs sequentially per story, no gaps or repeats** — engineers will cite "US4 AC3" in PRs/Slack, so the number must stay unambiguous.
   - Each AC row is a **self-contained sentence** a reader can act on with zero surrounding context — name the screen/field/state explicitly rather than "this field" or "as above."
   - Bake in the *why* where it's non-obvious, right inside the AC ("...since throwing those away would undo work that has nothing to do with whether background sampling continues") — this is what makes an AC defensible in a review instead of just asserted.
   - Surface unresolved calls as an inline "Open Question" clause inside the relevant AC, not as a silent assumption.

Voice throughout: direct, first-principles, no filler. Justify non-obvious decisions in the same breath you state them. Quantify wherever a number is available; flag it as open when it isn't.

## Inputs required

Ask what's available; proceed with what exists rather than blocking, but be explicit about what's missing:

1. **Problem/context material** — anything with real numbers or firsthand pain: metrics dashboards, prior incident writeups, stakeholder notes, transcripts of problem-framing conversations. This is where Context, Problem Statement, and Right to Win come from — don't invent numbers that aren't in the inputs.
2. **Design files** (Figma, a Claude design-prototype zip, wireframes, screenshots) — if present, walk them screen-by-screen the same way prd-buildout does: extract exact copy, fields, read-only vs. editable states, conditional logic, empty/loading/error states. This is the primary source for AC detail. If no design exists yet for a story, write the ACs from the input docs alone and flag the story as "design pending."
3. **Reference PRDs** (this skill's own reference doc, or another prior PRD Varun points to) — use only for structure/voice matching, never copy their actual product decisions into the new PRD.
4. **Any existing rough notes on goals/success metrics/user roles** — treat as raw input to be verified and reshaped into the structure above, not preserved verbatim.

If there's no problem/context material at all (only design files), ask Varun for at least the scale/pain framing before drafting Context and Right to Win — those sections can't be fabricated.

## Process

1. **Read every input fully before writing anything.** Build a mental (or scratch) list of: hard numbers found, roles mentioned, screens/flows present, and anything stated as already-decided vs. still open.
2. **Draft the narrative front matter first** — Context → Problem Statement → Summary → Right to win → Goals → Who uses it → Success Metrics. This section is what gets talked through live in a review, so get the reasoning chain right before touching user stories: each section should follow causally from the one before it (the numbers in Context justify the Problem Statement; the Problem Statement's sub-problems map to the Goals that address them).
3. **Build the Definitions glossary as you go** — every time you name a new domain concept in step 2, add it here immediately rather than backfilling later.
4. **Walk any design files screen by screen**, then draft User Stories & ACs. Group stories by the flow/surface they belong to, phase-tag anything not in v0 scope, and number ACs sequentially. Where a story's behavior comes only from prose input (no design), write ACs conservatively and mark the story "design pending" rather than guessing at exact UI behavior.
5. **Add an "Open Items for This Draft" table right after the header** — every place you had to flag a missing number, an undecided threshold, a dependency on another team, or a "design pending" story. Don't silently resolve these.
6. **Final pass — keyword-searchability and consistency**: pick 5-8 terms a reader would search for (a role, a screen name, a US number, a field name) and check each resolves to a self-contained AC or section. Check every domain term is used consistently with the Definitions table.

## Output

- One markdown file in `Outputs/`, named `<Project> - <Feature> PRD Draft v1.md`.
- This is a **draft**, not a build-ready v0 — if Varun later gets a design-prototype zip made against this draft, hand off to `prd-buildout` to harden it rather than re-running this skill.
- Don't fabricate numbers, thresholds, or decisions that aren't in the inputs — write them into the Open Items table instead.
