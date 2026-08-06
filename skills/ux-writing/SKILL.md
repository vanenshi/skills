---
name: ux-writing
description: >-
  Use when writing or editing any user-facing product text: interface copy (microcopy), UI strings, buttons, labels, error messages, notifications, forms, placeholders, tooltips, dialogs, onboarding, empty states — in English or Persian (Farsi) — or when translating/localizing app text, auditing product content, or improving existing interface text, even if "UX writing" isn't mentioned.
---

# UX Writing

Write clear, concise, user-centered interface copy (UX text/microcopy) for digital products. UX writing is not copywriting: copy persuades and sells; UX text helps the user complete a task. Every string must deliver **information** (what the user needs, when they need it), **logic** (how this screen connects to the next), and **empathy** (match the user's emotional state).

## Language Rules (always load one)

The patterns below are language-agnostic. Mechanics, capitalization, and terminology are NOT — before writing any text, read the rules file for the target language and apply it strictly:

- **English:** `references/english-rules.md` — sentence case, contractions, Submit/Cancel/Delete/Sign-in terminology traps, plain-word swaps.
- **Persian (Farsi):** `references/persian-rules.md` — ZWNJ (نیم‌فاصله), Persian numerals, formal-vs-conversational tone, translation traps, RTL constraints.

For translation/localization requests, read BOTH files: never map words 1:1 — localize to the target user's mental model.

## Workflow

1. **Identify the element type** (button? error? notification?) and the user's likely emotional state.
2. **Apply the matching pattern** below + the language rules file.
3. **Draft, then cut:** front-load the key information (inverted pyramid), remove every word without a job.
4. **Run the review checklist** at the bottom.

**Never invent product facts.** Help text stating why data is collected, an error's cause, what happens at expiry, what a deletion destroys — these must reflect real product behavior. If you don't know it, ask; a plausible-but-wrong consequence is worse than a vague one.

## Core Principles

Every piece of UX text must be:

1. **Purposeful** — helps the user or the business achieve a goal. No goal → delete the text.
2. **Concise** — fewest words without losing meaning. Users scan, they don't read; the first 2 words carry the message.
3. **Conversational** — natural and human, not robotic. Read it aloud: if you wouldn't say it, rewrite it.
4. **Clear** — plain language (7th-grade level for general products, 10th for professional). Jargon only when the audience uses it in speech themselves.

## UX Text Patterns

### Buttons and CTAs

- **Pattern:** `[Verb] [object]` — active imperative. "Save changes", "Delete account", "Place bid".
- Object may drop when context makes it unambiguous (icon button acting on the hovered item: "Archive").
- **Avoid:** "OK", "Submit", "Click here", "Yes"/"No" — the label must say what happens on tap.

### Links

- A link is a promise: **specific, sincere, substantial, succinct**. Name the destination: "See pricing", not "Learn more".
- Never rely on surrounding text — screen readers list links out of context.

### Error Messages

- **Pattern:** `[What happened]. [Why, if useful]. [What to do next].` Never a dead end.
- **Validation errors:** state the requirement, not the failure — "Password must be at least 8 characters".
- **System errors:** cause + recovery — "Couldn't save changes. Connection lost. Reconnect and try again."
- **Blocking errors:** blocker + resolution — "Subscription expired. Renew to restore access."
- **Avoid:** raw error codes, blame language ("invalid input"), vague causes, apologizing for user mistakes.

### Success Messages

- **Pattern:** `[What happened] [+ where it leads]`. Past tense, specific: "Receipt sent to your email."
- Proportional celebration — a saved setting gets a quiet confirmation, not confetti.

### Forms (labels, placeholders, help text)

- **Label:** short noun phrase naming the data — "Work phone number". No colon, no question form. Every field labeled — never use placeholder as label.
- **Placeholder:** a formatted example only — "+1 555 123 4567". Never instructions (it disappears on input and fails accessibility).
- **Help text:** why it's needed or the format rule, persistent below the field — "Used only for delivery updates."
- Mark **optional** fields ("Optional"), not required ones — most fields are required.

### Notifications (push / in-app)

- **Pattern:** title = the event, ≤ ~40 chars, key fact first ("Subscription expires in 3 days"); body = one sentence with the consequence or next action.
- Send only what the user gains from knowing NOW. No "We miss you" noise.

### Confirmation Dialogs (especially destructive)

- **Title:** a question naming action + object — "Delete this project?"
- **Body:** the consequence — what's lost, whether it's reversible: "All tasks, files, and comments will be permanently deleted."
- **Confirm button:** repeats the verb — "Delete project" (destructive styling). Never "Yes"/"OK".
- **Dismiss button:** "Cancel" — unless the flow itself cancels something; then the safe option is "Keep subscription"/"Go back" (see language rules).

### Empty States

- **Pattern:** what this space is for + CTA that fills it. "No messages yet. Start a conversation to connect with your team."
- Never a bare "No results" — for filtered-empty, suggest changing the filter.

### Tooltips

- Name the action or the meaning, nothing more: "Archive". Verb for action buttons, noun for status icons. 1–4 words; a tooltip needing a sentence is hiding a design problem.
- Never put essential information ONLY in a tooltip — touch devices can't hover.

### Onboarding / First-run

- **Headline:** the benefit ahead, not "Welcome to [Product]" — "Let's set up your first project."
- One action per screen. A welcome screen may preview the product's scope in the subtext, but still drives exactly one action. Explain a permission's user benefit before requesting it.

### Titles and Headings

- Front-load keywords — headings are pick-up lines for scanning eyes; the first 2 words decide whether the user reads on.
- Descriptive over clever. Good headings make the page scannable as layers ("layer-cake" scanning).

## Tone Adaptation

Voice stays constant; tone shifts with the user's state:

| User state | Tone |
|---|---|
| Frustrated (errors, failures) | Empathetic, solution-focused. No blame. |
| Confused (complex features) | Patient, step-by-step. |
| Confident (routine tasks) | Efficient, minimal. |
| Cautious (high-stakes actions) | Serious, transparent about consequences. |
| Successful (achievements) | Encouraging, proportional to the win. |
| New / first-time | Welcoming, benefit-first, zero assumed knowledge. |

## Consistency

- **One term per concept, product-wide.** If it's "project" on one screen, it's never "workspace" on the next. When editing existing text, match the product's established vocabulary before inventing your own.
- Same element type → same pattern everywhere (all destructive dialogs shaped alike).

## Accessibility & Benchmarks

- Label all interactive elements explicitly; write link text that stands alone.
- Sentences: 8–14 words (8 ≈ 100% comprehension, 14 ≈ 90%).
- Buttons/CTAs: 2–4 words. Titles: 3–6 words, ≤ 40 chars. Errors: 12–18 words including the fix.
- Budget for translation: text expands 20–30% in many languages (see Persian rules for RTL specifics).

## Review Checklist

Before delivering any string, verify:

- [ ] Purposeful, concise, conversational, clear (all four).
- [ ] Matches the element's pattern above.
- [ ] Language rules file applied (capitalization/ZWNJ, numerals, terminology traps).
- [ ] Key information in the first 2 words; no dead ends; next step always present.
- [ ] Terminology consistent with the rest of the product.
- [ ] Within length benchmarks; survives truncation.
