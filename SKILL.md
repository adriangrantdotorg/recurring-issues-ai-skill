---
name: recurring-issues-ai-skill
description: "Recognize that a defect the user reports has come up BEFORE and give it a PERMANENT fix instead of another patch. Use whenever the user reports a bug, visual glitch, misalignment, broken behavior or 'it's doing X again' in any software project — and ALWAYS when the user's message says 'again', 'still', 'this isn't the first time', 'keeps happening', 'I already asked', 'we talked about this', 'find a permanent fix', or when a search of the repo's RECURRING.md / CLAUDE.md / git log finds the same symptom. Holds the history check (where past reports live), the four-part permanent-fix checklist (one producer, full sweep, an automatic guard that fails on regression, a rule with its trigger), the RECURRING.md ledger format, and the report line. Also use after fixing ANY defect, to append the ledger line that lets the next session find it."
---

# Recurring issues: second mention → permanent fix

The standard: when the same issue is mentioned **two or more times**, a patch
is no longer acceptable — the fix must make the class impossible to reintroduce silently.
Sessions cannot see each other, so a repeat ALWAYS looks new unless you check history first.
In one case a keycap misalignment reached a third report before the user had to say "this isn't the first
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

When the project keeps issue notes (a notes folder or exported issue tracker), grep those
too — `grep -rliE '<words>' "<issue notes folder>"` — and the session transcripts as
a last resort (`grep -l -iE '<words>' ~/.claude/projects/<slug>/*.jsonl`).

**Discrimination**

| Counts as a repeat | Does not |
|---|---|
| Same SYMPTOM, any surface or cause (keycaps misaligned in Settings, then the strip) | A different symptom in the same component |
| An earlier fix that addressed one instance | A new feature request that touches the same area |
| The user's own words calling it a repeat — even with no ledger hit | |
| The same CLASS in a new flow: an earlier fix covered one path (a restored selection the view no longer held) and the report arrives through another (an account filter change left an excluded email in the pane) | A look-alike whose cause is a different class (a slow fetch is not a stale selection) |

| A WRITTEN RULE already covered the class but named it too narrowly (a skill said "never put a margin on a LINE"; the `---` rule WIDGET carried the margin and made the cursor jump) | |
| The first fix went into ONE instance and a skill NOTE, never into the TOOL that does the action (a script opened the wrong folder; the first fix patched one script and wrote a note, while the shared helper and other entry points kept doing it). A note is not the producer: fix the tool every path goes through, and add a hook or check that blocks the old way | |
| The fix SWAPPED the thing the user objected to for a look-alike (design preview pages: page screenshots → live scaled frames of the pages → a switcher strip above the page; "I'm still seeing screenshots", then "never include these top frames"). To the user it is the same thing whatever draws it. Fix = remove the whole category (the bare page, nothing added), never a new kind of wrapper | |

When the signals can't be read, **treat it as a repeat**.

**When a rule already existed and the defect still shipped, widen the rule's WORDING to the
whole class** (lines → everything in the flow; "content text" → every content size in every
mode) in the same fix — the narrow wording is the second producer.

### A "it took N seconds" report the app left no trace of

- **Trigger:** the report is a DELAY or a stall ("took about 15 seconds", "two windows opened after a while") and nothing records timings (no log, no trace), so the cause can't be proven from the code.
- **Discrimination:** reading code and guessing ships a fix for the wrong cause; measuring a copy of the data proves only that ONE path is fast. In the case behind this rule, the database write measured 17 ms on a copy, yet the real stall was six overlapping reads at launch, 2.3–4.3 s each, which no code reading showed.
- **Action:** in the same turn, add an EVENT-DRIVEN slow-step log (no timer, no background watcher): one line when an IPC call runs past a budget (exempt calls that wait on the network or a dialog), when a key reaches the page late (`performance.now() - e.timeStamp`), when a verb's handler runs long, and for each window open with the wait until it shows. Cap the file size. Fix what IS provable, and say plainly which cause is NOT found yet. Read the log after the next build: it often names the cause within minutes. For overlapping reads, the fix is one at a time with one rerun (`singleFlight`) plus "only the newest answer lands".

## 2. The permanent fix — all four, or it isn't permanent

**Steps 2 (twins) and 3 (a guard that fails) apply to EVERY defect fix, first report included.**
*Trigger:* you are fixing any defect, new or repeat. *Discrimination:* repeats often come from a first fix that skipped these steps because the checklist ran
only on second reports: a link-popup width fix that never looked at its twin (a hover popup with the same shape)
came back on the twin; a toolbar fix shipped with `guard: none`, and a later change broke it silently. A repeat still gets all four steps and the 🔁 report line. *Action:* before the ledger line,
grep for the shape (not a hand list: a scan of every `position: fixed` rule finds boxes a hand list misses) and fix
each twin in the same change; write a test that fails on the old code.

1. **Root cause at the ONE producer.** Find the single place the wrong thing is made (a
   formatter, a style, a query, a helper) and fix it there — or route every producer through
   one new component / function. Patching the screen in the screenshot is the failure mode.
2. **Sweep every instance.** `grep` the whole codebase for the pattern; migrate all of them
   in the same change, not only the one reported. **Include the TWINS**: when a flow
   has two paths (new item vs linked item; web vs native; edit vs preview mode),
   the same defect or feature almost always exists on the other path — three gaps in one day
   came from fixing one path only. Name the twin in the fix and give it the same test.
   **The guard reads EVERY page and EVERY data state, never a hand-picked list.** *Trigger:* the
   guard you are writing or extending walks pages, screens or views (a phone-width check, a look test, a "words on a
   card" scan). *Discrimination:* the phone test checked 7 hand-picked pages, so three other pages scrolled
   sideways unseen; the look test read only the sample data and the List view, so an empty Today, an empty Pipeline
   and the Board view broke unseen; a missing page and an error page had no state in any test. *Action:* loop over the
   app's shared every-page list (e.g. a `PAGES` list in `tests/e2e/pages.ts`), and add the states a sample can't reach as
   addresses: an empty filter (`?q=no-such-x`), each alternate view (`?view=board`), a missing address, a mistyped id.
   A state no test can reach (a whole empty database) gets one look in the screen review on a second server.
   The same holds for DATA a fix must reach: a sweep that rewrites stored links must
   cover every column that can hold one, so its guard walks the schema itself (drizzle `getTableConfig` over every
   table: each jsonb column and each text column named `*url` / `*site` / `*link`) and fails on a column that is in
   neither the sweep's list nor a `NOT_LINKS` list with its reason, and on a `NOT_LINKS` name the schema no longer has.
   Its first run found 6 columns a hand-written list had missed.
3. **An automatic guard that FAILS on regression** — a unit test, a source-scanning test, a
   lint rule, a build-script check, or a hook. A note alone is not a guard. Pattern that
   works (e.g. a `keycap.guard.test.ts`): a test that scans `src/**/*.tsx` for the banned
   shape (raw `<kbd`, a chord printed as JSX text) and lists every offender with file:line.
   Another pattern (e.g. a `web/test/rename.test.ts`): a test that scans source
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
   that set, so the test binary never puts a window on screen: split a view class into its own file. Prove the guard bites by compiling it
   once against the old file (`git show HEAD:Sources/X.swift > <scratch>/X.swift`): it must
   fail there and pass on the fix. Best of all is a MEASURED symptom (lay out with
   `NSLayoutManager` and compare rects), not only a source shape.
   **A guard walks the path the report took, not a shortcut to the same screen.**
   *Trigger:* the defect is about where a link lands or what a click does. *Discrimination:* the first deep-link guard
   opened a hand-picked address with a full page load (`page.goto("/trends#<id>")`) and passed for three weeks; the
   report came from an in-app click on a ticker item, a client-side navigation that never scrolled or marked the item,
   to an address the page did not draw at all. *Action:* read the links the site actually draws (every one, from the
   page), click each the way a person does (in-app, after hydration), assert the result, and put the reported case in
   the test data so the guard fails on the old code.
   The same holds for the INPUT path: a probe step selecting text by keyboard passed on the old
   code, because the defect (a format bar off the left edge) only happened after a MOUSE select, where the bar hides
   while the button is down. Replay the input the report used (mouse press, drag, release), and keep a step only once
   it fails on the old code.
   **A text-fit fix in one direction breaks the other: guard both wrap AND overflow.** *Trigger:* you
   fix text that wraps where it shouldn't (`white-space: nowrap`, shorter labels) or text that spills (wrapping
   again). *Discrimination:* the first fix stopped a sidebar answer wrapping; the next report was a value on another
   page running out of its box, because the same no-wrap rule now pushed it past the edge, and the wrap test passed.
   *Action:* give the text WIDTH (an icon above instead of beside, a wider column), never clip or spill; the guard
   measures every text element on every page against its nearest box (the first ancestor with a background, border
   or shadow): its text range must sit inside that box, and a row label must stay on one line where that is the rule.
   Run it on the old code first: it must name the reported text.
   **A size test is not a position test.** *Trigger:* the defect is about how two
   things line up (a word on a logo's baseline, a label beside an icon). *Discrimination:* the first lockup guard
   checked only the font size, passed, and the header kept centering the wordmark on the logo's box, and the report came back. *Action:* measure the relation itself in the browser: a zero-height marker inside the word for its
   baseline against the other element's edge, the font's own cap height (`measureText().actualBoundingBoxAscent`), after
   `document.fonts.ready`; prove the assumption it rests on (the logo's letters reach its bottom edge: 0 px, measured).
   **A guard on the reported symptom passes while the fix breaks a neighbour.** *Trigger:* you
   fix a layout defect (something sitting on something, a misplaced logo) and the new guard measures only that
   overlap. *Discrimination:* the new card layout passed all 118 browser tests (logo clear of the ring and tags) while
   the card's globe button had vanished and the logo was drawn too big: an `aspect-ratio` box stretched in one axis
   took its other side from it (a flex column's `flex:1` set the height, so the width came out 490px in a 413px card).
   Separately, a JS-placed element was measured before the web fonts loaded and passed at its pre-font spot. *Action:*
   (1) screenshot the real screen after every layout fix, even with every test green; (2) add a containment check
   beside the overlap check: every control and picture stays inside its card (`r.left >= card.left - 1 && r.right <=
   card.right + 1`); (3) measure after `document.fonts.ready` and two `requestAnimationFrame`s; (4) an `aspect-ratio`
   box that is stretched gets both sides set (`width:100%` with `align-self:stretch`), or loses the ratio.
4. **Written down with its trigger** — the project CLAUDE.md entry says what the class is,
   how a future session detects it (a grep, a measurement), and where the guard lives.

Verify with a **measurement**, not a look: geometry (`getBoundingClientRect` centers equal),
a failing-then-passing test, a count that went to zero.

**A decision inherited from an older repo is unguarded when its test can't run in the new one.**
*Trigger:* the repo was copied or grown from another (an engine folder copied from a sibling app), its `RECURRING.md`
carries the old repo's lines, and you are about to apply a general default (an accessibility norm, a library's
recommended setting) or fix a symptom. *Discrimination:* the older repo had decided that the app ignores the Mac's
Reduce Motion; the line sat in the new repo's ledger and its only test sat in the old repo's browser suite, which needs
the old repo's data and had never run in the new one. A session fixed a "not glowing" status dot by obeying Reduce Motion,
from habit, undoing a decision it should already have known. *Action:* (1) grep the ledger for the topic BEFORE choosing the fix, even when
the fix looks like a standard default; (2) for each old-repo decision whose guard cannot run here, add a guard that
does (a source-scan unit test runs anywhere, with no data) and list the rest in the project CLAUDE.md, read
every session; (3) name the old suite that doesn't run as work to port.

**A "never do X" decision is guarded by a scan for X, everywhere X can come from.** *Trigger:* the first
fix of a repeat settled a rule of the form "never X" (never obey an OS setting, never call an API from the client) and
its guard checked the instances first reported. *Discrimination:* the first guard pressed three named animations and
passed for three weeks while a redesign, ported from mockups, brought the banned OS check back on four new ones; the
mockups themselves held 18 more. A guard on instances can't see new code. *Action:* a source-scan test that fails on
the banned shape across the whole source AND the folders code is copied from (mockups, templates, an older repo's
copy), with its failure message naming what to use instead; keep the instance test as a second guard.

**A look guard that reads an empty page is not a guard.** *Trigger:* a browser test checks how
pages look (fonts, contrast, cards) on a made-up test copy of the data. *Discrimination:* the test data had no funds,
deals or Tracking items, so "small labels are never in the heading font" passed on an empty page; the
first real rows showed names in the tiny heading font, a wrong header and an empty picture slot.
*Action:* give the test data at least a few rows for every page the look tests visit, and add rows whenever a page
turns out empty; the same data serves the review copy the user looks at.

**A fix whose test can't fail the way production fails is not a guard.** *Trigger:* a fix
depends on an outside resource being there later: a file in a scratchpad or worktree, a cache, a server, a page path
saved in a record. *Discrimination:* a "pictures in the review" fix was tested against a fixture page that
never disappears, while 44 of 57 real pages were deleted with their scratchpads and worktrees before the weekend; the
test passed and the review went back to text. *Action:* the guard deletes (or breaks) the resource the way production
does, then checks the result still holds; when the resource is known to vanish, copy what you need at the moment it
exists (e.g. keep pages and their pictures in a `design/kept/` folder), and make the fallback LOUD (a red line on the
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
source scan missed a third caller reached through another entry point. A clean normal run proves nothing for a
load-dependent bug. *Action:* reproduce under the real load (a burst of requests while the store change reloads every
window), `sample` the stuck process and COUNT threads (100+ waiting = pool starvation, not one bad call), fix the
shared mechanism, and add a runtime guard that recreates the load; keep any source scan only as a second guard.

**Run the new guard on the OLD code before trusting a fix for an intermittent failure.** *Trigger:* a
test fails now and then (once in a full run, never alone) and you have written a test or probe that is meant to
recreate it. *Discrimination:* a test that held a save back 1.5 s, written for a suspected save-versus-page race,
PASSED on the old code: the race was real but not the cause. Measured instead: the flaky test run beside the suite
with `--repeat-each=6` failed 2 of 6 (alone 0 of 3): another test changed the same row at the same time; a trace kept
on failure later named a second cause. Timeouts on a loaded machine are a third look-alike. *Action:* (1) swap the old
file back in (`git show HEAD:<file> > <file>`), rebuild, run the new test: it must FAIL (several files in zsh: a list in
a string never splits into words, so `for f in $FILES` swaps nothing and the "old" run tests the new code; use an array,
`FILES=(a b)`, and print `git diff --stat` after the swap, which must be empty); if it passes, remove the
speculative fix instead of shipping it; (2) run the flaky test beside the others with `--repeat-each` before
suspecting the app; (3) keep a trace for every failed test (Playwright `trace: "retain-on-failure"`) and read its
network log; (4) before blaming a change for timeouts, read `uptime` and run the same tests on a build WITHOUT the
change (a load average of 40 timed out 6 of 6 either way).

**A fix that did not hold means your model of the bug is wrong: reproduce it where the user sees it
before writing another fix.** *Trigger:* a repeat report right after a fix you had
"verified". *Discrimination:* an email app's To-field cursor got three fixes in one day (Tab order, an
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
something still wrong after the fix ("one gap remains", "still possible", "except when …"). *Discrimination:* a report that ends "⚠️ One gap remains: … the card can still get stuck" with no fix
leaves the user to ask for the solution.
*Action:* small and safe → fix it before replying and report it as fixed; otherwise one line `❌ Not fixed: <what goes
wrong>`, then your fix in one or two lines, then the question (both options
named, a short table, answer words = column names). A fix that needs the user's OK first (a background watcher, a privacy
setting) is still named, never left out.

Tell the user about the written rule from the checklist in plain words: what happened before, what happens now,
and the file it was saved in; never as Trigger / Discrimination / Action labels.

## 4. Ledger — after EVERY defect fix, repeat or not

`RECURRING.md` at the repo root (committed, so every worktree sees it). Create it on the first
fix. A defect in how the AI's replies read or how its rules behave has no project repo: its ledger is
`~/.claude/RECURRING.md`. One line per defect, newest at the top of its section; update the count when a known
symptom returns:

```
MM-DD-YY · <symptom keywords, the words the user would use> · <Nth> report · cause: <one phrase> · fix: <what> · twins: <each place with the same shape and what was done, or "none (<why>)"> · guard: <test / check, or "none (<why no test can run>)">
```

- Symptom words are for FUTURE GREPS — write the words the user uses ("misaligned", "cut off",
  "nothing happens"), plus the component name.
- A first-time fix still gets a line (`1st report`) — that line is what turns the next mention
  into a detected repeat. A bare `guard: none` no longer passes: name why no test can run.
- A line with `guard: none` whose count reaches 2 is the ledger telling you step 2 is due.

## Where the rule lives

Add a short rule to your global `~/.claude/CLAUDE.md`, such as "A defect reported a SECOND time gets a
PERMANENT fix", that points to this skill. This skill is the playbook it points to.
