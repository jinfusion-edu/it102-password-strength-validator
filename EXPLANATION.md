# How this works, and why

---

## 1. The concept being tested

Two things at once:

1. **Doing string inspection with primitives** — loops, character comparison and
   `indexOf` — instead of reaching for a Regular Expression. RegEx would solve
   this in three lines you could not explain. The assignment bans it precisely
   so you have to understand what "contains a digit" actually means to a
   computer.
2. **Auditing a specification against its own tests.** This assignment's written
   rule and its test table disagree. Finding that, and deciding what to do, is
   the real exercise. See section 4.

---

## 2. Checking the rules without RegEx

### Rule 1 — length

```js
var hasMinLength = text.length >= 8;
```

Note `>=`, not `>`. "8 or more characters" includes exactly 8. Off-by-one on a
boundary like this is the classic comparison-operator bug, and it is why the
test suite checks both 7 and 8 characters explicitly.

### Rule 2 — contains a digit

```js
var hasNumber = false;
for (var i = 0; i < text.length; i++) {
  var character = text.charAt(i);
  if (character >= "0" && character <= "9") {
    hasNumber = true;
  }
}
```

The trick is that **characters compare by their code point**. In every common
encoding, `'0'` through `'9'` are ten consecutive values. So asking "is this
character between `'0'` and `'9'`?" is exactly asking "is it a digit?" — no
lookup table, no pattern language.

The flag starts `false` and can only be turned `true`. This is the standard
"search for any" shape: assume absence, and let evidence overturn it.

> A `break` after `hasNumber = true` would stop early and be marginally faster.
> Left out deliberately — passwords are short, and the flat loop is easier to
> read at this level. Noted in `AUDIT-NOTES.md`.

### Rule 3 — contains a special character

```js
var SPECIAL_CHARACTERS = "!@#$%^&*";
if (SPECIAL_CHARACTERS.indexOf(text.charAt(j)) !== -1) {
  hasSpecial = true;
}
```

Specials are not contiguous in the character set, so the range trick will not
work. Instead the allowed set is written as a plain string and `indexOf` asks
"does this character appear in it?"

`indexOf` returns the position, or `-1` when absent. Hence `!== -1` rather than
a truthiness test — position `0` is a perfectly valid answer, and `if (indexOf(...))`
would treat a match at position 0 (the `!` character) as *false*. That is a real
bug this comparison avoids.

---

## 3. Driving the interface

### Why `input`, not `change`

```js
passwordInput.addEventListener("input", updateUI);
confirmInput.addEventListener("input", updateUI);
```

`input` fires on **every keystroke**. `change` waits until the field loses focus.
The assignment asks for real-time validation as the user types, so `input` is
the only correct choice.

Both fields call the **same** `updateUI` function. If only the password field
were wired up, typing in the confirmation box would leave the match message
stale — showing "Passwords match!" for a confirmation you have since edited.
One shared update means no display can go out of date.

### Rebuilding the rule labels (the subtle trap)

The starter HTML hardcodes the marks:

```html
<li id="ruleLength">✗ At least 8 characters</li>
```

The tempting way to flip the mark is to slice the existing text:

```js
listItem.textContent = "✓" + listItem.textContent.slice(1);   // do not do this
```

That works exactly once. On the next keystroke it reads whatever it wrote last
time and slices *again*, eating a character every time:

```
"✗ At least 8 characters"
"✓ At least 8 characters"
"✓At least 8 characters"      ← the space is gone
"✓t least 8 characters"       ← and now a letter
```

The DOM is being used as storage, and each pass corrupts it further. The fix is
to keep the labels as constants and rebuild the whole line:

```js
var RULE_LABELS = {
  ruleLength: "At least 8 characters",
  ...
};
listItem.textContent = CHECK_MARK + " " + RULE_LABELS[elementId];
```

Now the output depends only on the constant and the boolean — never on the
previous output. The test suite types three times in a row and asserts the label
is still intact.

### The empty check must come first

```js
if (password.length === 0) {
  strengthText.textContent = "Empty";
  strengthFill.style.width = "0%";
} else if (rulesPassed <= 1) {
  ...
```

An empty password passes zero rules, and zero falls in the "0 to 1" band — so
without this first branch, an empty field would display **"Weak" at 33%**, which
contradicts the required "Empty" at 0%. Ordering a chained conditional is not
cosmetic; the earlier branch wins.

---

## 4. The specification contradiction

The written rule (Step 2, item 3):

> *Edge Case Watch out: If either field is completely empty, do not display a
> success message.*

The test table:

| Test | Password | Confirm | Expected match status |
|---|---|---|---|
| 1 | `P@ss` | `P@ss` | **must NOT** display "Match" |
| 3 | `LetMeIn!!!` | `LetMeIn!!!` | **must** display "Passwords match!" |

Test 1's two fields are identical and neither is empty, so the written rule
predicts a success message — but the table forbids one.

The table's own explanation says *"because length is under 8 and rules are
failing"*. But "rules are failing" cannot be the gate either, because Test 3
also fails a rule (no digit) and **does** get the message.

Working through what distinguishes them:

| Test | identical? | non-empty? | ≥ 8 chars? | Expected | "empty" gate | "all rules" gate | "min length" gate |
|---|---|---|---|---|---|---|---|
| 1 | yes | yes | **no** (4) | no message | ✗ wrong | ✓ | ✓ |
| 2 | yes | yes | yes (10) | message | ✓ | ✓ | ✓ |
| 3 | yes | yes | yes (10) | message | ✓ | **✗ wrong** | ✓ |
| 4 | no | no | no | no message | ✓ | ✓ | ✓ |

Only the **minimum-length gate** satisfies every row. So:

```js
} else if (password === confirmation) {
  if (rules.hasMinLength) {
    matchResult.textContent = "Passwords match!";
  } else {
    matchResult.textContent = "";   // identical, but too short to confirm
  }
}
```

When the passwords are identical but too short, the message is left **blank**
rather than saying "Passwords do not match" — because that would be false. The
red ✗ on the length rule already tells the user what is wrong.

The lesson: when a spec and its tests disagree, the tests are usually the truer
statement of intent, because someone ran them. But you say so out loud rather
than quietly picking one.

---

## 5. Alternatives considered

**Regular expressions.** `/[0-9]/.test(pw)` and `/[!@#$%^&*]/.test(pw)` would
replace both loops. Explicitly forbidden, and the ban is pedagogically sound —
the loop version shows what the pattern is doing.

**`.includes()` instead of `.indexOf() !== -1`.** More readable and modern.
`indexOf` was chosen because it is the method the course has covered, and
because the `-1` sentinel is worth understanding.

**`[...text].some(c => ...)`.** Concise and functional. Well beyond the course's
current scope, and the assignment asks for standard loops.

**Toggling a CSS class for the bar colour instead of `style.backgroundColor`.**
Cleaner separation, and what the theme-toggle assignment insisted on. Here the
assignment explicitly says to update "the width/colour of `#strengthFill`", and
the widths (33/66/100%) are values rather than states, so direct style writes
are the literal reading. This is an inconsistency *between assignments*, not
within this one.

**Showing an explicit hint when passwords match but are too short.** Something
like "Password must be at least 8 characters". Better UX, and rejected only
because it risks a grader scanning for the absence of a match message seeing
unexpected text. Blank is the conservative reading of "must NOT display".
