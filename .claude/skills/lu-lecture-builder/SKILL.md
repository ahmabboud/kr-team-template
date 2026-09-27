---
name: "lu-lecture-builder"
description: "Build or fix an interactive HTML lecture in a LebUniv_Course_Template course repo (lectures/*-session-NN.html), including any code (demos, whether a notebook or otherwise, and setup/verify scripts) that supports it. Use when asked to build a session, a lecture, or slides for a course that uses the lu-deck design system, to diagnose a deck that is clipping, overlapping, or rendering wrongly, to verify a deck or its supporting code before calling it done, or to start, continue, hand off or push any work session in such a repo (it covers reading and updating PROGRESS.md, REPLAN-STATE.md and the other metafiles)."
---

# Building a lecture in the LU deck system

You are writing a teaching lecture as one static HTML file against the `lu-deck`
design system (`assets/lu.css`, `assets/lu-deck.js`). The repository's own
`AGENTS.md` is the authoring contract and it wins over your instincts. This
skill adds what `AGENTS.md` cannot: the failure modes that have actually
shipped, and the discipline that catches them. Start with the metafiles
section right below: it decides what you work on and how you hand it over.

## Before and after every work session: the metafiles (do this first)

A course repo built on this system is worked on by several sessions, agents
and devices: a desktop Cowork session, a cloud or Claude Code session, a
second computer, all against one iCloud-synced folder and one GitHub repo.
The only memory they share is a handful of files in the repo, the
**metafiles**. Work that is not written into them does not exist for the
next session: it gets rebuilt over, contradicted, or reported as lost.

This has happened. A round of deck edits (two new diagrams, question-first
part dividers, bolded key claims) was committed and pushed but never written
into the tracker. Days later another session regenerated the same decks from
their generators without knowing the edits existed or why they were made.
Then a third session read only the last five commits, did not fetch, and
told the instructor the edits had been "lost from history". They were in
history all along; the regeneration had replaced the files. Every step
below exists to prevent one part of that chain.

### Which files, and what each is for

Names differ by course; the roles do not. Look for these at the repo root.

| File | Role |
|---|---|
| `PROGRESS.md` | What is built, per session. Status facts only. Read first. |
| `REPLAN-STATE.md`, or whatever open-work file `PROGRESS.md` points to | The single to-do list: numbered items, "Where to continue", "Settled decisions a new session must not reopen", how pushes work, and a dated log. Read second, in full. |
| `AGENTS.md` | The authoring contract, including the course's own teaching, IRI and lab standards. Where it disagrees with this skill (pacing, slide count, diagram component), it wins. |
| Plan documents they cite (`COURSE-REPLAN.md`, `PROJECT-REDESIGN.md`) | Approved outlines and decisions. Read the section for the session you touch. |
| `GLOSSARY.md` | Every term. A new term goes in, and the glossary is rebuilt, in the same change as the slide that introduces it. |
| `scripts/deckgen-sN/` | Generators. If one exists for a session, `lectures/*-session-NN.html` is build output, not source. |

### Start of a work session, before you say anything about the repo's state

1. **Sync first.** `git fetch origin`, then `git status`,
   `git log --oneline origin/main..HEAD` (local work not yet pushed) and
   `git log --oneline HEAD..origin/main` (work pushed from somewhere else).
   If GitHub is ahead, pull with `--ff-only` before touching anything. If the
   working tree has changes you did not make, stop and ask: another session
   may be in the middle of a change in the same folder.
2. **Read the metafiles now,** `PROGRESS.md`, then the open-work file in full,
   then the `AGENTS.md` sections they cite. Do not rely on what you remember
   from an earlier conversation or an earlier compaction summary. The files
   are newer than your memory, possibly by days and by whole rebuilt decks.
3. **Take the next task from "Where to continue"** unless the user gave you a
   different one. If the user's request conflicts with a settled decision,
   say so and ask; never quietly reopen it. If the next item is "waits for
   the instructor's review", ask for that review before starting anything
   else.
4. **Before claiming a commit or a change is missing,** check the whole
   history (`git merge-base --is-ancestor <sha> HEAD`,
   `git branch -a --contains <sha>`), not the last few lines of `git log`.
   Then check whether a generator rebuilt the file after it.
5. **Before editing any `lectures/*.html`,** check for a generator for that
   session. If there is one, edit its part files and rebuild; a hand edit to
   generated HTML is erased, silently, by the next build.

### During the work

- **Record as you go, in the same commit as the work.** Tick the item, and add
  a dated log line saying what changed, what was verified and how, and what
  could not be run. A decision the instructor makes in conversation goes into
  the log the moment it is made, in their words, and into "Settled decisions"
  if it binds later sessions.
- **A new requirement that applies beyond today's slide** (a teaching style, a
  tool rule, a layout rule, "always bold the main point", "open every part
  with the problem") goes into `AGENTS.md`, the standard, as well as the log.
  Otherwise the next rebuild will not follow it.
- **Status honesty.** Never mark a session done without the instructor's
  explicit agreement. "Built", "audited", "awaiting review" and "approved" are
  different states; write the one that is true.

### End of a work session

1. **Run the repo's own checks on every deck you touched:** its audit script,
   at rest and fully revealed, and its glossary check. Fix the findings, or
   record them as open items with numbers.
2. **Update `PROGRESS.md`** (status facts) **and the open-work file** (items,
   "Where to continue", log). "Where to continue" must be right for a reader
   who has seen nothing else.
3. **Commit as the repo's configured author.** No `Co-Authored-By` line, no
   session link, no AI attribution trailer of any kind.
4. **Push the way the open-work file says.** For KR that is a repo-local
   credential helper reading a token the instructor keeps outside every repo.
   Run a plain `git push`. Never read, print, copy or paste the token, never
   put it in a remote URL, never force push. If the helper cannot find the
   token, your session is missing the folder that holds it: a Cowork session
   given only the course folder cannot see a file one level above it. Do not
   conclude that you cannot push. Request access to the parent folder (for
   KR, `LebUniv/`), then run git from the repo's path under that new mount,
   where the helper's relative path resolves. Only if the push is still
   refused, stop and tell the user; do not work around it.
5. **Confirm** with `git ls-remote origin refs/heads/main` that GitHub matches
   `HEAD`. Then clear leftover `.git/*.lock` and `tmp_obj_*` files: on an
   iCloud mount `rm` may be refused while `mv` to a stale name works.
6. **Say plainly in your last message** what is pushed, what is only committed
   locally, and what the next session should do first.

## How this deck system actually renders (read this before diagnosing "responsive" complaints)

A `.slide` is a **fixed 1600x900 logical canvas**, not a fluid, reflowing web
page. `.deck__stage` scales that canvas with a CSS transform to fit whatever
physical screen it is shown on, the same model PowerPoint, Keynote, and
reveal.js use: one canonical layout, scaled uniformly, so line breaks and
pacing are identical on a phone, a laptop, and a projector. This is
deliberate, not an oversight, and it is what makes "3 to 4 minutes per slide"
pacing and "how many beats fit" reasoning possible at all: a fluid layout
would reflow differently per device and make that unpredictable.

Given that model, **cropping is never a screen-size bug.** It happens at
every screen size identically, because the authored content is taller than
the fixed 900px-tall (about 650px body) canvas at its one canonical size. The
fix is always to the content or its layout (shorten it, split the slide, drop
a companion element, widen a component to full width), never to add
"responsiveness," because the canvas is already scaling correctly to every
screen. If a stakeholder asks "why does this crop on my screen," the honest
answer is "it crops on every screen, including the one it was authored on,
because this slide currently holds more than the fixed canvas can show," not
a device-compatibility problem.

Why it went untested: rendering this design system requires an actual browser
paint, and a sandbox whose browser tool cannot reach the machine that would
serve the file (see the Cowork note below) makes it easy to ship a deck that
was written correctly but never actually rendered. That is exactly the gap
this skill exists to close. Treat "I wrote the HTML" and "I confirmed it
renders without cropping" as two separate, both-required steps, and say so
explicitly when only the first has happened.

## The one rule that matters

**A lecture is not verified until it has been rendered, measured, and clicked
through.** The same rule applies to any code the lecture depends on (a demo
notebook, a setup script): it is not verified until it has actually been run,
or at minimum had its logic exercised, not just written and read back.

Structural checks pass on badly broken decks. Balanced tags, present
attributes, unique ids: all green while 11 of 22 slides silently cut their
content, every answer rationale rendered one word per line in a 44px column,
the printed handout lost 237px from every page, a query sandbox returned
nothing and read as a broken button, and a study-mode note quietly wrecked a
definition grid only when printed. None of that is visible in the source, and
some of it only shows up when two modes are on at once (see "Testing every
interactive element" below). The equivalent for code: a demo notebook shipped
with a function that only ever imported one SDK, while its own markdown and
`.env.example` both promised a second provider would work too. Nothing in the
file itself flagged this, it only surfaced when someone actually tried the
untested path.

If you cannot render and measure, or cannot run and check the code, say so
plainly and call the work unverified.

## Pacing: how many slides

If the repo's `AGENTS.md` states its own pacing or depth standard, it replaces
this section. Knowledge Representation does (`AGENTS.md` 2c rule 7: about 30
content slides, about 180 minutes, realistic `data-minutes`, because a dry
run delivered a deck planned at 112 minutes in about 40). What follows is the
default for a course that has not set one.

**3 to 4 minutes per content slide.** A 180-minute session with 60 minutes of
lecture wants 15 to 20 lecture slides. With dividers and the wrap, the whole
session lands around **24 to 28 slides**.

Do not aim for 20 slides in a three-hour session. That is 6 to 7 minutes each,
which asks the instructor to talk for six minutes against six things on screen
and leaves the student a study artifact holding a fraction of what was said. It
is also what pushes slides past the fixed slide box, because the fix for an
over-full slide is to split it, not to shrink it.

While writing, count **beats**: the things you can point at and talk to (a
bullet, a code line, a table row, a callout, a reveal, a walkthrough step). Aim
for **5 to 8 beats on a 4-minute slide**. Below about 1 beat per minute the
slide is carrying more time than content, and that is the signal to split it.

Reconcile before you finish: sum `data-minutes` per `data-section` and check it
against the syllabus segment budget. A deck whose lecture sections sum to 73
against a 60-minute budget will overrun, and nothing in the file will tell you.
If a segment's slide content comes in well under its budget (say 19 of 70
minutes), that is not automatically fine just because the syllabus's own
prose names open work time, checkable time needs its own scaffolding: a
checklist, a worked example, an exercise, not just a bigger number sitting
unused. A section can legitimately end in a short "hand it to the room"
transition slide with a large `data-minutes` covering real open work time
that follows it, but that is a deliberate choice to make and state, not a
default to fall back on when content is thin.

### When one lecture has to cover several concepts, not one

A module built around a single deep idea (say, "structured output is the
whole contract") can give that idea real depth across many slides inside its
lecture budget. A module built around several genuinely distinct
architectural ideas in the same lecture budget (a raw call loop, a state
graph, a tool-wiring protocol, a routing pattern, all in the same 50 minutes,
say) cannot give each one that same depth through prose alone, the arithmetic
does not work. That is not a reason to ask for more minutes by default, and
it is not a reason to accept shallow treatment either: it is the specific
situation where a diagram earns its keep, because showing a structural
relationship is faster than describing it in words, and an interactive node
(see below) lets a student go one level deeper at their own pace instead of
the instructor narrating every part linearly. When a lecture segment's
concept count goes up, the fix is richer visual density per slide, not
automatically more slides or more minutes.

## Workflow

0. **Run the start-of-session steps** in the metafiles section above:
   sync, read `PROGRESS.md` and the open-work file, take the task from
   "Where to continue", check for a generator.
1. **Read `AGENTS.md` in full**, including its trap section. Read the existing
   reference lecture and imitate its structure.
2. **Gather the content first.** Objective, segments and minutes, deliverable,
   reading. Usually from a syllabus.
3. **Copy `lectures/_template.html`**, fill the `<body data-*>` block, write the
   slides at the pacing above. If the course builds decks with a generator
   (`scripts/deckgen-sN/`), write the parts there and build instead.
4. **Add the lecture card to `index.html`.**
5. **Verify by measuring.** See the protocol below.
6. **Verify by clicking.** See "Testing every interactive element" below, and
   run it once per mode AND once with modes combined (see the note on that).
7. **Run the end-of-session steps** in the metafiles section: checks, both
   metafiles updated, commit without AI attribution, push, confirm.

Do not report a deck as done after step 4. Steps 5 to 7 are the deliverable,
not an optional follow-up.

## Sizing a single slide

A `.slide` is about **1600x900 canvas px, with the body around 650px tall**, and
it is `overflow:hidden`. Content that does not fit is cut with no scrollbar and
no error, on every screen size, because the canvas is fixed (see above).

- **The taller column sets the height** in a two-column layout. Measure both
  before trimming; shortening the short one changes nothing.
- **Measure the revealed state too.** Open reveals and shown rationales are the
  state you teach in and they add hundreds of pixels.
- Do not shrink type to fit: an overloaded slide needs a scale below 0.78,
  which puts the 20px projection floor under 16px. Prototyped and rejected.
- Two proven fixes when a slide still overflows after trimming text: **split
  it into two slides** (each keeps its own share of the original minutes), or
  if it is a single component like an MCQ carrying a companion callout, **drop
  the companion and widen the component to a single full-width column**. Both
  have measured to a clean 0px overflow in practice; trimming words alone
  often only gets partway there.
- **An MCQ or poll option has a height floor once its rationale is revealed,
  and trimming the rationale text does not lower it.** `.lu-mcq__opt` lays
  out as a grid with implicit row sizing; once `.lu-mcq__why` is shown, each
  option settles to roughly **103-119px regardless of how short its text
  is** (shortening a rationale all the way down to "No." changed nothing,
  confirmed by direct testing). If an MCQ or poll overflows in its answered
  state, do not keep editing the rationale copy hoping to claw back pixels;
  the two levers that actually work are **fewer options** (cut a weaker
  distractor) and/or **dropping a companion column and widening the
  component to full width** (see above). Move whatever you cut into
  `<template data-notes>` so the detail is not lost, just no longer on-slide.
- **Splitting a slide means renumbering the deck's own order-comments and
  reconciling minutes, not just adding a new `<section>`.** This deck marks
  slide order with decorative HTML comments (`<!-- ==== NN ==== -->`) above
  each `<section>`; inserting a new slide in the middle means shifting every
  later comment's number up by one (a high-to-low loop, e.g. via `perl` or a
  small script, avoids double-renumbering the same file), and splitting one
  slide's `data-minutes` across the two new slides so their sum still equals
  the original budget.
- **A scoped CSS custom-property override on one `<section class="slide">`
  shrinks that slide's shared gap without touching any other slide.** Layout
  gaps in this system read shared tokens like `var(--lu-s5)`. Adding
  `style="--lu-s5:12px"` directly on one slide's `<section>` element
  overrides that token for everything inside it alone; every other slide
  using `--lu-s5` is unaffected. Prefer this over editing the token in
  `assets/lu.css` (which is global, see "Editing shared assets") whenever
  the problem is really just one crowded slide.
- **A `:has()` selector can hide an element based on a sibling's revealed
  state with no JS change at all.** For example,
  `.lu-poll:has(.lu-poll__bars:not([hidden])) .lu-poll__timer { display: none; }`
  hides a countdown timer the moment its poll's results are revealed,
  because the timer is irrelevant once answered and the vertical space it
  frees can be the difference between fitting and overflowing. Reach for
  this before adding a `lu-deck.js` change when the rule is purely "hide X
  once Y is in state Z."

## Verification protocol

Serve over http, or use a live pushed URL if that is the only thing reachable
from your environment (see the Cowork/sandbox note below). Then:

1. Open with a **`?cb=<random>` cache-buster** and confirm the version you
   loaded. A service worker or the HTTP cache can hand you a page that is not
   the one on disk. Hours have been lost to this.
2. **Clear the deck's saved state**, or a restored answer (or a restored
   study-mode toggle, see below) inflates a slide:
   `Object.keys(localStorage).filter(k=>k.startsWith('lu:')).forEach(k=>localStorage.removeItem(k))`
   plus a plain `localStorage.clear()` if you are not sure what keys this
   version of the deck uses, then reload.
3. Paste **`scripts/audit-deck.js`** into the console. Fix everything it reports.
4. **Print preview with Handout on.** Print is a second layout with its own
   rules and has shipped broken independently of the screen.

### Identify slides by `data-label`, never by a remembered slide number

Informal "slide 11" or "slide 14" numbering is fragile the moment a deck has
been edited, split, or reordered even once, and trusting it costs real time:
a slide believed to be the culprit at "position 11" was actually fine on
live testing, while the real problem was a different slide entirely,
misdiagnosed under the wrong name for a significant stretch, purely because
of an assumed position. Always resolve a flagged slide by its content, not
its ordinal:

```
[...document.querySelectorAll('.slide')].find(s => s.dataset.label === 'Exact label text')
```

`window.LUDeck.go(N)` is **1-based** (index `N-1` internally) if you do need
to jump by position, but treat any number you were told or remember as a
hypothesis to confirm against `data-label`, not a fact to act on directly.

### A fast full-deck sweep is a screening tool, not a measurement

Looping `window.LUDeck.go(i+1)` across every slide with a short settle (around
150ms) is a good way to screen an entire deck for overflow candidates in one
pass, but its numbers can be **inflated and unreliable for some slides**: one
sweep reported two slides at 614px and 758px overflow that individually,
with a longer settle (300-400ms) and careful per-slide testing, actually
measured at 189px and 372px. Never fix a slide off a sweep number alone.
Use the sweep only to build a candidate list, then re-verify every candidate
individually with a longer wait before trusting its number or starting a fix.

### Units, or every number you report will be wrong

- `getBoundingClientRect()` returns **scaled screen px**.
- `clientHeight` / `scrollHeight` return **unscaled canvas px**.

Mixing them inflates everything by `1/scale`, roughly 2.15x. Pick one, convert
deliberately, and say which you are reporting.

### Verifying before deployment

Two techniques, both with limits worth knowing:

- **Inject each slide body into a running deck** and read
  `scrollHeight - clientHeight`. This is accurate for plain content and **lies
  about runtime-wired components**: un-wired `.lu-mcq__opt` and `.lu-sort__item`
  collapse into the narrow grid column and read far too tall, and an un-wired
  `.lu-query` collapses to nothing and reads far too short. Compare those
  against the same wired component in an already-deployed deck instead.
- **Compare content mass against slides known to fit.** If every slide in the
  new deck has fewer words, code lines and blocks than slides that measured zero
  overflow, that is a sound structural argument. Say it is an argument, not a
  measurement.
- **Never detach and reattach an already-wired component while live-testing
  it (`el.replaceWith(...)`, `el.remove()` + `append()`).** Doing this to
  simulate "drop the companion column, widen to full width" silently breaks
  the component's `hidden`-attribute-driven show/hide behavior, because it
  re-enters the DOM without going through the runtime's own init path. This
  produced a false reading of ~941px overflow on a component that, tested
  correctly, was nowhere near that. Simulate structural changes with
  **CSS-only mutation on the live node instead**, which never detaches it:
  `container.style.gridTemplateColumns = '1fr'; companion.style.display =
  'none';`. If a live-DOM test result looks implausibly large, suspect a
  detach/reattach before suspecting the actual content.
- **Content marked `data-build` is hidden until its build step is advanced.**
  A live-injected diagram placed inside an element that still carries
  `data-build="1"` measures and renders as blank on first view. Remove the
  attribute (or advance the build) before judging whether it renders, and
  decide deliberately whether a picture should be a build step at all: a
  concept slide's leading picture usually should show on arrival.

A query sandbox needs no browser at all:

```
node -e "global.window={}; require('./assets/sparql-lite.js');
  const {parse,query}=global.window.LUSparql;
  /* pull the data-data and data-query templates from the HTML,
     decode the entities, then parse() and query() them */"
```

### Cowork / sandboxed-shell environments: localhost is not reachable

One exception first: if the repo ships its own headless audit (KR has
`scripts/audit-all.py`, Playwright against a server on a local port), run it
in the same shell that serves the files. That path does reach localhost,
because the browser and the server are on the same machine. It is the
check the open-work file expects. Still confirm the live site after
pushing. The rest of this note is about the browser pane, which is a
different machine.

If you are working from a session whose browser tool is separate from the
shell that would serve the file (a "browser pane" that cannot reach a
localhost server your own sandbox started), do not burn time on that path.
Push the branch, use the deck's real published URL (GitHub Pages or
equivalent), and run the whole verification protocol against that. This is
slower per iteration (you must push to see a change) but it is the only path
that actually renders. Before pushing a batch of speculative fixes, you can
test a candidate change without committing it at all: navigate to the live
page, then patch the live DOM directly with a script call (replace a slide's
`innerHTML`, force a reveal open, remove an element, change a grid style) and
re-measure. Once a candidate measures at 0px overflow live, write the same
change into the file and commit it. This turns a slow push-and-check loop
into a fast, disposable one, and it is how the single-column-MCQ and
split-slide fixes above were actually found. Never report a deck as verified
based only on reading the source in this situation; say plainly that live
rendering has not happened yet if you have not reached a real page.

When you exercise a graded component (an MCQ, a walkthrough) live, be honest
about what a repeated `.click()` on the same button proves after the first
click: the component disables its options once answered, so a second or
third programmatic `.click()` on an already-disabled button silently does
nothing, and reading stale feedback text afterward looks exactly like a wrong
answer being marked correct (or vice versa) when it is actually just an inert
click. Test each option's real behavior from a clean, unanswered state (clear
storage and reload, or navigate fresh, before every single click you intend to
mean something), not by clicking several options back to back on the same
instance. Likewise, a full page navigation is not guaranteed to be a hard
reset: this deck restores slide position (and, once answered, quiz answers)
from `localStorage`, so a saved position or a saved answer can silently carry
over into what looks like a "fresh" load unless you actually clear storage
and confirm the state is unanswered before you click.

### Transcribing a live-tested fix into source: verify every mutation landed, then re-sweep the whole deck after pushing

When a live test mutates several elements in one go (a `.forEach()` over a
`querySelectorAll()` result, say), **every one of those elements' changes
must show up in the eventual source edit, not just most of them.** A
`blanks.forEach(b => b.style.width = '110px')` touching four inputs, followed
by a source edit that only added `style="width:110px"` to three of them
(the fourth was skipped on the assumption its paragraph already looked fine
in isolation), shipped a real, deployed overflow regression that survived
the commit and the push, and was only caught by a full fresh re-sweep
afterward. Two habits close this gap:

- After transcribing a live-tested change, **re-run the exact mutation
  selector against the source diff**: if the test used
  `querySelectorAll('.lu-blank')` and got 4 results, confirm the source edit
  touched all 4, not "the ones that seemed to need it."
- **Always run one more full-deck re-verification sweep against the fresh
  live deployment after pushing**, covering every slide, not only the ones
  you touched. A partial transcription is exactly the kind of bug a diff
  review will not catch, because the diff looks correct for the elements it
  does contain, and only a live re-render exposes the one it is missing.

If the deck is generated, the live-tested change is transcribed into the
generator's part file, not the HTML, then rebuilt; the same completeness
check applies to the rebuilt output.

## Testing every interactive element

"The audit script passed" is not the same claim as "every button on every
slide works." Structural measurement cannot click things. Before calling a
deck done, walk every slide and exercise whatever is actually on it:

- **Term popovers** (`.lu-term`): click each one open, confirm the popover
  lands on-slide (not clipped by the edge) and closes on outside-click or its
  close button.
- **Interactive diagram nodes** (`.lu-node[data-term]`): same click-to-reveal
  mechanism as a term popover, on a node inside a `.lu-board`. Click each one
  open, then also check study mode (`S`): a paragraph term becomes an inline
  note in place, but a node cannot (it is absolutely positioned inside the
  board's percentage canvas, an inline note there would overlap the diagram),
  so it should appear instead as a row in one `.lu-board__legend` placed after
  that board. Confirm the legend appears with study mode on and disappears
  cleanly when it is turned back off.
- **Reveal buttons** (`.lu-reveal__btn`): click to open, confirm the panel's
  content does not push the slide into overflow (measure after opening, not
  before).
- **Walkthrough steps** (`.lu-walk`): step forward through every step using
  its real nav control (the "Next step" button, or its dot buttons), not by
  clicking the hidden step panels themselves, which are inert content divs
  with no click handler of their own.
- **MCQs** (`.lu-mcq`): click every option, including the correct one and the
  visually longest rationale, and confirm the feedback box does not overflow.
  Test each option from a clean, unanswered instance (see the note above on
  stale/disabled-button false readings), not several options in a row on the
  same one.
- **Sort/drag items**, **fill-in-the-blank**, and any other graded component:
  interact with it the way a student would, not just glance at the markup.
- **Query sandboxes** (`.lu-query`): actually run every seeded query. A
  find-the-violation query over data with no violation legitimately returns
  "0 rows," and that reads exactly like a broken button unless you already
  know the expected result. Check the expected result against what the
  speaker notes claim.
- **Study mode (`S`) and Handout print, independently AND together.** Each
  mode has its own layout rules, but the deck is not fully tested until you
  have tried print WITH study mode already on. Study mode's inline
  definitions are real DOM insertions, not just a CSS toggle, so a bug that
  only appears when a definition list gets a study-mode note injected into it
  will not show up under either mode alone. Study mode also **persists across
  reloads via localStorage**, so if you or a previous session turned it on
  while testing something else, it will silently still be on the next time
  you print or export unless you explicitly clear it (see the localStorage
  clear above). This exact combination shipped a real overlap bug once
  already: see the trap table below.
- **Presenter view (`P`)** and the **session timer (`T`)**: open them, confirm
  they show the right slide and do not desync from the main view.

## Traps that have already shipped

| Trap | Rule |
|---|---|
| Work done and pushed but never written into the metafiles | A pushed round of deck edits never reached `PROGRESS.md` or the open-work file. Days later another session regenerated the decks without knowing the edits or the reason for them. Tick the item, log it, and write any rule that binds the future into `AGENTS.md`, in the same commit as the work. |
| Hand-editing a deck that a generator builds | The next `build.py` run replaces the HTML wholesale and the edit disappears with no conflict and no warning. Check `scripts/deckgen-sN/` before touching `lectures/*.html`; edit the part file and rebuild. |
| Reporting the repo's state from memory, or from `git log -5` | A session told the instructor its own pushed commit was lost; it was in history, overwritten by a later rebuild. Fetch first, read the metafiles, and check ancestry (`git merge-base --is-ancestor`) before saying anything is missing. |
| AI attribution in commit messages | Another session's commits carried `Co-Authored-By` and session-link trailers against the instructor's standing preference. Never add them. |
| Concluding "I cannot push from here" because the token file is not visible | A Cowork session told the instructor it could not push, when it only lacked the parent folder that holds the token. Request access to that folder, run git from the repo's path under the new mount, and push. The instructor has pushed from Cowork this way before. |
| A class rule setting `display` beats the UA `[hidden]` rule, so hidden things keep their space or stack | Never set `display` on something the runtime hides. If you must, ship a `[hidden]{display:none}` companion in the same commit. Shipped four times now (`.lu-reveal__panel`, `.lu-mcq__why`, `[data-walk-step]`, and later `.lu-poll__bars`), the second and later times as a regression while fixing something else. Given this history, treat it as a class of bug: whenever you add or touch any revealable/hideable component, proactively grep its CSS for a bare `{ display: ... }` rule with no matching `[hidden]{display:none}` companion, rather than waiting for it to show up as an overflow symptom. |
| A grid child with no `grid-column` lands in the narrow first column and renders one word per line | Any child of a narrow-first-column grid gets an explicit `grid-column`. |
| Study mode's inline definition note (`.lu-inline-def`) is inserted as a bare `<div>` sibling inside a `.lu-defs` dt/dd grid | A plain div consumes one grid cell and throws every dt/dd pair after it out of alignment, and blows the label column width out to fit a full sentence. Only visible with study mode on, and easy to miss because it looks fine with study mode off. Fixed upstream in `lu-deck.js`'s `Term.inline()` by forcing the note onto its own full-width row (`grid-column:1/-1`) whenever its host is a `<dd>`. If you see a definition list overlap in print or study mode, check this first. |
| `width:100%` plus `aspect-ratio` ignores available height and overlaps its neighbours | In a height-constrained box use an explicit height and `aspect-ratio:auto`. |
| Two single-class rules tie and source order silently wins | Raise specificity deliberately (`.parent > .child`). |
| Print keeps the fixed slide height and clips | Check print preview with Handout on after any sizing change. |
| A stale service worker or HTTP cache serves an old page | Verify with a cache-buster and confirm the version. |
| A sandbox that runs and returns nothing | A find-the-violation query over data with no violation reports "0 rows" and reads as a broken button. Run every seeded query through the engine, and every query you suggest in a caption. |
| Em dashes and stray characters | This course does not use em or en dashes as punctuation. Non-ASCII on a slide must be a deliberate symbol. Check the runtime too: five were being emitted into visible UI from `lu-deck.js`. |
| Too few slides carrying too much time | See Pacing above. This is the root cause of most overflow, and it is a content problem, not a screen-size problem: the canvas is fixed and scales identically everywhere. |
| A mode that persists silently | Study mode's on/off state is saved to localStorage and survives a reload. Testing "with study mode off" only tells you about a state a real user (or a previous test pass) may not actually be in. Always clear it before a fresh baseline measurement. |
| A demo notebook's markdown and `.env.example` promise a second provider works, but the actual function only ever imports the first | Grep the code for every SDK import and provider name the notebook's own text promises. If a promised path was never exercised, either wire it for real or stop promising it. |
| Clicking several MCQ options in a row on the same instance to "test" each one | The component disables all options after the first answer. The second and third `.click()` calls are silent no-ops on disabled buttons; whatever feedback text you read back is leftover from the first click, not a fresh evaluation, and can look exactly like the correct option being marked wrong. Reset to a clean, unanswered instance before every option you actually mean to test. |
| Detaching and reattaching an already-wired component (`replaceWith`, `remove()`+`append()`) while live-testing a structural change | This breaks the component's `hidden`-attribute-driven behavior because it skips the runtime's init path, producing a false, inflated overflow reading. Simulate the structural change with CSS-only mutation on the live node instead (change `grid-template-columns`, toggle `display` on a sibling), never by detaching it. |
| An MCQ or poll option's rationale text gets shortened again and again but the option's box height does not shrink | `.lu-mcq__opt` has an implicit-grid height floor of roughly 103-119px once its `.lu-mcq__why` is revealed, essentially independent of text length. Stop trimming words and instead cut an option or drop a companion column to go full width. |
| A live test mutates N elements via a loop (`forEach`/`querySelectorAll`), but the source-edit transcription only covers some of them | Every element the test loop touched needs the matching change in source, not just the ones that "looked like they needed it." Diff the transcription against the original selector's result count before calling the fix complete, and always run one more full-deck re-verification sweep against the live site after pushing, not just of the slides you touched. |
| Believing a remembered/assumed "slide N" position instead of the slide's actual `data-label` | Positional numbering drifts the moment a deck is split, reordered, or edited even once, and following it can mean fixing (or clearing) the wrong slide entirely while the real problem goes untouched. Always resolve a flagged slide by `data-label` content. |
| Trusting a fast full-deck overflow sweep's numbers directly | A short-settle `go(i+1)` loop across the whole deck is a good screening pass but can report numbers far higher than reality for some slides. Always individually re-verify any sweep-flagged slide with a longer settle before trusting the number or starting a fix. |
| A flat demos folder with every module's file side by side and one ambiguous shared setup | Give each module's code its own `module-NN/` subfolder; keep the one shared `requirements.txt`, `.env.example`, and top-level README one level up, not duplicated per module. See "One folder per module" below. |
| A dependency floor like `>=1.0` lets pip resolve a version that predates the API the code actually calls | Pin the floor to the version confirmed (via current research, not memory) to have the capability you call, e.g. `google-genai>=2.3.0` for the Interactions API, not `>=1.0`. Symptom looks like a code bug (`AttributeError: 'Client' object has no attribute 'interactions'`) but is a requirements-file bug; comment the file explaining why the floor is where it is, so nobody "helpfully" loosens it later. |
| A later module's demo needs a tool with its own materially different dependency floor (e.g. a CLI requiring a newer Python than earlier modules needed) | Fold it into the one shared `requirements.txt` anyway, with a comment naming the floor and which module needs it, and call it out prominently in the shared README's setup steps and troubleshooting. One shared file that occasionally raises the floor for everyone is simpler to actually use than a second requirements file instructors have to discover and remember exists; state the tradeoff in the README rather than routing around it with a per-module file. |
| A single top-level README covers setup for every module's demo, but nobody can tell how to actually run any one of them from it | One shared README is for one-time setup only. Give each module's own `module-NN/` subfolder its own `README.md` with exact, copy-pasteable run steps for that specific demo (the command to run, what a CLI-driven demo opens and what to do once it does, which fake inputs exist to test with). The shared README should say this and point to them, not restate them. |
| A CLI or dev-server-driven demo needs its own platform credential, separate from the model API key the course's own code calls, because the tool's hosted UI has to authenticate the connection to your local server | `langgraph dev` + LangGraph Studio is the concrete example: Studio is a page at `smith.langchain.com`, so it needs its own LangSmith API key to connect to your local server, on top of whatever key the agent itself calls. This is easy to miss when writing setup instructions, since the course's own code never calls this credential, only the tool does. Actually run the CLI end to end at least once, all the way to the point where it asks for something, before calling setup instructions complete. Document that credential with the same treatment as any other: the literal URL, whether it costs money, and where the value goes. |
| A stale cross-reference to a sibling repo's folder structure (a lecture's speaker notes describing "the team template's notebooks/ folder and its app.py") outlives a rename or restructure of that other repo | A claim about another repo's structure is a fact that can go stale independently of anything in this file. When touching a slide that names another repo's folders or files, open that repo and check the claim is still true, not just that this file's own syntax is fine. |

## Diagram geometry

Node positions are percentages of `.lu-board`; SVG paths are viewBox units.
Setting them independently by eye does not work. **Compute the paths from the
node positions:**

```
viewBox x = left% * (viewBoxWidth / 100)
viewBox y = top%  * (viewBoxHeight / 100)
```

Pull each endpoint back by about half the node's width or height. Set the
**viewBox aspect ratio to match the board's rendered aspect ratio**, or
`preserveAspectRatio="none"` stretches every label. Verify by measuring
endpoint-to-node distance; over about 40 canvas px is a stray. Give each step's
`<marker>` a **unique id**, or markers defined only in step 1 stop resolving
when step 1 is hidden.

If the course has adopted a newer diagram component (KR uses `lu-flow`,
`assets/lu-flow.js`, for anything with arrows since design system v1.2), use
that, per its `AGENTS.md` graphics section, instead of hand-placing a
`.lu-board`.

## Making concepts visual and interactive, not just described

A module whose content is mostly one conceptual thread (say, "the schema is
the contract") can teach well with prose, a term popover here and there, a
walkthrough, an MCQ. A module whose content is real structure, a state
machine, a protocol, a multi-step call sequence, a pipeline, needs that
structure shown, not narrated, especially once several such structures share
one lecture budget (see "When one lecture has to cover several concepts, not
one" above). Reach for a diagram primitive before prose, per `AGENTS.md` §7's
graphics order, and prefer an **interactive** diagram over a static one
whenever a specific part of it needs its own explanation:

- **`.lu-node[data-term]`** (a `.lu-node` that is a `<button type="button">`
  with `data-term`, `data-kind`, and a nested `<template>` or `data-def`, the
  same contract as a `.lu-term`) turns one node on a `.lu-board` into a
  click-to-reveal definition, so a block-diagram or data-flow diagram can
  carry per-part detail without cramming it into a caption next to the
  board. Study mode renders these as one legend after the board
  (`.lu-board__legend`), not inline at the node, since a node's absolute
  position inside the board's percentage canvas cannot host an inline note
  the way a paragraph term can. See `design-system.html` §8, component 12,
  for the live example and markup, and `AGENTS.md` §5 for the cheat-sheet row.
- A **plain `.lu-node`** (no `data-term`) is unaffected and stays a
  non-interactive div, exactly as before. Only add `data-term` to a node when
  that specific node needs explaining beyond its label.

Two things this system does **not** have a component for yet, stated
honestly rather than worked around with prose or an ad hoc one-off:

- **Sequence diagrams** (time-ordered message arrows across lifelines, the
  natural shape for "here is the raw agent loop, call by call" or an MCP
  tool-call exchange). No primitive exists for this; it is a new component,
  not a reuse of `.lu-board`, and belongs behind the "Adding a component to
  the system" procedure in `AGENTS.md` §11 (CSS, then `init()` in
  `lu-deck.js`, study mode and print, a live demo in `design-system.html`,
  the cheat-sheet row, the `?v=` bump) before it is used in a real lecture.
- **Scoped further reading**, a pointer to one paper or article tied to the
  specific concept a slide is teaching, distinct from the module's own
  reading list in the syllabus, and shown only when it is genuinely relevant
  to that slide, not as a standing bibliography. No component exists for
  this yet either.

Do not invent a one-off version of either inside a single lecture file. If a
module's content calls for one, build it as a real shared component first,
following AGENTS.md §11, the same way `.lu-node[data-term]` was added.

## Editing shared assets

`assets/` is shared by every deck in every course.

- A defect there is usually reproducible in the repo's own reference lecture.
  Check, and say so: it tells the owner the fix belongs upstream.
- Fix it **upstream in the template**, then flow it down. If the open-work
  file tracks drift between this repo's assets and the template repo's,
  change both or record the drift there.
- Bump the `?v=` query on every HTML file and in `SHELL` in `sw.js` **only if**
  the service worker's fetch strategy for that asset is cache-first. Check
  `sw.js` first: a stale-while-revalidate strategy (serve the cached copy
  immediately, refresh in the background) means an edit reaches every open
  tab on its next load with no version bump needed at all, and bumping
  anyway is harmless but pointless busywork.
- Re-run the audit on **two** decks, since a fix for one can regress another.
- When adding a genuinely new component (not just using an existing one),
  follow `AGENTS.md` §11 in order: CSS in the right numbered section using
  existing tokens only, an `init()` in `lu-deck.js` called from `boot()`,
  study mode and print handled explicitly (do not assume an existing mode
  handler will do something sensible with a component it does not know
  about), a live demo and markup in `design-system.html` §8, a row in
  `AGENTS.md` §5's cheat sheet, then the `?v=` bump. A component not
  documented in `design-system.html` does not exist, the next session will
  not find it and will either miss it or reinvent it.

## Writing the slides

- Every slide needs `data-label`, `data-section`, `data-minutes` and a
  `<template data-notes>`.
- Speaker notes are part of the deliverable: what to say, how long, the question
  the room will ask, the misconception the slide breaks, the blocker that
  appears 20 minutes into the lab. Never restate the slide. If a sandbox or
  exercise has a known correct result, **name it in the notes**.
- Every graded component needs a unique `data-qid` prefixed with the session.
- Every answer option needs a rationale, correct ones included.
- Two grounds only: paper, and `slide--night` for title and dividers.
  `slide--tint` marks a change of activity.
- Nothing below 20px. No emoji. No `<style>` blocks. Never draw a screenshot as
  SVG: ship a `.lu-figure__ph` placeholder naming the capture needed.
- A divider before every part, a check question after every concept block, never
  more than four consecutive paper slides, and the last two slides are always
  wrap then self-check.
- When you cut detail from a slide to fix overflow (shorten a sentence, drop
  a rationale, remove an option, remove a companion callout), move the cut
  content into `<template data-notes>` rather than discarding it, so nothing
  taught is actually lost, only moved off the visible canvas.
- The instructor's teaching style, which KR records in `AGENTS.md` 2c: lead
  every concept slide with a picture; open each part with the problem, and
  ask the room about it before giving the answer; bold and define every term
  at first use, visibly; say why before how. If a course has not written this
  down yet and the instructor states it, write it into that course's
  `AGENTS.md`.

## Writing any code the course ships (demos, setup scripts, starter code)

A lecture increasingly comes with real Python alongside the HTML: an
instructor demo, a setup/verify script, starter code in a team template. The
same "not done until checked" discipline applies, and a few things specific
to code:

- **A demo does not have to be a notebook.** Pick whatever tool actually
  shows the concept: a notebook for a side-by-side comparison of two
  approaches, a plain script that prints state after every step for a raw
  loop, a framework's own CLI/dev server (`langgraph dev` and Studio, say)
  when the thing worth showing is the framework's own tooling, not cells of
  output. Do not default to a notebook out of habit once a module's concept
  calls for something else. Whatever the shared top-level folder for this
  material is called (`demos/`, not `notebooks/`, once a course has more
  than notebooks in it), the folder convention below still applies to every
  kind of demo the same way.
- **One folder per module for its own code, one shared requirements file for
  all of them, no exceptions.** Do not let a flat folder (say, `demos/`)
  accumulate every module's demo side by side with no separation; six
  months in, nobody can tell which file belongs to which lecture without
  opening each one. Give each module its own subfolder named for the
  lecture it supports (`demos/module-01/` for `dsca-module-01.html`,
  `demos/module-02/` for the next one), and put only that module's own demo
  (and anything specific to it) inside. Keep genuinely shared, common
  things one level up instead of copied into every subfolder: a single
  `requirements.txt` and `.env.example` that cover every module's demo.
  Duplicating a shared `requirements.txt` into every module folder is worse
  than one shared file, not safer: it invites the copies to drift out of
  sync with each other, and it is one more file an instructor has to
  discover and remember exists before they can run anything. This holds
  even when a later module's tool has a materially different dependency
  floor than the earlier ones (a CLI needing a newer Python, say): fold it
  into the one shared file with a comment naming the floor and which module
  needs it, and call the floor out prominently in the shared README's setup
  steps and troubleshooting, rather than forking a second requirements file
  to route around it. The same one-file rule applies to `.env`: one shared
  `demos/.env` (and `.env.example`) for every module's demo, read from
  wherever the module's own script or config resolves the path to (a
  `Path(__file__).resolve().parents[N]` computed to land on the shared
  `demos/` folder specifically, not the repository root and not a
  per-module copy). A later module needing an additional credential the
  earlier ones did not (see the LangSmith example in the trap table) still
  goes into this same shared file, one new line, not a new file. When you
  move a demo into its own subfolder, fix its own relative paths (a
  `%pip install -r requirements.txt` cell needs to become
  `../requirements.txt`) and update every place that names its old path:
  the lecture's own speaker notes, the top-level README's mention of it,
  and the demos README's cross-link table.
- **Give each module's demo subfolder its own README with exact run
  steps.** A single shared README covering setup for every module's demo
  is not the same thing as "how do I run this specific one," and an
  instructor who only has the shared page cannot tell. The shared
  `demos/README.md` should cover one-time setup only (environment, install,
  every credential any module's demo needs, even ones most modules never
  touch) and then point to each `module-NN/README.md` for the actual
  command to run, what a CLI-driven demo opens once started and what to do
  in it, and which fake inputs exist to test with. Write that per-module
  README as part of building the demo, not as an afterthought once someone
  asks how to run it.
- **Keep it minimal.** Write only the provider, path, or option the course
  actually uses. Do not add a second provider, a config flag, or a branch
  "in case someone wants it" unless the course has actually decided to
  support both, that decision belongs to the person running the course, not
  a default to reach for. Extra branches are extra untested surface, and each
  one needs the same testing as the one you actually need.
- **Comment for the reader, not for yourself.** Every function gets a short
  docstring or comment saying what it does and why, in plain language a
  student encountering it for the first time can follow. A one-line "what
  this does" beats a paragraph of implementation narration. This applies to
  every function in a file, not only the ones you happen to touch on a given
  pass: a script rewritten in parts (say, two functions rewired for a
  provider swap) can end up with docstrings on exactly those two and none on
  the rest, which is worse than consistently having none, it looks
  deliberate. When you rewrite part of a file, sweep the whole file for this
  before calling it done.
- **Test what you can before calling it ready to teach**, even without a
  real API key: extract the function, mock the client, and run it against
  the actual expected inputs (a real conversation, a real turn) to catch
  wrong branching, undefined names, or a response shape that does not match
  what the rest of the code expects. This will not catch a wrong model name,
  a real API contract change, or a platform credential a CLI tool demands
  only once it actually tries to connect somewhere (see the LangSmith
  example above), say so plainly: that last mile still needs one real,
  end-to-end run with live keys before Module day, not just a mocked test
  of the code's own logic.
- **Pin dependency floors to the version that actually has the capability
  you call**, not just the package's lowest possible version. `google-genai
  >=1.0` is technically satisfied by a release that predates the Interactions
  API the course's code calls, so pip can resolve an old version and the
  first real run fails with `AttributeError: 'Client' object has no
  attribute 'interactions'`, a runtime error that looks like a code bug but
  is actually a requirements-file bug. Confirm the version floor against
  current documentation or release notes, not memory, and comment the file
  with why the floor is where it is.
- **Setup instructions must say exactly how to get each credential**, not
  just "get an API key": the literal URL, whether a card is required, and
  where the value goes. "Get an OpenAI key" is not setup instructions,
  "go to platform.openai.com, sign up, click API keys, no purchase needed
  for the free trial, paste it into `.env` as `OPENAI_API_KEY`" is. This
  includes credentials the tool itself demands, not just the model
  provider's: a CLI or dev-server-driven demo can need its own platform
  key to authenticate its hosted UI (see the LangSmith/Studio trap above),
  and that one is genuinely easy to miss on a first pass, since nothing in
  the course's own code calls it, only the tool does when you actually run
  it end to end. The way to catch it is to run the CLI far enough to hit
  the point where it asks, not to reason from the code alone.
- **Check the provider's current SDK and API shape before writing code
  against it**, do not rely on training-data memory for a fast-moving API.
  A model name, a package name, or the recommended client method can change
  between when you last learned it and today. The same applies to a
  platform's own pricing when you tell an instructor something is free: a
  free tier's limits (trace volume, seat count, whether a card is required)
  are exactly the kind of fact that changes and should come from a current
  fetch of the provider's own pricing page, not memory, before you write
  "free, no card needed" into a guide.

## Reporting

Report what you measured, with numbers, and which interactive elements you
actually clicked through (not just which components exist). When something is
unverified, say which part and why, rather than implying the whole deck was
checked. If you introduced a regression, say so directly and name it. If a
fix came from a live-tested loop over several elements, say how many elements
the loop touched and confirm the same count landed in the source edit. End
every work session by saying what is pushed, what is only committed locally,
and what the next session should do first, and make sure the metafiles say
the same thing.

