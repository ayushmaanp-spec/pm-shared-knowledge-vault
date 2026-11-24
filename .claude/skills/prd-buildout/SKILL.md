---
name: prd-buildout
description: Turn a draft PRD plus a Claude-generated design prototype (exported as an HTML/React zip) into a detailed, build-ready v0 PRD — one that engineers can work from purely by keyword search (via GStack Skills, so no one reads it linearly while coding) and that also doubles as a walkthrough document for EMs, GPMs, SDE2/3s, and DS counterparts to review live and give feedback on. Trigger this whenever Varun has a rough/draft PRD and a Claude design zip and wants them merged into something "detailed," "build-ready," or "final," when he's prepping to walk engineering/stakeholders through a PRD, when he mentions GStack Skills or an eng handoff, or when he asks to flesh out acceptance criteria from a design prototype — even if he doesn't use these exact words (e.g. "here's my draft and the design export, make this build-ready" or "I need to walk the team through this tomorrow, can you tighten it up").
---

# PRD Buildout

## What this produces and why it's built that way

The output PRD has to satisfy two very different readers, and the structure below exists because of that tension:

1. **Engineers using GStack Skills.** They will never read this document top to bottom. They search for a keyword — a screen name, a user story ID, a field name — and land on one section. If a requirement only exists as a clause buried inside a paragraph three sections away from where the term is used, it effectively doesn't exist for them. Every requirement must be **findable from the term someone would actually search for**, and phrased so a reader who jumps in cold (no context from earlier sections) still understands it.
2. **The walkthrough audience** (EMs, GPMs, SDE2/3s, DS). They see this PRD once, live, for maybe 20–30 minutes, to align and give feedback. They need the top of the document to carry the narrative — problem, goal, who it's for, what's decided vs. still open — without having to reconstruct it from scattered ACs.

Both needs are satisfied by the same document because they operate on different parts of it: the walkthrough reads the top-level narrative sections; the engineer skips straight to a User Story block and its AC table. Don't compromise one for the other — keep the narrative sections tight enough to talk through live, and keep every AC table self-contained enough to stand alone under a keyword hit.

## Inputs required

Ask for both if either is missing — this skill doesn't work well with only one:

1. **The draft PRD** — Varun's rough thinking: problem statement, goals, who it's for, a first pass at user stories. This is the source of *intent* — why a feature exists.
2. **The Claude design prototype export** — a zip of HTML/React from a Claude design session. This is the source of *ground truth on behavior* — actual screens, states, copy, and interactions. Unzip it and read the components/screens directly rather than guessing at behavior from the draft PRD's prose. Where the design and the draft PRD disagree or the draft is vague, the design prototype wins (it's the more concrete artifact) — but flag the disagreement rather than silently picking one.

If Varun only has one of the two, ask whether to proceed anyway (e.g. a pure copy-editing pass on a draft with no design yet) or wait for the other input.

## Process

1. **Read the draft PRD fully first** to understand intent, goals, and the rough shape of user stories before touching the design zip — this keeps you anchored to *why* a screen exists, not just *what* it shows.

2. **Unzip and walk the design prototype screen by screen.** For each distinct screen/state you find (empty state, loading, error, populated, a modal step, etc.), extract: exact copy/microcopy, what fields or actions are present, what's read-only vs. editable, what happens on each interaction, and any conditional logic visible in the component code (e.g. a field only shown when a toggle is on). This is where most of the AC detail comes from — the draft PRD rarely specifies every state a screen can be in, the prototype does.

3. **Merge into User Stories with AC tables**, following this shape (this is the proven format from Varun's Auto Evals PRD — reuse it, don't reinvent):
   - `### USn: <short name>` — one line "As a `<role>`, I want to `<action>`, so that `<outcome>`."
   - An `| AC | Description |` table below it. **Number ACs sequentially with no gaps or repeats** — a duplicate or skipped number is a real defect here, not cosmetic, because an engineer citing "US4 AC3" in a PR or a Slack thread needs that number to be unambiguous.
   - Each AC row is a **self-contained sentence** — someone landing on this row from a search, with zero surrounding context, should be able to act on it. Name the screen/state/field explicitly in the row rather than saying "this field" or "as above."
   - Fold in every state and edge case found in the design zip as its own AC or its own sub-clause, not left implicit.

4. **Keep a running Definitions/glossary table** for every domain term introduced (entity, the object being scored, the actor roles, etc.), and use those exact terms consistently everywhere else in the document. Keyword search only works if the same concept is always called the same thing — if the draft PRD calls something "check" in one place and "metric" in another, pick one term and normalize the whole document to it.

5. **Write the narrative front-matter** for the walkthrough audience, in this order: Context (the scale/problem in concrete numbers), Problem Statement, Right to win (if there's a business case), Goals, Success Metrics (Input/Output/Outcome, each with a one-line "why it belongs here"), Who uses it (roles). Keep this section tight — it's what gets talked through out loud, not read.

6. **Add an "Open Items for This Review" table right after the header**, listing anything genuinely unresolved — a dependency on another team, an undefined number, an explicit open question from a user story. Don't silently resolve these yourself or bury them in prose; surface them as a visible table so the walkthrough audience can weigh in live. This is the one place linear reading matters most, since it's usually the first thing discussed in the review.

7. **Do a final keyword-searchability pass**: pick 5–8 terms a searching engineer would plausibly type (a screen name, a role, a US number, a field name) and mentally search the document for each. If a term's relevant requirements are scattered across sections with no single AC that stands alone, tighten it — either consolidate or make each instance self-contained.

## Output

- One markdown file, saved to `Outputs/` (or wherever the project's finalized-deliverables folder is), named `<Project> - <Feature> PRD v<N>.md`.
- Preserve any embedded images/design references from the draft PRD as-is (copy the file rather than re-typing embedded image data through a context window — it's wasted tokens for no benefit).
- Do not silently rewrite Varun's stated goals, success metrics, or prioritization calls — those are product decisions his call, not a copyedit target. This skill's job is completeness and structure (extracting design detail, fixing structural defects like AC numbering, adding the open-items and glossary scaffolding), not re-litigating decisions already made.
