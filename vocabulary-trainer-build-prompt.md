# Prompt: Build a Spaced-Repetition Vocabulary Trainer

Copy everything below into a new conversation with Claude (or another AI coding assistant).

---

## The task

Build a single-file web app that helps someone with self-described low memory learn a fixed set of vocabulary terms through **daily practice**, until the terms are permanently retained. This is not a static quiz — it must implement real spaced-repetition scheduling so that review frequency adapts automatically to what the person actually knows.

I will supply the vocabulary as a JSON deck (schema below). Ask me for it if it's not attached.

## The pedagogy (build to this exactly — don't substitute a simpler quiz loop)

**1. Leitner box scheduling, not a fixed daily list.**
Every card belongs to a box, 1 through 5. Each box maps to a review interval:
- Box 1 → review again tomorrow (1 day)
- Box 2 → 2 days
- Box 3 → 4 days
- Box 4 → 7 days
- Box 5 → 14 days (effectively "mastered," still gets rare refreshers)

New cards start in Box 1. A correct answer moves the card up one box (max Box 5). An incorrect answer sends it straight back to Box 1, regardless of what box it was in. Each card stores its own `box` and `dueDate`. On any given day, only cards whose `dueDate` has arrived are eligible for review — the app must never show a card before it's due.

The point: cards the person is shaky on surface constantly; cards they've proven they know fade into the background automatically. Don't let the person "browse all terms" as a substitute for this — the whole value is in the algorithm deciding what's due.

**2. Recognition before recall.**
Question format depends on the card's current box, not on randomness:
- **Box 1 (new / just-failed cards):** multiple choice. Show the definition, offer 4 term options (1 correct + 3 plausible distractors from elsewhere in the deck), person picks the term. This is recognition — easier, appropriate for a term they haven't secured yet.
- **Box 2 and above:** fill-in-the-blank / free recall. Show a cloze sentence with the term blanked out, person types the answer. This is production — harder, and it's what builds durable memory once recognition is already working.

Do not use flashcard-style "flip and self-grade" for this — the app should evaluate the answer itself (exact/fuzzy match for typed answers, exact match for MCQ) so the honesty of "did I actually know that" isn't left to the person's self-assessment, which is unreliable for someone worried about their memory.

**3. Personalized memory hooks over dictionary definitions.**
Each card has a personal "why this matters to you" line (specific to the person's own project/context) in addition to the neutral definition. After every answer — right or wrong — show the definition, a plain-language analogy, and the personal relevance line, in that order. The personal line is doing the real memory work; the definition alone is what's easy to forget. Never skip showing this feedback, even on a correct answer — reinforcement matters as much as correction.

**4. Pace the daily session — cap it.**
Even if 30+ cards are due on a given day, cap the session (e.g. 10–15 cards). Sort the queue so lower-box cards (things being forgotten) come first. The goal is a sustainable daily habit for someone with limited memory bandwidth, not an overwhelming backlog-clearing marathon. Never let the app frame "everything due" as a single guilt-inducing number — show it, but let the cap protect the person from it.

**5. Persistent state across sessions.**
The following must survive a page reload / new day: each card's box and due date, a day-over-day streak counter, and total review count. If building this as a Claude.ai artifact, use the `window.storage` API (never `localStorage`/`sessionStorage` — those are unsupported and will silently fail in that environment). If building it as a standalone web app elsewhere, `localStorage` or a small backend is fine — just make the persistence real, not in-memory-only.

## JSON deck schema

Each card looks like this:

```json
{
  "id": "unique_slug",
  "category": "Grouping label",
  "term": "The term itself",
  "definition": "One or two plain-language sentences.",
  "analogy": "A short everyday comparison.",
  "why_it_matters": "The personalized line tying it to the person's actual situation/project.",
  "cloze": "A sentence with ___ where the term goes.",
  "mcq_options": ["Correct term", "Distractor 1", "Distractor 2", "Distractor 3"]
}
```

`mcq_options` should include the correct term plus 3 real distractors pulled from other terms in the same deck (not made up) so wrong options are still plausible, not throwaway.

## What "done" looks like

- Session view: shows streak, mastered count (cards in Box 5), and how many are due right now.
- Question view: shows category label, a visual indicator of which box the card is currently in (e.g. a small progress track), and the question in the correct format for that box.
- Answer flow: person answers → immediate right/wrong feedback → definition + analogy + personal relevance line → explicit "Next" action (don't auto-advance; let them sit with the correction).
- End-of-session state: correct count out of cards reviewed, and a clear "come back tomorrow" message — not an empty screen.
- Empty state when nothing is due: reassure the person the schedule is working, not that they're "done" or "behind."

## Visual style

Warm, editorial, slightly old-fashioned — parchment background, deep forest green for headers, amber for accents, DM Mono for small labels/metadata, Playfair Display serif for headings and term names, DM Sans for body text. Should feel like a considered piece of paper, not a generic app UI. Single file, no external dependencies beyond Google Fonts.

## Constraints

- Single HTML file (inline CSS and JS), or single React component if the target environment is React-based — ask which before starting if unclear.
- No backend required; all logic client-side.
- No localStorage/sessionStorage if this is a Claude.ai artifact — use `window.storage` per that platform's persistence API instead.
- Must handle the deck growing over time (new cards added to the JSON later should initialize into Box 1 automatically, without wiping existing progress on cards already being tracked).

---

Attach or paste the vocabulary JSON deck as the next message.
