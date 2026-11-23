# Designing a round

## Challenge types
Debug family:
1. **Fix it**: broken code with a bug report (or a failing test at level ≤4). Find the root cause, fix, add a regression test.
2. **Refactor it**: messy but working legacy code with passing tests. Improve the design without changing behavior.
3. **Harden it**: working code that falls over in production. Make it handle concurrency, failures, retries or load.

Build family:
4. **Build it**: implement a component against a given interface and tests (LRU cache, rate limiter, retry with backoff, job scheduler, money type…).
5. **Feature it**: add a feature to a small app: endpoint/handler, validation, persistence, error handling, tests.

When the player picks "Debug it" choose among 1–3, "Build it" among 4–5, rotating so the same exact type doesn't repeat back to back. Every round should test a different skill from the previous one.

## CHALLENGE.md

The player reads this cold, with no context from the chat. They should finish the first screen knowing what they're building or fixing, which files to touch, and how they'll know they're done. Write it the way a good colleague hands over a ticket at the desk: task first, background second, details last.

### Layout (in this order)

```markdown
# <Title in plain words>

**Your task:** <one or two sentences: the verb (implement / fix / refactor / harden / add), the thing, the outcome.>

<topic> · level N (<rank>) · <Fix it | Build it | ...> · ~NN min

## What to do
1. <Concrete step, naming the file.> e.g. "Implement `TopCounter` in `src/main/java/gym/search/TopCounter.java`."
2. ...
3. Write tests for <the parts that are yours to cover>.

Keep the public method signatures as they are. Add whatever private code you like.

## Done when
- [ ] `<build_cmd>` passes
- [ ] <each observable outcome, one line>
- [ ] Your own tests cover <what>

## Why this exists
<3-5 short sentences: who uses it, what goes wrong or what's needed. Real-ticket tone. Load numbers and other facts go here, said plainly.>

## How it should behave
<Group by class/endpoint under ### headings. One rule per bullet, one idea per sentence.
Give a tiny input → output example for anything with more than one moving part.>

## Run, submit, rules
- Run: `<build_cmd>`. Right now <what the visible tests do and why, e.g. "the 2 given tests fail because nothing is implemented yet">.
- Submit: say `submit` in chat, plus a few lines on your approach, tradeoffs, and what you'd do with more time. I read the files myself.
- Stuck on what the requirements mean? `ask <question>` (free). Want help with the code? `hint` (costs XP).
- Allowed: <from profile>. Not allowed: <from profile>.
```

### Writing rules
- **Task first.** The "Your task" line is the most important line in the file. If the player only reads that and "What to do", they should be able to start.
- **Name the files.** Say exactly which files to edit and which to leave alone. Don't make them hunt for "the signatures in package X".
- **Define terms in the same breath.** Explain anything that's not everyday vocabulary the first time it appears. "Ties (items with the same count) are ordered by the comparator you pass in, smallest first." Not a bare "tie-break comparator". Same for gym jargon: don't print "Placement round" or "Journeyman" without a word on what it means, or leave it out.
- **Examples beat prose.** `"  Running  SHOES "` → `"running shoes"` says more than a sentence about whitespace runs. One small example per tricky rule.
- **One idea per sentence, one rule per bullet.** If a bullet has two "and"s and a "which", split it.
- **Plain words.** "copy" not "snapshot", "in the same file as" not "colocated". If you do use the precise term, explain it: "a copy: later `add` calls don't change it, and callers can't modify it".
- **Facts, not hints.** State constraints as facts ("`trending(10)` runs on every search page view; there are ~500k distinct terms") and leave the design to the player. Plain language must never turn into pointing at the solution.
- **Short.** Aim for one screen for levels 1-4, two at most above that. Cut anything the player doesn't need to do the task.
- **Smell test.** Before writing the file, read it as the player opening it tomorrow with no memory of the chat. Can they say what the task is in one sentence after 30 seconds? Is every term they haven't seen defined where it appears? No em dashes. No "leverage", "facilitate", "in order to".

At level 5+ some requirements stay unspecified on purpose (the player earns credit for asking). Leave them out cleanly. Don't write vague sentences that look like an accident.

Never point at the bug or hint at the solution in the brief, file names, class names, comments or commit messages. `ReservationBugService` or `// TODO fix race` gives the game away.

## Starter code quality (strict)
- Builds on the stated versions. You verify by actually building, not by eyeballing.
- Visible tests behave exactly as stated. A test you say fails must fail for the intended reason only.
- No flaky or unspecified behavior: no hash-ordering assumptions, no timing assertions with tight margins, no reliance on thread scheduling. Concurrency tests use latches, barriers or executors deterministically, and are run several times during verification to confirm they're stable.
- Looks like real production code, not a contrived puzzle: real domain names, realistic structure, plausible history (a slightly crufty helper, a reasonable-looking but wrong assumption).
- Size: roughly ≤5 files and ≤250 lines of challenge code at levels 1–6 (scaffold boilerplate doesn't count). Larger at 7+ if the scenario needs it.
- If the challenge needs an extra dependency, add it to the scaffold and mention it in the brief.

## Hidden tests
2–5 tests on edge cases and failure modes the player may miss: boundaries, duplicates, concurrent calls, failure mid-operation, invalid input, rollback. They test behavior through the public API the brief defines, so a correct solution with a different internal design still passes. Store with a `NOTES.md` listing what each test probes, the intended root cause or core idea, and what an excellent solution looks like.

## Level fit
- 1–2: give all tests; spec is complete.
- 3–4: some tests given, player writes the rest; bugs surface as failing tests.
- 5–6: symptom only, no failing test; player must reproduce first. Grade Testing heavily on whether they did.
- 7–8: plan the stages up front in hidden `NOTES.md` (what changes after stage 1, what failure gets injected after stage 2).
- 9–10: give a problem and constraints, not a skeleton. Play product owner and on-call as the round progresses.
