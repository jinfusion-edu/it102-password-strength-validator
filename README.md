# Password Strength Checker & Validator

IT102 · Introduction to Programming · Seattle Colleges

A real-time password strength meter and confirmation matcher, built with basic
loops and conditionals — no Regular Expressions.

## What it does

As you type, the page evaluates three rules (length ≥ 8, contains a digit,
contains a special character), marks each rule ✓ or ✗, and drives a coloured
strength bar. A second field confirms the password and reports whether the two
match.

## Files

| File | Role |
|---|---|
| `password-validator.html` | **The assignment deliverable.** Starter markup + the `<script>` block. |
| `index.html` | A one-line redirect, so the live URL works (Pages serves `index.html`). |

## How to run it

```bash
git clone https://github.com/jinfusion-edu/it102-password-strength-validator.git
cd it102-password-strength-validator
```

Open `password-validator.html` in a browser. No dependencies, no build step.

## Expected output

A **Create Account** card. Typing in the password field updates, on every
keystroke:

| Rules passed | Strength text | Bar colour | Bar width |
|---|---|---|---|
| field empty | `Empty` | — | 0% |
| 0–1 | `Weak` | red `#dc3545` | 33% |
| 2 | `Medium` | orange `#ffc107` | 66% |
| 3 | `Strong` | green `#28a745` | 100% |

The confirmation message appears only when both fields are filled, identical,
**and** the password is at least 8 characters. See the note below.

## Live URL

https://edu.jinfusion.dev/it102-password-strength-validator/

## AI collaboration — tool and prompt

**Tool used:** Claude (Anthropic), via Claude Code.

The assignment specifies the prompt:

> Act as a JavaScript developer. Write a function called "checkPasswordRules"
> that accepts a string parameter called "passwordString".
>
> The function should evaluate three specific rules and return an object with
> three boolean properties indicating whether the rule passed (true) or failed
> (false):
> 1. `hasMinLength`: true if the string is 8 or more characters long.
> 2. `hasNumber`: true if the string contains at least one digit character
>    ('0' through '9').
> 3. `hasSpecial`: true if the string contains at least one special character
>    from this list: !, @, #, $, %, ^, &, *
>
> Write this logic using standard loops, conditions (if), comparison operators,
> or basic string methods taught in introductory programming. Do not use
> advanced Regular Expressions (RegEx). Return only the JavaScript function code.

The event handling, rule visuals, strength meter and match logic were written by
hand, per Step 2.

### ⚠ One deliberate departure from the written spec

The assignment's Step 2 says to suppress the match message only when a field is
**empty**. Its own test table contradicts that: Test 1 (`P@ss` / `P@ss` —
identical, both non-empty) expects **no** match message, while Test 3
(`LetMeIn!!!` / `LetMeIn!!!` — also failing a rule) expects the message to
**show**.

The only rule consistent with all four supplied rows is gating on the
**8-character minimum**. That is what ships, so the professor's table passes.
The full reasoning, including a truth table of both readings, is in
`AUDIT-NOTES.md`.

## Verification

```bash
node ../../tools/run_tests.js
```

**All four of the professor's test rows pass**, plus rule-flag checks, boundary
checks at 7/8 characters, and a grep proving no RegEx is used anywhere. 17
assertions for this assignment. See `TEST-CASES.md`.
