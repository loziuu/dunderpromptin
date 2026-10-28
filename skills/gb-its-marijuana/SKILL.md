---
name: gb-its-marijuana
description: Diagnose step of the /gangbang workflow for bugs — reproduce the bug as a failing test, find the root cause, propose fixes. Use when /gangbang routes a bug, or when the user types /gb-its-marijuana.
argument-hint: <the bug>
---

# gb-its-marijuana

A bug starts with proof that it exists: a test that fails because of the
bug. It shows the bug is real and understood, and the same test proves the
fix at acceptance. Input: $ARGUMENTS, or the bug and criteria from the
/gangbang state file. Work from the project profile (gb-what-i-do-here-every-day).

## 1. Reproduce as a test

Start here: no full suite run first. (Under /gangbang, gb-im-going-to-die has already
checked that what the tests need is up.)

Write a test in the project's style that shows the bugged behaviour, at the
level the bug shows up (through the API if users hit it through the API).
Its data must be possible: something the business rules allow and the code
can actually reach, set up through the same paths a user would. A bug
reproduced only on impossible data isn't the reported bug.

Run it, together with the tests already around the buggy code (its test
class, the area's tests), not the whole suite. The new test must **fail,
and fail for the bug's reason**: read the failure and make sure it's the
wrong behaviour, not a setup error or a typo. Note any neighbouring test
that already fails: it may be this bug, another bug or a broken setup, and
must not be confused with the new test's failure. Show the user the test
and its failing output.

If you can't reproduce it, stop and say what you tried. Don't guess at a
fix for a bug you haven't seen.

## 2. Find the root cause

Follow it to where the behaviour goes wrong, not just where it shows. Ask
why that code is wrong: a missed case, a wrong assumption, a rule applied
in one place and not another. Check whether the same mistake exists
elsewhere (the same pattern, copied code, sibling features).

If the cause is a design flaw (the intended behaviour was never decided,
or two decisions contradict), say so: the work escalates to gb-would-an-idiot-do-that.

## 3. Propose fixes

Two options where there are two real ones, typically the minimal fix and
the fix of the cause, with trade-offs and your recommendation. Say what
else the fix touches and what other places share the mistake.

## 4. Hand off

Show the user the failing test, the root cause, and the fix options, and
stop for their choice. Under /gangbang, write the test's name and file, the
cause and the chosen fix into the state file; gb-dont-know-where-this-is-going works from that.
