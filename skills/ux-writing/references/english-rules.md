# English UX Writing Rules

When generating or editing UX text in English, apply these mechanics, voice rules, and terminology choices. They resolve the decisions the core patterns leave open (capitalization, contractions, standard verbs).

## 1. Mechanics

- **Sentence case everywhere:** buttons, titles, labels, menu items, notifications. `Save changes` (Correct) | `Save Changes` (Incorrect). Only deviate if the product's brand style guide mandates title case.
- **Contractions:** Use them — `you'll`, `can't`, `it's`. Spelled-out forms (`cannot`, `do not`) read as legal/robotic; reserve them for high-stakes warnings where extra weight is intentional.
- **Numerals, not words:** `3 days left` (Correct) | `three days left` (Incorrect). Numerals stop the scanning eye.
- **No ALL CAPS** except established acronyms (PDF, URL). Caps read as shouting and slow reading.
- **Exclamation marks:** At most one per screen, only for genuine user achievements. Never in errors or warnings.
- **Ampersands:** Avoid in running text; acceptable in space-constrained labels (`Terms & privacy`).

## 2. Voice and Address

- **Second person for the user, "we" for the product:** `We'll email your receipt` / `Your changes are saved`. The product never says "I".
- **"Your", not "my":** Interface speaks TO the user. `Your projects` (Correct) | `My projects` (only acceptable in user-authored contexts like checkbox agreements: "I agree to the terms").
- **"Please":** Use only when asking the user to do burdensome or repeat work (`Please try again in a few minutes`). Routine instructions don't need it — `Enter your email`, not `Please enter your email`.
- **"Sorry":** Apologize once, only for real product failures (data loss, outage). Never for validation errors or things the user controls. Never `Oops!`/`Whoops!` for anything with consequences.

## 3. Terminology Traps

Standard English UI words carry precise, conventional meanings. Never blur them.

### 1. The "Submit" Trap

- **Problem:** `Submit` describes the mechanism, not the outcome.
- _Order flow:_ `Place order` / `Pay $24.99`
- _Messaging:_ `Send message`
- _Account creation:_ `Create account`
- _Feedback form:_ `Send feedback`

### 2. The "OK / Confirm" Trap

- **Problem:** Generic confirmation forces the user to re-read the dialog to know what they're agreeing to.
- _Destructive dialog:_ Button repeats the verb — `Delete project`, never `Yes`/`OK`.
- _Permission dialog:_ `Allow access` / `Don't allow`.
- _Pure acknowledgment (info only, no choice):_ `Got it`.

### 3. The "Cancel" Trap

- **Problem:** `Cancel` means both "abandon this flow" and "revoke that thing" — lethal ambiguity when the flow IS a cancellation.
- _Form / wizard exit:_ `Cancel` (abandons unsaved work; pair with a warning if data is lost).
- _Dismiss informational UI:_ `Close` or the ✕ icon.
- _Revoking an active item:_ `Cancel subscription` / `Cancel order` — and the safe option next to it must NOT be bare `Cancel`; use `Keep subscription` or `Go back`.

### 4. The "Delete" vs "Remove" Trap

- **Problem:** Users learn the difference; violating it destroys trust in destructive actions.
- _Data destroyed permanently:_ `Delete` (`Delete account`, `Delete file`) — high friction, destructive styling.
- _Item leaves a list but data survives:_ `Remove` (`Remove from cart`, `Remove member from team`).

### 5. The "Sign in / Log in" Trap

- **Problem:** Mixing pairs (`Sign in` on one screen, `Login` on another) makes users doubt they're in the same product.
- Pick ONE pair product-wide: `Sign in` + `Create account`, or `Log in` + `Sign up`.
- `Login`/`Signup` (one word) are nouns/adjectives only (`login page`); the verb is always two words (`log in`).

### 6. The "Learn More" Trap

- **Problem:** `Learn more`, `Click here`, `Get started` are broken promises — no information scent. Links must be specific, sincere, substantial, succinct.
- _Instead:_ Name the destination — `See pricing`, `How billing works`, `View security settings`.
- Front-load the keyword: scanning eyes read the first 2 words.

### 7. The "Enable / Disable" Trap

- **Problem:** System vocabulary, not human vocabulary.
- _Settings toggles:_ `Turn on notifications` / `Turn off sound`.
- `Enable` is acceptable only in technical products whose users say it themselves (dev tools, admin consoles).

### 8. The "Invalid Input" Trap

- **Problem:** `Invalid`, `illegal`, `forbidden`, `prohibited`, `bad request` blame the user in machine language.
- _Instead, state the requirement:_ `Enter an email address with an @` / `Password must be at least 8 characters`.
- `Something went wrong` is allowed only when the cause is genuinely unknown — and always with a next step.

## 4. Plain-Word Swaps

| Avoid | Use |
|---|---|
| utilize | use |
| in order to | to |
| assist | help |
| request | ask |
| additional | more |
| receive | get |
| purchase | buy |
| prior to | before |
| at this time | now |
| terminate | end |
| modify | change |
| select | choose (or pick) |

## 5. Dates, Times, Truncation

- Unambiguous dates: `Mar 3, 2026`, never `03/04` (month/day vs day/month varies by country).
- Relative time for recency (`2 min ago`), absolute for records (`Mar 3, 10:24 AM`) — include time zone when users collaborate across them.
- Design for truncation: front-load the differentiating word so `Quarterly report — final…` survives where `Final version of the quarterly…` doesn't.
