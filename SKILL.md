---
name: recurring-issues-ai-skill
description: "Recognize that a defect the user reports has come up BEFORE and give it a PERMANENT fix instead of another patch. Use whenever the user reports a bug, visual glitch, misalignment, broken behavior or 'it's doing X again' in any software project — and ALWAYS when the user's message says 'again', 'still', 'this isn't the first time', 'keeps happening', 'I already asked', 'we talked about this', 'find a permanent fix', or when a search of the repo's RECURRING.md / CLAUDE.md / git log finds the same symptom. Holds the history check (where past reports live), the four-part permanent-fix checklist (one producer, full sweep, an automatic guard that fails on regression, a rule with its trigger), the RECURRING.md ledger format, and the report line. Also use after fixing ANY defect, to append the ledger line that lets the next session find it."
---

# Recurring issues: second mention → permanent fix

The standard: when the same issue is mentioned **two or more times**, a patch
is no longer acceptable — the fix must make the class impossible to reintroduce silently.
Sessions cannot see each other, so a repeat ALWAYS looks new unless you check history first.
A keycap misalignment can reach a third report before anyone says "this isn't the first
time", while the cause (⌘⇧ glyphs missing from the app's fonts, six separate keycap styles) has
been there all along.

## 1. Detect — before touching code for any reported defect

**Instant repeat (no search needed):** the user's message says *again, still, this isn't the first
time, keeps happening, I already asked, we talked about this, same issue, permanent fix*.

**Otherwise search, in this order** (≈30 seconds; pick 2–4 symptom words — the thing, not the
screen, plus the UI's own label for the state when it has one (`not shown in list`): `keycap|shortcut glyph|kbd`, `truncat|ellips`, `focus|keyboard dead`):

```bash
grep -niE '<words>' RECURRING.md 2>/dev/null           # the ledger — first stop
grep -niE '<words>' CLAUDE.md                           # project gotchas / standing rules
git log --oneline -i --grep='<word>' | head -20         # past fix commits
```

When the project keeps issue notes (an issues folder), grep those
too — `grep -rliE '<words>' "<issues folder>"` — and the session transcripts as
a last resort (`grep -l -iE '<words>' ~/.claude/projects/<slug>/*.jsonl`).

**Discrimination**

| Counts as a repeat | Does not |
|---|---|
| Same SYMPTOM, any surface or cause (keycaps misaligned in Settings, then the strip) | A different symptom in the same component |
| An earlier fix that addressed one instance | A new feature request that touches the same area |
| The user's own words calling it a repeat — even with no ledger hit | |
| The same CLASS in a new flow: an earlier fix covered one path (a restored selection the view no longer held) and the report arrives through another (an account filter change left an excluded email in the pane) | A look-alike whose cause is a different class (a slow fetch is not a stale selection) |

| A WRITTEN RULE already covered the class but named it too narrowly (a skill said "never put a margin on a LINE"; the `---` rule WIDGET carried the margin and made the cursor jump) | |

When the signals can't be read, **treat it as a repeat**.

**When a rule already existed and the defect still shipped, widen the rule's WORDING to the
whole class** (lines → everything in the flow; "content text" → every content size in every
mode) in the same fix — the narrow wording is the second producer.

## 2. The permanent fix — all four, or it isn't permanent

1. **Root cause at the ONE producer.** Find the single place the wrong thing is made (a
   formatter, a style, a query, a helper) and fix it there — or route every producer through
   one new component / function. Patching the screen in the screenshot is the failure mode.
2. **Sweep every instance.** `grep` the whole codebase for the pattern; migrate all of them
   in the same change, not only the one reported. **Include the TWINS**: when a flow
   has two paths (new item vs existing; web vs native; edit vs preview mode),
   the same defect or feature almost always exists on the other path — gaps usually
   come from fixing one path only. Name the twin in the fix and give it the same test.
3. **An automatic guard that FAILS on regression** — a unit test, a source-scanning test, a
   lint rule, a build-script check, or a hook. A note alone is not a guard. Pattern that
   worked: a test that scans `src/**/*.tsx` for the banned
   shape (raw `<kbd`, a chord printed as JSX text) and lists every offender with file:line.
   Another pattern: a test that scans source
   for banned lifecycle listeners (e.g. `addEventListener("blur"` or `.onblur` on inline rename
   elements) to prevent fragile synthetic commit triggers. Make the guard's failure message say
   what to use instead.
4. **Written down with its trigger** — the project CLAUDE.md entry says what the class is,
   how a future session detects it (a grep, a measurement), and where the guard lives.

Verify with a **measurement**, not a look: geometry (`getBoundingClientRect` centers equal),
a failing-then-passing test, a count that went to zero.

**A fix that did not hold means your model of the bug is wrong: reproduce it where the user sees it
before writing another fix.** *Trigger:* a repeat report right after a fix you had
"verified". *Discrimination:* an email app's forward To-field cursor got three fixes in one day (Tab order, an
inert frame, an activation repair), each passing a headless probe that could not show the symptom at
all (an inactive window draws no caret); a log of focus events could not either. *Action:* stop
theorizing; reproduce the exact symptom yourself in the user's real environment (the installed app, the
active window; ask in chat when that needs the user's screen) and confirm the reproduction is valid; then
bisect with ONE diagnostic build whose suspects are runtime switches, and ship only the change that
flips the measured symptom. Delete the diagnostics and any logging that did not help afterwards.

## 3. Report

First line of the reply: **`🔁 Second report of <symptom> — permanent fix`** (third, fourth…
as counted). Then, briefly: why it kept recurring (the real cause), what now makes it
impossible (the producer + the guard), and the measurement.

## 4. Ledger — after EVERY defect fix, repeat or not

`RECURRING.md` at the repo root (committed, so every worktree sees it). Create it on the first
fix. One line per defect, newest at the top of its section; update the count when a known
symptom returns:

```
MM-DD-YY · <symptom keywords, the words the user would use> · <Nth> report · cause: <one phrase> · fix: <what> · guard: <test / check or "none">
```

- Symptom words are for FUTURE GREPS — write the words the user uses ("misaligned", "cut off",
  "nothing happens"), plus the component name.
- A first-time fix still gets a line (`1st report`, `guard: none` is honest) — that line is
  what turns the next mention into a detected repeat.
- A line with `guard: none` whose count reaches 2 is the ledger telling you step 2 is due.

