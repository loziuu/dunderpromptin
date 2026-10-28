---
name: gb-how-the-turntables
description: Test step of the /gangbang workflow — prove each acceptance criterion with a test on realistic data and run the project's test suites; for refactors, pin current behaviour first. Use when /gangbang reaches its test step, or when the user types /gb-how-the-turntables.
argument-hint: <what to test, or what acceptance sent back>
---

# gb-how-the-turntables

Acceptance asks for proof that the work is right. This step produces it: a
test for every acceptance criterion, on data that could really occur, all
passing. Input: $ARGUMENTS, or the /gangbang state file (the criteria are
there or in the design doc it names; the log says what acceptance sent
back). Work from the project profile (gb-what-i-do-here-every-day).

Run every git command in this skill from the git root (`cd <git root> &&`
in the same command): the paths in `Excluded` are repo-relative.

## 1. Check the environment once

Check once that what the tests need is up: the list in
`~/.claude/work/<project>/env.md` if gb-im-going-to-die wrote one, otherwise the
profile's, along with any checks the user's CLAUDE.md asks for. Only check;
don't start or create anything here (that's gb-im-going-to-die's job, and it may need
the user's permission). If something is down, stop and report it, with the
command to start it. Don't retry in a loop and don't fall back to only
compiling: untested code must not look tested.

## 2. A test for each criterion

For each acceptance criterion, find or write the test that proves it, in
the project's test style and at the level that actually exercises the
behaviour (through the API if the criterion is about the API, not a unit
test of a helper). Cover the edge cases in the criteria: access, invalid
input, empty states.

By kind:

- **Bug:** the reproducing test from gb-its-marijuana is already there and must
  now pass. Add tests for the other places that shared the mistake, and one
  that the surrounding behaviour didn't change.
- **Refactor, before the change** (pin mode): write tests that pin the
  current behaviour of the code being restructured, and make sure they
  pass on the code as it is now. They are the proof that nothing changed.
  If the code is still at Start apart from the pinned tests you just
  wrote and `Excluded` (`git diff --quiet <Start> -- . ':!<pinned test>'…
  ':!<Excluded path>'…` succeeds, and `git ls-files --others
  --exclude-standard` with the same pathspecs lists nothing), skip
  section 4. Otherwise (files the
  user said belong to this work, or a later round), the pinned tests
  must also pass at Start: run them there as in section 4.
- **Measured criteria** (faster, smaller, fewer queries): no assertion
  test; record the command, the baseline at Start, the result now, and
  how many runs each, on the same machine and data. Measure the baseline
  in the "before" worktree (section 4) unless an earlier report in
  `<slug>/` already has it; in pin mode with the code still at Start,
  measure it in the real tree. A result that misses the criterion's
  target is a code failure, like a failing test (section 3), in every
  mode.

**Check the test data is possible.** For every test, ask:

- *Business:* could this situation really happen? (An order shipped before
  it was placed, a user in no account, a negative price: if the rules
  forbid it, a test built on it proves nothing.)
- *Implementation:* can the code actually reach this state? Prefer setting
  data up through the same public paths a user would (the API, the
  commands) rather than inserting rows directly, so validation and rules
  apply. If you must insert directly, make sure the rows are ones the code
  could have produced.

Fix any test whose data fails either check, existing ones included if the
work depends on them.

Keep a list: criterion → test (name and file) → the data in a few words
and why it's realistic → how it did before the change (below). It goes in
your report; the main session copies it into the state file's Proof
section, and acceptance uses it as its proof. A criterion that truly can't be tested
automatically (how a screen looks) gets "manual" and exact steps to check
it. One that no test can prove, or that a project rule keeps from being
tested (no assertions on message strings), but that reading the code can
settle, gets "code-read" with the file:line and the reasoning.

## 3. Run

What to run depends on the mode the brief gives:

- **Pin mode** (a refactor's first step): the pinned tests and the tests
  around the code being restructured.
- **Area mode** (inside the build loop, the default): the proof tests,
  the tests covering the area the change touches (the profile's
  "Running and testing" says which tests cover which area; if it doesn't,
  take the test classes and modules next to the changed code, and name
  them in the report), and the quick checks (compile, type check, linters
  if they're fast). Not the full suite: that's the final check's job.
- **Final mode** (the final check, once the build loop is clean): every
  suite, linter and type check the profile lists. Write and change no
  tests; skip sections 2 and 4. Only run and report, plus re-run each
  measured criterion's command for a current "now" (the baseline at
  Start stays).

Without /gangbang, run every suite, linter and type check the profile
lists.

For each failure, decide whose fault it is:

- **The test** (wrong setup, wrong expectation you wrote): fix it here.
  Never a refactor's pinned tests after pin mode: they must pass
  unchanged, so their failure is the code's or the design's.
- **The code**: don't fix it here. Under /gangbang, it goes back to
  gb-dont-know-where-this-is-going with the failures (the build loop): list
  them in your report.
- **The design or diagnosis**: stop and say so instead of bending the test
  to fit.

Leave out the paths in the state file's `Excluded`: don't test them, and
don't count a failure that only they cause as this work's (say so in the
report). Don't edit them either: if a test belongs in one, put that at the
top of your report as a question; the main session asks the user
(/gangbang, "Excluded") and runs you again.

## 4. Check each proof fails without the change

Once everything passes: a test that passes with or without the change
proves nothing. Run the proof tests on the code as it was at the state
file's `Start`, in a separate worktree so the real working tree is never
touched. On a later pass, only the proof tests whose Proof line has no
"before" result yet, or whose files changed since the state file's
`Reviewed` snapshot (`git diff --name-only <Reviewed>`, plus new untracked
files): nothing changes between the last check and that review.

```
before=~/.claude/work/<project>/<slug>/before
git worktree remove --force "$before" 2>/dev/null; git worktree prune
git worktree add --detach "$before" <Start>
# copy the proof tests (and test helpers they need) into it, run only them
git worktree remove --force "$before"
```

A fresh worktree lacks the ignored files the build needs (dependencies,
build output, `.env`). Reuse what the real tree has without creating
anything: symlink `node_modules` or the env file, point the build at the
existing dependency cache (Gradle, Maven, Cargo, pip caches are shared
already). Installing or downloading anything there needs the user's
permission, so as a subagent, don't: mark those tests **couldn't run
before**, with why, rather than skipping them silently.

In a compiled language, one test using the new API stops every copied
test from compiling: copy the new-API tests separately, or leave them out
and mark them **new API** directly.

Don't run it alongside another test run if they share a database. Record
per test:

- **fails before**, for the criterion's reason (read the failure): good
  proof.
- **passes before**: it doesn't catch the problem. Make it sharper (data
  that triggers the difference, a check on the actual outcome); if it
  can't be, mark it **weak** with why (e.g. "test DB is UTC, so the time
  zone bug can't show").
- **new API**: it can't compile before because it uses what this work
  added. Fine for new behaviour; say so.

Skip tests that already showed they fail: gb-its-marijuana's reproducing test.
"Behaves as before" criteria and a refactor's pinned tests are the
reverse: they must **pass** both before and after.

## 5. Report

The mode and exactly what ran (which suites or test classes), pass/fail
counts per suite, code failures for gb-dont-know-where-this-is-going, and
the criterion → test → data → before list. Never let a partial run read
like a full one. End with a snapshot of the tree as you leave it (recipe
in `~/.claude/skills/gangbang/SKILL.md`, "Snapshots") and name it in the
report.

If you run as a subagent, you can't ask the user anything: put questions,
a service that is down, or a design problem at the top of your report, and
stop there instead of guessing. Write the full report to the path the
brief gives (under `~/.claude/work/<project>/<slug>/`) and return a short
summary; don't write the state file, the main session does.
