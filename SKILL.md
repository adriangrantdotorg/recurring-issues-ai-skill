---
name: recurring-issues-ai-skill
description: "Recognize that a defect the user reports has come up BEFORE and give it a PERMANENT fix instead of another patch. Use whenever the user reports a bug, visual glitch, misalignment, broken behavior or 'it's doing X again' in any software project — and ALWAYS when the user's message says 'again', 'still', 'this isn't the first time', 'keeps happening', 'I already asked', 'we talked about this', 'find a permanent fix', or when a search of the repo's RECURRING.md / CLAUDE.md / git log finds the same symptom. Holds the history check (where past reports live), the four-part permanent-fix checklist (one producer, full sweep, an automatic guard that fails on regression, a rule with its trigger), the RECURRING.md ledger format, and the report line. Also use after fixing ANY defect, to append the ledger line that lets the next session find it."
---

# Recurring issues: second mention → permanent fix

The standard: when the same issue is mentioned **two or more times**, a patch
is no longer acceptable — the fix must make the class impossible to reintroduce silently.
Sessions cannot see each other, so a repeat ALWAYS looks new unless you check history first.
In one app, a keycap misalignment reached a third report before the user had to say "this isn't the first
time"; the cause (⌘⇧ glyphs missing from the app's fonts, six separate keycap styles) had
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

When the project keeps issue notes (for example a folder of bug-report notes), grep those
too — `grep -rliE '<words>' "<notes folder>"` — and the session transcripts as
a last resort (`grep -l -iE '<words>' ~/.claude/projects/<slug>/*.jsonl`).

**Discrimination**

| Counts as a repeat | Does not |
|---|---|
| Same SYMPTOM, any surface or cause (keycaps misaligned in Settings, then the strip) | A different symptom in the same component |
| An earlier fix that addressed one instance | A new feature request that touches the same area |
| The user's own words calling it a repeat — even with no ledger hit | |
| The same CLASS in a new flow: an earlier fix covered one path (e.g. a restored selection the view no longer held) and the report arrives through another (e.g. an account filter change left an excluded email in the pane) | A look-alike whose cause is a different class (a slow fetch is not a stale selection) |

| A WRITTEN RULE already covered the class but named it too narrowly (e.g. a skill said "never put a margin on a LINE"; the `---` rule WIDGET carried the margin and made the cursor jump) | |
| The first fix went into ONE instance and a skill NOTE, never into the TOOL that does the action (e.g. one script was fixed and a note written, while the shared helper and the other entry points kept doing it the old way). A note is not the producer: fix the tool every path goes through, and add a hook or check that blocks the old way | |
| The fix SWAPPED the thing the user objected to for a look-alike (e.g. design pages: page screenshots → live scaled frames of the pages → a switcher strip above the page; "I'm still seeing screenshots", then "never include these top frames"). To the user it is the same thing whatever draws it. Fix = remove the whole category (the bare page, nothing added), never a new kind of wrapper | |

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
   has two paths (new item vs existing one; web vs native; edit vs preview mode),
   the same defect or feature almost always exists on the other path — three gaps in one day
   came from fixing one path only. Name the twin in the fix and give it the same test.
3. **An automatic guard that FAILS on regression** — a unit test, a source-scanning test, a
   lint rule, a build-script check, or a hook. A note alone is not a guard. Pattern that
   worked (a `keycap.guard.test.ts`): a test that scans `src/**/*.tsx` for the banned
   shape (raw `<kbd`, a chord printed as JSX text) and lists every offender with file:line.
   Another pattern: a test that scans source
   for banned lifecycle listeners (e.g. `addEventListener("blur"` or `.onblur` on inline rename
   elements) to prevent fragile synthetic commit triggers. Make the guard's failure message say
   what to use instead.
   **A word scan reads string escapes as breaks.** *Trigger:* a guard or codemod finds a word
   in source with a letter-boundary check (`(?<![A-Za-z])Name`). *Discrimination:* in source text
   `\nAcme Ventures` (a firm's name after a line break) the letter before the name is the `n` of `\n`, so a
   37-file rename skipped an email
   sign-off and the guard written the same way would have passed it. *Action:* replace `\\[ntr]` with a space
   before matching; strip a trailing comment only after whitespace (`(^|\s)//`), so a URL's `//` doesn't hide the
   rest of the line; then drop a probe file holding the real missed line and watch the guard fail before you trust it.
   **Swift app with no Xcode project or Package.swift:** put
   the guard in `tests/<Name>/main.swift` (top-level code compiles only in a file named
   `main.swift` when several files are passed to `swiftc`), compile it with the SMALLEST set of
   `Sources/*.swift` it needs, and run it from `build.sh` BEFORE the script deletes the
   installed app, so a failure leaves the working app in place. Keep window-showing code out of
   that set: a test binary should never order a
   window in, so split a view class into its own file. Prove the guard bites by compiling it
   once against the old file (`git show HEAD:Sources/X.swift > <scratch>/X.swift`): it must
   fail there and pass on the fix. Best of all is a MEASURED symptom (lay out with
   `NSLayoutManager` and compare rects), not only a source shape.
4. **Written down with its trigger** — the project CLAUDE.md entry says what the class is,
   how a future session detects it (a grep, a measurement), and where the guard lives.

Verify with a **measurement**, not a look: geometry (`getBoundingClientRect` centers equal),
a failing-then-passing test, a count that went to zero.

**A decision inherited from an older repo is unguarded when its test can't run in the new one.**
*Trigger:* the repo was copied or grown from another (a shared engine folder taken from an earlier app), its `RECURRING.md`
carries the old repo's lines, and you are about to apply a general default (an accessibility norm, a library's
recommended setting) or fix a symptom. *Discrimination:* the older app had decided to ignore the system's Reduce Motion setting; the line sat in the new repo's ledger and its only test sat in a browser suite that needs the older app's data and had never run in the new repo. A session fixed a "not glowing" status dot by obeying Reduce Motion, from habit, and undid a decision that had already been made. *Action:* (1) grep the ledger for the topic BEFORE choosing the fix, even when
the fix looks like a standard default; (2) for each old-repo decision whose guard cannot run here, add a guard that
does (a source-scan unit test runs with no data) and list the rest in the project CLAUDE.md, read
every session; (3) name the old suite that doesn't run as work to port.

**A look guard that reads an empty page is not a guard.** *Trigger:* a browser test checks how
pages look (fonts, contrast, cards) on a made-up test copy of the data. *Discrimination:* the test data had no rows for several pages, so "small labels are never in the heading font" passed on an empty page; the
first real rows showed names in a tiny heading font, a wrong column header and an empty picture slot.
*Action:* give the test data at least a few rows for every page the look tests visit, and add rows whenever a page
turns out empty; the same data serves the review copy the user looks at.

**A fix whose test can't fail the way production fails is not a guard.** *Trigger:* a fix
depends on an outside resource being there later: a file in a scratchpad or worktree, a cache, a server, a page path
saved in a record. *Discrimination:* a "pictures in the review" fix was tested against a fixture page that
never disappears, while 44 of 57 real pages were deleted with their scratchpads and worktrees before the weekend; the
test passed and the review went back to text. *Action:* the guard deletes (or breaks) the resource the way production
does, then checks the result still holds; when the resource is known to vanish, copy what you need at the moment it
exists (for example, keep pages and their pictures in a committed `design/kept/` folder), and make the fallback LOUD (a red line on the
card), never a quiet degrade.

**A test fed a message shape you INVENTED is not a guard.** *Trigger:* the code under test
reads data another program produces (a daemon's WebSocket or HTTP JSON, a CLI's `--json`, a file another app
writes), and the test builds that data by hand. *Discrimination:* the tests sent `"device": ["id": "x"]` while
the real daemon sends `"device": "Mac Mini 2024"`; every test passed, and in production every message read as
"paused", so Repeat Off took a route that did nothing. A shape copied from the producer's docs is better than a
guess and still second best. *Action:* capture one real sample from the running producer (`curl` its state route,
run the CLI) and use it as the fixture, or at least one fixture in that exact shape; name where it came from in a
comment; prove the guard fails on the old code before calling the fix done.
The same holds for a LAYOUT you never saw drawn: a test pinned the new
Box shape exactly, passed, and pinned the very shape that overlapped on desktop; the harness has no pixels. When the
test can't see the screen, the guard encodes a shape a sibling element ALREADY draws right in the user's screenshots (name
it in the test's comment), never your new theory. Check the sibling has the same NEIGHBOURS too: a second
fix copied the header's − ("a Button direct in its row draws right"), but the header had nothing above it and also
overflowed, hidden by the band's scroll; the third report showed the real rule (lines of Buttons need a blank line
between). Before copying, read the screenshot for the sibling's edges, not only its middle.

**A rule about what the AI's replies SAY: delete the template, and guard where running sessions are reached.** *Trigger:* the repeat is a line or phrase in Claude's chat replies that an instruction file asked for (a
template sentence in CLAUDE.md, an output style, a skill). *Discrimination:* the first fix added an exception under the
template ("skip it when the reply already lists the files"); a session that had loaded the file earlier never saw the
exception, and even with it a ready-made sentence gets pasted by habit. Instruction files and the hook LIST load at
session start; a registered hook SCRIPT is read fresh on every run. *Action:* (1) remove the template outright instead
of adding a condition; (2) put the check inside an already-registered Stop hook script, with the real flagged lines as
fixtures and a quoted mention of the line allowed, so it bites in running sessions too; (3) log it in
`~/.claude/RECURRING.md`, the ledger for the AI's own replies and rules.

**A second report the SAME day means the first fix covered instances, not the class: load-test it the way
production loads it.** *Trigger:* you fixed a hang, freeze or race by moving the
named callers (two bridge calls off the main thread), and the same symptom comes back. *Discrimination:* the first fix
held for the calls it moved, but the cause was a shared resource (the dispatch pool) every caller drew on; a name-list
source scan missed a third caller reached through a shared "sync all" entry point. A clean normal run proves nothing for a
load-dependent bug. *Action:* reproduce under the real load (a burst of requests while the store change reloads every
window), `sample` the stuck process and COUNT threads (100+ waiting = pool starvation, not one bad call), fix the
shared mechanism, and add a runtime guard that recreates the load; keep any source scan only as a second guard.

**A fix that did not hold means your model of the bug is wrong: reproduce it where the USER sees it
before writing another fix.** *Trigger:* a repeat report right after a fix you had
"verified". *Discrimination:* the forward To-field cursor got three fixes in one day (Tab order, an
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

**A gap the fix leaves open carries its fix in the same breath.** *Trigger:* your report names
something still wrong after the fix ("one gap remains", "still possible", "except when …"). *Discrimination:* a fix report that ends "⚠️ One gap remains: … the card can still get stuck" with no fix leaves the user to ask what the fix is; a problem stated should come with a solution, briefly.
*Action:* small and safe → fix it before replying and report it as fixed; otherwise one line `❌ Not fixed: <what goes
wrong>`, then your fix in one or two lines, then the question (both options
named, a short table, answer words = column names). A fix that needs the user's OK first (a background watcher, a privacy
setting) is still named, never left out.

## 4. Ledger — after EVERY defect fix, repeat or not

`RECURRING.md` at the repo root (committed, so every worktree sees it). Create it on the first
fix. A defect in how the AI's replies read or how its rules behave has no project repo: its ledger is
`~/.claude/RECURRING.md`. One line per defect, newest at the top of its section; update the count when a known
symptom returns:

```
MM-DD-YY · <symptom keywords, the words the user would use> · <Nth> report · cause: <one phrase> · fix: <what> · guard: <test / check or "none">
```

- Symptom words are for FUTURE GREPS — write the words the user uses ("misaligned", "cut off",
  "nothing happens"), plus the component name.
- A first-time fix still gets a line (`1st report`, `guard: none` is honest) — that line is
  what turns the next mention into a detected repeat.
- A line with `guard: none` whose count reaches 2 is the ledger telling you step 2 is due.

