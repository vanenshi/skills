---
name: ux-writing
description: Create user-centered, accessible interface copy (microcopy) for digital products including buttons, labels, error messages, notifications, forms, onboarding, and empty states. Make sure to use this skill whenever the user mentions writing or editing interface copy, UI strings, translations for apps/websites, designing conversational flows, auditing product content, or improving existing interface text, even if they don't explicitly ask for "UX writing"[cite: 1, 2].
---

# UX Writing

Write clear, concise, user-centered interface copy (UX text/microcopy) for digital products and experiences[cite: 2]. This skill provides frameworks, patterns, and best practices for creating text that helps users accomplish their goals[cite: 2].

## Progressive Disclosure & Localization

- **For General Requests:** Follow the core principles outlined in this file.
- **For Persian (Farsi) Requests:** You MUST read and strictly apply the rules in `references/persian-rules.md` before generating any text.

---

## Core UX Writing Principles

### The Four Quality Standards

Every piece of UX text should be:

1.  **Purposeful** — Helps users or the business achieve goals[cite: 2].
2.  **Concise** — Uses the fewest words possible without losing meaning[cite: 2].
3.  **Conversational** — Sounds natural and human, not robotic[cite: 2].
4.  **Clear** — Unambiguous, accurate, and easy to understand[cite: 2].

### Key Best Practices

- **Conciseness:** Use 40-60 characters per line maximum[cite: 2]. Every word must have a job[cite: 2]. Break dense text into scannable chunks and front-load important information[cite: 2].
- **Clarity:** Use plain language (7th-grade reading level for general, 10th for professional)[cite: 2]. Avoid jargon, idioms, and technical terms[cite: 2]. Choose meaningful, specific verbs[cite: 2].
- **Conversational Tone:** Write how you speak and use active voice 85% of the time[cite: 2]. Include prepositions and articles[cite: 2].
- **User-Centered:** Focus on user benefits, not features[cite: 2]. Anticipate and answer user questions[cite: 2].

---

## UX Text Patterns

Apply these common patterns for interface elements[cite: 2].

### Buttons and Links

- **Purpose:** Enable users to take action[cite: 2].
- **Format:** Active imperative verbs, sentence case[cite: 2].
- **Pattern:** `[Verb] [object]`[cite: 2].
- **Examples:** "Save changes", "Delete account", "View details"[cite: 2].
- **Avoid:** Generic labels like "OK", "Submit", "Click here"[cite: 2].

### Error Messages

- **Purpose:** Explain problem and provide solution[cite: 2].
- **Format:** Empathetic, clear, actionable[cite: 2].
- **Pattern:** `[What failed]. [Why/context]. [What to do].`[cite: 2].
- **Validation Errors:** Brief, specific guidance (e.g., "Email must include @")[cite: 2].
- **System Errors:** Explain backend failures (e.g., "Couldn't save changes. Connection lost. Reconnect and try again.")[cite: 2].
- **Blocking Errors:** Clear explanation of blocker and resolution (e.g., "Subscription expired. Your account is paused. Renew subscription to restore access.")[cite: 2].
- **What to Avoid:** Technical codes without explanation, blame language ("invalid input"), robotic tone, dead ends, and vague causes[cite: 2].

### Success Messages

- **Purpose:** Confirm action completion[cite: 2].
- **Format:** Past tense, specific, encouraging[cite: 2].
- **Pattern:** `[Action] [result/benefit]`[cite: 2].

### Empty States

- **Purpose:** Guide users when content is absent[cite: 2].
- **Format:** Explanation + CTA to populate[cite: 2].
- **Example:** "No messages yet. Start a conversation to connect with your team."[cite: 2].

---

## Tone Adaptation

While a brand's voice remains constant, tone shifts based on user context and emotional state[cite: 2].

- **Frustrated (errors, failures):** Empathetic and solution-focused[cite: 2]. Acknowledge the problem without blame[cite: 2].
- **Confused (first use, complex features):** Patient and explanatory[cite: 2]. Break down steps clearly[cite: 2].
- **Confident (routine tasks):** Efficient and direct[cite: 2]. Minimal explanation[cite: 2].
- **Cautious (high-stakes actions):** Serious and transparent[cite: 2]. Clear consequences[cite: 2].
- **Successful (achievements):** Positive and encouraging[cite: 2]. Proportional to achievement[cite: 2].

---

## Accessibility & Benchmarks

- **Screen Reader Optimization:** Label all interactive elements explicitly and write descriptive link text[cite: 2].
- **Cognitive Accessibility:** Target 8-14 words per sentence (8 words = 100% comprehension, 14 words = 90%)[cite: 2].
- **Sentence Length Targets:**
  - Buttons/CTAs: 2-4 words ideal, 6-word maximum[cite: 2].
  - Titles: 3-6 words, 40 characters maximum[cite: 2].
  - Error messages: 12-18 words (including solution)[cite: 2].
