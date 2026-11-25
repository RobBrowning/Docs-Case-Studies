# AI in my QA workflow

**Author:** Rob Browning · **Period:** 13 to 25 November 2026 · **Status:** written 20 Nov 2026; [chapter 2](#chapter-2-test-agents-under-house-conventions) added 25 Nov 2026

## Summary

Over one week I put AI into three parts of my testing: writing and repairing automated tests,
reviewing pull requests, and running and checking manual QA on a ticket. Every piece followed the
same rule: **AI drafts, something independent checks, a human decides.**

To find out whether that rule earns its keep, I seeded defects on purpose and counted what was
caught.

| Strand | What I measured | Result |
|---|---|---|
| Playwright Test Agents | First-pass quality of plan, tests and heals | 16-scenario plan; 7 of 8 generated tests passed first time; healer fixed the root cause once and **refused** to change an expected value once |
| AI pull request reviewer | Seeded PR with 4 known defects | **4 of 4** caught plus 3 real extras, **0 false positives**, 53 seconds. A generic code-review plugin caught 3 of 4 |
| Independent result verifier | 3 runs (UI, API, live site), verdicts withheld from the verifier | Both seeded defects caught. Verifier disagreed with the tester **5 times**: every one a real evidence gap, **0 false positives** |
| Test Agents under house conventions (chapter 2) | A second agent run after writing the lessons into the generator's rules | Generator reused **every** existing page object method and marked **24** gaps honestly; consolidation was mechanical. Cost moved to the planner (**87 min**) and to tests nobody had run: one real timing bug found only by running them |

The most useful finding: **the AI checking the work caught more of my mistakes than the AI doing
the work caught of the software's.** In five places I would have signed off a pass that the
evidence did not support.

## Why I set it up this way

AI makes test output fast and confident. The risk in QA is not slow output; it is a **confident pass
that isn't true**. A false pass ships a defect and teaches the team to trust a green tick that means
nothing.

So I held to three design rules throughout:

1. **AI output is a draft.** Plans, generated tests, heals and reviews all go through a pull request
   and a human, against written project rules.
2. **Check with something that hasn't seen the answer.** A model shown a conclusion tends to find
   reasons it holds. The checker gets the evidence, not the verdict.
3. **Measure with seeded defects.** "It seems helpful" is not a result. A known defect either gets
   caught or it doesn't.

## Strand 1: Playwright Test Agents

**What:** Playwright's planner, generator and healer agents, driven by Claude Code, in my public
[playwright-qa-showcase](https://github.com/RobBrowning/playwright-qa-showcase) repo against
SauceDemo. Pull request #3, decision recorded in
[ADR 0003](https://github.com/RobBrowning/playwright-qa-showcase/blob/main/docs/adr/0003-playwright-test-agents.md).

**Set-up that mattered:** the scaffolded seed imported Playwright directly, so generated tests would
have bypassed my page objects and fixtures. I rewrote it to start from the repo's `loggedIn`
fixture and kept it out of CI so the test count stayed honest.

**What the agents got right**

- **Planner:** explored the live site, followed every rule I passed it (no seeded-defect users,
  `data-test` or role locators, third-party quirks asserted rather than "fixed"), and wrote a
  16-scenario plan. It flagged 3 scenarios as replacing no manual check, which I moved to the
  "deliberately not automated" table.
- **Generator:** 8 spec files; **7 of 8 passed first time.**
- **Healer, round 1:** a page-object locator had been broken on purpose. It confirmed the real
  value on the live page and fixed the one line in the page object: the root cause, not the
  symptom.
- **Healer, round 2:** an expected price had been changed to a wrong value ($39.99 against a live
  $29.99). It diagnosed the cause, confirmed locators and timing were fine, and **changed nothing**.
  The session asked me which price was right.

**What needed a human**

- The planner miscounted its own scenarios (17 reported, 16 written), and its save tool had no
  field for the manual-check note, so I moved those notes by hand.
- The generator put locators inline because no page object existed yet. I refactored them into a
  `ProductDetailPage`.
- One generated test failed on a real race: SauceDemo changes the URL before the detail view
  renders. Fixed with an `expectLoaded()` that waits for exactly one product.

**Lesson:** the healer stopped in round 2 **because its instructions told it to**, not by default. A
healer that fixes tests by editing expected values turns real defects green. That rule now lives in
the repo's `CLAUDE.md` and must be restated every time the healer runs.

## Strand 2: AI pull request reviewer

**What:** the official Claude GitHub Action reviewing every pull request in my
[cypress-qa-showcase](https://github.com/RobBrowning/cypress-qa-showcase) and Playwright repos, with
a QA-focused prompt: every acceptance criterion has a test, no fixed waits, no weakened assertions,
house conventions followed. It comments only; it cannot push, approve or merge.

**How I measured it:** pull request #10 in the Cypress repo carried 4 deliberate defects, including
a fixed wait, a weakened assertion and an untested acceptance criterion.

| Reviewer | Seeded defects caught | Extra real findings | False positives | Time |
|---|---|---|---|---|
| QA-tuned prompt | **4 / 4** | 3 | 0 | 53 s |
| Generic code-review plugin | 3 / 4 (missed the untested criterion) | n/a | n/a | n/a |

One of the extras was subtle: an "A to Z sort" test **could never fail**, because the default order
was already A to Z. A human reviewer skimming the diff would likely have passed it.

**Afterwards:**

- I dropped the generic reviewer and kept the QA-tuned one (ADR 0005).
- The first real, unseeded review (Playwright PR #7) came back "Ready for human review".
- Cypress PR #13 was a docs change written by Claude. The reviewer caught a formatting slip in it,
  and the fix and re-review took about 2 minutes. **The reviewer checks AI-written changes too.**

**Gotcha worth knowing:** the reviewer's token has to come from `claude setup-token` signed into the
subscription account, not the API console, or every review fails instantly with zero usage.

## Strand 3: qa-toolkit, a QA plugin with an independent verifier

**What:** a Claude plugin I built that takes one ticket through a fixed chain. It is a generic
version of a plugin I built and use at work. The chain:

1. A setup interview writes a profile for each site (environments, API route, roles, sign-in rules,
   quirks).
2. The ticket becomes numbered criteria (AC-1, AC-2, ...).
3. Each criterion is tested and written as an **evidence record of observations only**.
4. A separate verifier agent, with no browser and a blank context, gets the criteria and the
   evidence but **never the tester's pass or fail**. It decides for itself: Supported,
   Insufficient, Contradicted, Omitted, Open or Void.
5. Every disagreement with the tester is shown. A retest round closes the gaps, then it stops.

**How I measured it:** two demo tickets with one seeded failing criterion each, plus a live check.

| Run | Seeded defect caught | Verifier round 1 | Verifier round 2 |
|---|---|---|---|
| SauceDemo UI, 6 criteria | Yes: tax shown at 8%, ticket said 10% | 3 supported, 1 insufficient, 1 contradicted, 1 open | 4 supported, 1 contradicted, 1 open |
| Restful-Booker API, 7 criteria | Yes: price sent without decimals | 3 supported, 2 insufficient, 1 void, 1 open | 5 supported, 1 contradicted, 1 open |
| BBC home page, live, 1 criterion | n/a | 1 insufficient | 1 supported |

**The five disagreements, where I would have passed something the evidence didn't prove:**

1. "Cart badge goes up by one" passed on a badge reading 2. The count before the first add was never
   written down.
2. "Every booking has an id" was checked on 60 of 2,124 records.
3. A failed price-format check was right but unproven: parsed JSON cannot show trailing zeros, so
   only the raw response settles it.
4. "Unknown id returns 404" used the 404 as its own proof. Nothing showed the id didn't exist.
5. The scope was desktop only, but the evidence came from a 407px-wide browser pane.

Every one was **missing evidence, not broken software**, and all were closed with new observations
in round 2. The one exception was a recheck cut short when the demo site's session expired; it was
left as a decision for me rather than looped.

**Side benefit:** the "live data first" rule paid off. The public API reset its whole dataset
between rounds (2,124 bookings became 1,587), and the retest re-hunted its data instead of reusing
stale ids.

## Chapter 2: Test Agents under house conventions

**Date:** 25 November 2026 · **Repo:** [playwright-qa-showcase](https://github.com/RobBrowning/playwright-qa-showcase),
pull requests [#12](https://github.com/RobBrowning/playwright-qa-showcase/pull/12) and
[#14](https://github.com/RobBrowning/playwright-qa-showcase/pull/14)

### The question

Chapter 1 ended with a refactor: the generator put locators inline and I moved them into page
objects by hand. Repeating "use the page objects" in every prompt is easy to forget and does not
scale. So: **if the lessons are written into the agent's own instructions, does the output arrive in
house style, and where does the remaining cost go?**

### What I changed first (PR #12, ADR 0006)

Reading the generator's definition turned up a constraint I had missed: **it can read files and
write a test file, nothing else.** It can reuse page objects; it cannot create or extend them. So
the rule could not be "use page objects". It had to be a hand-off:

1. **Generator rules.** A "House conventions" section in the agent file: import from the project's
   fixtures, reuse existing page object methods, keep assertions in the spec, and where a page
   object is missing, write the step inline and mark it `// TODO(page-object): Page.method`.
2. **Consolidation step.** Before a PR, the main Claude Code session (not an agent) merges the
   drafts into one spec, turns every TODO into a real page object method, and removes duplicates.
3. **Reviewer rules.** The AI PR reviewer treats a leftover TODO, a duplicated locator or a direct
   Playwright import in a spec as a Blocker.
4. **Blockers block.** The branch ruleset already required every review thread resolved before
   merge. Each reviewer finding is a thread, so a Blocker can't merge unnoticed.

Rejected: giving the generator Edit and Write tools. That would let an agent change shared code
before anyone looks at it, and add one more customisation to reapply on every Playwright upgrade.

### The run: SauceDemo's burger menu (PR #14)

A new area with no tests and no page object: open and close, All Items, About, Logout and Reset
App State.

| Step | Time | Output |
|---|---|---|
| Planner | **87 min** (63 to first save) | 8 scenarios, 10 observed quirks, 1 flagged as replacing no manual check |
| My plan review | not timed | 4 changes: two overlapping scenarios merged, About checked by link only, a coordinate click dropped, a missing strategy row added |
| Generator | **81 s** | 6 draft files, **24** `TODO(page-object)` markers |
| Consolidation | about 5 min | 1 spec, 1 new `MenuPage`, 6 tests |
| CI and AI review | minutes | All green; reviewer 0 findings, twice |

### What worked

- **The generator followed every rule.** It imported from the fixtures, started from the logged-in
  fixture, named the manual check each test replaces, and used role or `data-test` locators only. It
  reused every existing method that fitted, across four page objects, and inlined nothing they
  already exposed. **It invented no methods.**
- **The TODOs made consolidation mechanical.** All 24 pointed at one missing page object. Adding
  `MenuPage` with five actions resolved every one. In chapter 1 the same step was an unplanned
  refactor.
- **The planner reused, too.** It named the existing methods to use, pointed at existing login and
  cart tests instead of re-planning them, and left out a fifth menu item nobody asked for.
- **Quirks stayed quirks.** "Reset App State" clears the cart but leaves stale Remove buttons until
  a reload. The tests assert that and comment it, rather than treat it as a bug to fix.

### What still needed a human

- **The planner was slow.** 87 minutes, most of it two browser calls that hung for the full 30 minute
  tool limit. The brief was broad (five pages, keyboard focus, back navigation), which gave it more
  places to stall.
- **The planner's save tool forces a format.** It has no field for "manual check replaced", so it
  wrote that line in as a test step with nothing to check. Same as chapter 1. It was fixed by hand
  twice, because the second save undid the first fix.
- **My plan review changed four things,** mostly about upkeep rather than correctness: a duplicate
  scenario, a test that would make CI depend on an external site, a click at fixed coordinates that
  would have landed off-screen on the mobile project, and a missing note that these tests have no
  Cypress twin yet.
- **A real timing bug, found only by running the tests.** "Escape closes the menu" failed. The menu
  items count as visible while the panel is still sliding in, but the menu ignores Escape until it
  has finished opening. The planner reported "Escape closes the menu" without seeing the timing,
  and the generator wrote a test it never ran. The fix was a wait in the page object, not a weaker
  assertion: 35 of 35 passes on repeat, desktop and mobile.
- **The reviewer missed a duplicate.** It checked every rule and found nothing, twice. But the new
  spec repeated an error message already asserted in the login tests, which its own prompt lists as
  a Minor. I caught it in my review, and it now lives in one shared helper.

### What changed between chapters

| | Chapter 1 (13 Nov) | Chapter 2 (25 Nov) |
|---|---|---|
| Generator rules | Passed in the prompt | Written into the agent file (ADR 0006) |
| Page object reuse | Inline locators, refactored by hand | Every fitting method reused, gaps marked |
| Consolidation | Unplanned refactor | A defined step: add one page object, merge, dedupe |
| First-run failures | 1 of 8 (a real race) | 1 of 6 (a real timing bug) |
| Where the cost sits | Generator output | Planner time, and tests nobody had run |

**Lesson:** writing the rules into the agent moved the problem rather than removed it. The
generator is now cheap and predictable. What is left is the part no agent did here: **running the
tests**. Both chapters' only real failure was found that way. An agent run isn't finished until a
human or CI has actually run its tests.

## What I'd tell another QA lead

- **Separate the doer from the checker, and hide the answer from the checker.** That single design
  choice produced the most value all week.
- **Write the rules down and pass them every time.** The agents followed project rules well when
  given them explicitly, and not otherwise. Better still, write them into the agent itself
  (chapter 2): then nobody has to remember.
- **Read the agent's tool list before you write its rules.** The generator couldn't create page
  objects, so the useful rule was a hand-off, not a demand.
- **Keep planner briefs narrow.** One area per run, or a time limit. A broad brief cost 87 minutes.
- **Seed defects to measure.** It turned "seems useful" into 4 of 4, 0 false positives, and gave me
  a fair comparison between two reviewers.
- **Cap the loops.** Two verification rounds, then a human decision. Looping past that looks
  diligent and adds nothing.
- **Keep sign-in and secrets out of the AI's hands** on anything that isn't your own test
  environment.

## Limits of this evidence

- Small samples: one seeded PR, two seeded tickets, one live check. The direction is clear; the
  rates are not yet statistics.
- The seeded defects were written by me, so they test what I thought to seed.
- In the two demo runs the verifier ran as a general-purpose agent loaded with the verifier's
  instructions. The live check used the installed plugin agent.
- The plugin has not yet been run on a site with a separate baseline environment.
- Chapter 2 is one run on one area. Times are single measurements, and consolidation was timed from
  file timestamps, not a stopwatch.

## What's next

- Run qa-toolkit on a real project with a staging environment.
- Done since chapter 1: [Testing an AI chatbot](../testing-an-ai-chatbot/README.md) and
  [Automated evals and a QA plugin on the same AI chatbot](../evals-and-qa-plugin-on-an-ai-chatbot/README.md).
- Test Agents, chapter 3: a narrow planner brief with a time limit, to see whether the planner's
  cost drops and its quality holds.
