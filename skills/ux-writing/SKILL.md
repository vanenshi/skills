---
name: ux-writing
description: >-
  Create user-centered, accessible interface copy (microcopy) for digital products including buttons, labels, error messages, notifications, forms, onboarding, and empty states. Make sure to use this skill whenever the user mentions writing or editing interface copy, UI strings, translations for apps/websites, designing conversational flows, auditing product content, or improving existing interface text, even if they don't explicitly ask for "UX writing".
---

# UX Writing

Write clear, concise, user-centered interface copy (UX text/microcopy) for digital products and experiences. This skill provides frameworks, patterns, and best practices for creating text that helps users accomplish their goals.

## Progressive Disclosure & Localization

- **For General Requests:** Follow the core principles outlined in this file.
- **For Persian (Farsi) Requests:** You MUST read and strictly apply the rules in `references/persian-rules.md` before generating any text.

---

## Core UX Writing Principles

### The Four Quality Standards

Every piece of UX text should be:

1.  **Purposeful** — Helps users or the business achieve goals.
2.  **Concise** — Uses the fewest words possible without losing meaning.
3.  **Conversational** — Sounds natural and human, not robotic.
4.  **Clear** — Unambiguous, accurate, and easy to understand.

### Key Best Practices

- **Conciseness:** Use 40-60 characters per line maximum. Every word must have a job. Break dense text into scannable chunks and front-load important information.
- **Clarity:** Use plain language (7th-grade reading level for general, 10th for professional). Avoid jargon, idioms, and technical terms. Choose meaningful, specific verbs.
- **Conversational Tone:** Write how you speak and use active voice 85% of the time. Include prepositions and articles.
- **User-Centered:** Focus on user benefits, not features. Anticipate and answer user questions.

---

## UX Text Patterns

Apply these common patterns for interface elements.

### Buttons and Links

- **Purpose:** Enable users to take action.
- **Format:** Active imperative verbs, sentence case.
- **Pattern:** `[Verb] [object]`.
- **Examples:** "Save changes", "Delete account", "View details".
- **Avoid:** Generic labels like "OK", "Submit", "Click here".

### Error Messages

- **Purpose:** Explain problem and provide solution.
- **Format:** Empathetic, clear, actionable.
- **Pattern:** `[What failed]. [Why/context]. [What to do].`.
- **Validation Errors:** Brief, specific guidance (e.g., "Email must include @").
- **System Errors:** Explain backend failures (e.g., "Couldn't save changes. Connection lost. Reconnect and try again.").
- **Blocking Errors:** Clear explanation of blocker and resolution (e.g., "Subscription expired. Your account is paused. Renew subscription to restore access.").
- **What to Avoid:** Technical codes without explanation, blame language ("invalid input"), robotic tone, dead ends, and vague causes.

### Success Messages

- **Purpose:** Confirm action completion.
- **Format:** Past tense, specific, encouraging.
- **Pattern:** `[Action] [result/benefit]`.

### Empty States

- **Purpose:** Guide users when content is absent.
- **Format:** Explanation + CTA to populate.
- **Example:** "No messages yet. Start a conversation to connect with your team.".

---

## Tone Adaptation

While a brand's voice remains constant, tone shifts based on user context and emotional state.

- **Frustrated (errors, failures):** Empathetic and solution-focused. Acknowledge the problem without blame.
- **Confused (first use, complex features):** Patient and explanatory. Break down steps clearly.
- **Confident (routine tasks):** Efficient and direct. Minimal explanation.
- **Cautious (high-stakes actions):** Serious and transparent. Clear consequences.
- **Successful (achievements):** Positive and encouraging. Proportional to achievement.

---

## Accessibility & Benchmarks

- **Screen Reader Optimization:** Label all interactive elements explicitly and write descriptive link text.
- **Cognitive Accessibility:** Target 8-14 words per sentence (8 words = 100% comprehension, 14 words = 90%).
- **Sentence Length Targets:**
  - Buttons/CTAs: 2-4 words ideal, 6-word maximum.
  - Titles: 3-6 words, 40 characters maximum.
  - Error messages: 12-18 words (including solution).
