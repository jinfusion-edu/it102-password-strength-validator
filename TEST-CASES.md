# Test cases

The assignment supplies a four-row verification table (Step 3) and asks for test
cases in the video. The professor's four are reproduced first; further cases
were authored.

> All rows were **executed** by `node ../../tools/run_tests.js`, which loads the
> real `<script>` block from `password-validator.html`, fires the actual `input`
> handler, and reads the actual `#strengthText`, `#strengthFill` and
> `#matchResult` elements. The "Actual" column is harness output.

## Step 3 — the professor's verification sandbox

| Test | Inputs | Expected Output Behavior | Actual Output & UI Status | Pass/Fail |
|---|---|---|---|---|
| **Test 1** | Password `P@ss`<br>Confirm `P@ss` | Rules met: special only.<br>Strength: Weak.<br>Match: should **NOT** display "Match". | Rules `{len:false, num:false, special:true}`<br>Strength `Weak`, width `33%`<br>Match: `""` (blank) | ✅ **PASS** |
| **Test 2** | Password `Secret123!`<br>Confirm `Secret123!` | Rules met: all 3.<br>Strength: Strong.<br>Match: "Passwords match!" in green. | Rules `{len:true, num:true, special:true}`<br>Strength `Strong`, width `100%`<br>Match: `"Passwords match!"` + `.text-success` | ✅ **PASS** |
| **Test 3** | Password `LetMeIn!!!`<br>Confirm `LetMeIn!!!` | Rules met: length and special.<br>Strength: Medium (missing a number).<br>Match: "Passwords match!" in green. | Rules `{len:true, num:false, special:true}`<br>Strength `Medium`, width `66%`<br>Match: `"Passwords match!"` | ✅ **PASS** |
| **Test 4** | Password `Admin`<br>Confirm *(leave empty)* | Rules met: none.<br>Strength: Weak.<br>Match: error or blank (must not match). | Rules `{len:false, num:false, special:false}`<br>Strength `Weak`, width `33%`<br>Match: `""` (blank) | ✅ **PASS** |

> **Note on Test 1 vs Test 3.** These two rows are why the implementation gates
> the match message on the 8-character minimum rather than on emptiness alone.
> Both pairs are identical and non-empty; both fail at least one rule; only the
> short one suppresses the message. `AUDIT-NOTES.md` sets out the full truth
> table and the risk in choosing this reading.

## Additional executed cases

### Direct function calls

| Call | Result | Status |
|---|---|---|
| `checkPasswordRules("P@ss")` | `{hasMinLength:false, hasNumber:false, hasSpecial:true}` | ✅ PASS |
| `checkPasswordRules("Secret123!")` | `{true, true, true}` | ✅ PASS |
| `checkPasswordRules("LetMeIn!!!")` | `{true, false, true}` | ✅ PASS |
| `checkPasswordRules("")` | `{false, false, false}` | ✅ PASS |

### Edge cases

| # | Case | Expected | Actual | Status |
|---|---|---|---|---|
| E1 | Both fields empty | `Empty` at width `0%`, no match message | `Empty`, `0%`, `""` | ✅ PASS |
| E2 | Genuine mismatch — `Secret123!` vs `Secret124!` | Explicit failure message | `"Passwords do not match."` | ✅ PASS |
| E3 | Boundary — exactly 8 chars (`abcdefgh`) | `hasMinLength` true | `true` | ✅ PASS |
| E4 | Boundary — 7 chars (`abcdefg`) | `hasMinLength` false | `false` | ✅ PASS |
| E5 | Label erosion — type three times in succession | Rule label text intact | `"✓ At least 8 characters"` | ✅ PASS |
| E6 | No RegEx anywhere in the source | grep finds nothing | no matches | ✅ PASS |

**E1** matters because an empty password passes zero rules, and zero falls in the
"0 to 1 → Weak, 33%" band. Only an ordered conditional produces `Empty` at `0%`.

**E3/E4** exist because "8 or more" is a `>=` boundary — the single most common
place for an off-by-one.

**E5** is the trap described in `EXPLANATION.md`: flipping the ✓/✗ mark by
slicing the element's existing text works once and then eats a character on every
subsequent keystroke. Typing repeatedly and asserting the label is still intact
is the only way to catch it.

**E6** is a requirement, not a nicety — the assignment forbids Regular
Expressions. The grep covers `new RegExp`, `.test(`, `.match(` and regex
literals, excluding the `<style>` block.

## Video walkthrough — 6 cases to demonstrate

Suggested set, mapping to the above:

**Normal:** Test 2 (Strong, all rules, match) · Test 3 (Medium, missing a
number, still matches) · a live typing demo showing rules flip ✓ one at a time.

**Edge:** Test 1 (identical but too short — no match message) · Test 4 (confirm
left empty) · E2 (genuine mismatch shows red).

## Explicitly NOT tested

- Any visual rendering — no browser was opened. The harness confirms
  `style.width` and `textContent` were **set**, not that the bar animates or the
  colours appear.
- The `transition: width 0.3s` animation
- Screen-reader announcement of strength changes (**expected to be absent** —
  there is no `aria-live` region; see `AUDIT-NOTES.md`)
- Unicode/emoji passwords (**expected to miscount** — `.length` counts UTF-16
  code units)
- Paste-to-field and programmatic `.value` assignment (the latter does not fire
  `input`)
