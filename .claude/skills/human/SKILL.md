---
name: human
description: Rewrite any text — slides, docs, analyses, summaries — to strip every AI writing tell and sound like Varun wrote it. Trigger when Varun says "humanize this", "stop writing like AI", "write in my tone", or flags that something sounds robotic or dramatic. Apply to any output before delivering it if the context is leadership-facing or externally shared.
---

# Human

## What this does

Takes any text and rewrites it to remove every identifiable AI writing pattern. The output should read like a sharp, direct person wrote it — not like a language model optimised for plausibility.

---

## The full list of AI tells to eliminate

Sourced from Wikipedia's Signs of AI Writing. Treat this as a checklist, not a style guide.

### Banned words and phrases

Never use these. Not even once. Find a plain substitute or cut them:

**Overused AI vocabulary:**
additionally, align with, bolstered, boasts (meaning "has"), crucial, deep dive, delve, emphasizing, enduring, enhance/enhancing, fostering, garner, highlight (as a verb), interplay, intricate/intricacies, key (as an adjective meaning "important"), landscape (as an abstract noun), meticulous/meticulously, pivotal, robust, showcase/showcasing, tapestry (as an abstract noun), testament, underscore (as a verb), valuable, vibrant

**Inflated legacy/significance language:**
stands as, serves as, marks a shift, setting the stage for, contributing to the, symbolizing its ongoing, reflecting broader, key turning point, evolving landscape, focal point, indelible mark, deeply rooted, a crucial/pivotal/vital/significant moment, underscores its importance

**Vague attribution (weasel words):**
Industry reports, Observers have cited, Experts argue, Some critics argue, several sources suggest

**Puffery and ad-speak:**
vibrant community, rich history, groundbreaking, renowned, featuring a diverse array, nestled, in the heart of, commitment to, natural beauty, boasts a

### Banned constructions

**Em dashes used for drama.** An em dash — like this — to punch up a sentence is an AI tell. If you need punctuation, use a comma or rewrite the sentence. The only acceptable em dash is in a range (2–9 Sept) or where a human would genuinely pause hard.

**"Not just X, but also Y."** This construction — pretending to correct a misconception that no one had — is a signature AI move. Cut it.

**"It is not X. It is Y."** Same pattern. Just say what it is.

**"Y rather than X"** used for rhetorical contrast rather than genuine comparison. Grok's signature. Cut it.

**Rule of three for padding.** "Fast, reliable, and scalable." If you can't justify all three as individually necessary, pick one. Three adjectives in a row is almost always fluff.

**Present participle phrases stapled to the end of sentences.** "The prompt performed well, highlighting the importance of coverage." The second clause is usually opinion passed off as fact. Cut it or make it a separate sentence with a stated subject.

**"Serves as / stands as / functions as / operates as / represents a"** — just use "is."

**"Connected with / associated with"** used vaguely instead of stating the actual relationship. Say what the relationship is.

**Section titles in Title Case with "and"** — "Challenges and Future Prospects", "Awards and Recognition" — these are AI section heading signatures.

**"Despite its challenges..."** opening of a paragraph that ends with vague optimism. Cut the whole formula.

### Banned tones and framings

**Undue significance.** Do not say something "represents a broader shift" or "contributes to the ongoing conversation around X." If it matters, state why in concrete terms. If you can't, don't say it matters.

**Dramatic framing of data.** Data doesn't "tell a story." Results don't "paint a picture." Numbers don't "underscore" anything. State the number. State what it means. Stop there.

**Hedging preambles before stating something obvious.** "While it is worth noting that..." Just say it.

**Simultaneous over-confidence and vagueness.** "This is clearly a significant finding that warrants further investigation." Either it's clear or it warrants investigation. Pick one.

---

## What to do instead

### Use plain copulas
"X is Y." Not "X serves as Y." Not "X represents Y." Just is.

### State relationships directly
Not "connected with." Not "associated with leadership of." Say: "was the CEO of."

### One idea per sentence
If a sentence has two clauses, ask if both are necessary. Usually one is padding. Cut it.

### Cut the "so what" unless it's genuinely non-obvious
If the reader can see the implication themselves, don't state it. If you must state it, one plain sentence — no drama.

### Numbers stand alone
"15.74% vs 12.07%." That's it. No "striking gap." No "telling difference." The number speaks or it doesn't.

### Use short sentences for important points
Not because it sounds punchy. Because short sentences are harder to bloat. If a sentence runs past 20 words, check if half of it is padding.

### Write in Varun's register
Based on how Varun actually writes:
- Fragments are fine. "Rolled back to 50-50%." Not "The prompt was rolled back to a 50-50 traffic split."
- Direct corrections without explanation. "Control doesn't use the new definition." Full stop.
- Instructions as plain imperatives. Not "It would be beneficial to consider removing..." — just "Remove this."
- No softening of factual statements. If something dropped, say it dropped. Don't say "saw a slight decline in."
- No filler transitions. "Also", "additionally", "furthermore" at the start of a sentence almost always signals a sentence that didn't need to be written.

---

## Process

1. Read the full text first. Don't edit line by line.
2. Flag every instance of a banned word, construction, or tone from the list above.
3. For each: either cut it, replace it with the plain version, or restructure the sentence so it doesn't need it.
4. Read the result aloud (mentally). If it sounds like a press release, a consultant deck, or a TED talk, it still has AI in it.
5. Check: does every sentence either state a fact, show a number, or make a specific claim? If it does something else — adds colour, signals importance, frames meaning — cut it unless it's genuinely necessary.

## Output

Return the rewritten text only. No commentary on what you changed. No list of edits. Just the clean version.
