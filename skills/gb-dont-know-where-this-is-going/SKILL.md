---
name: gb-dont-know-where-this-is-going
description: Implementation step of the /gangbang workflow — build an approved design, bug fix, refactor or small change following the project's architecture and conventions. Use when /gangbang reaches its implement step, or when the user types /gb-dont-know-where-this-is-going.
argument-hint: <what to build, or what acceptance sent back>
---

# gb-dont-know-where-this-is-going

Build what was approved, nothing more. Input: $ARGUMENTS, or the /gangbang
state file: for a feature, the design doc it names; for a bug, the cause
and chosen fix from gb-its-marijuana; for a refactor or small change, the
criteria triage set. On a later build pass, the log has the findings
gb-how-dare-you or the failures gb-how-the-turntables sent back; on a later round, what
acceptance rejected. Fix exactly those. Work from the project profile (gb-what-i-do-here-every-day); if
it isn't loaded, load it first.

## How

- Find the existing code closest to this change and follow its shape: the
  same layers, file layout, naming, error handling and tests. New code
  should read like the code around it.
- Respect the profile's layers of abstraction: each piece goes in the layer
  that owns it (no storage calls from the HTTP layer if the project keeps
  them apart, and so on).
- Apply the project's security and validation the same way its other
  features do: access checks, what outsiders see, where input is checked.
  A change that skips them is not done, however well it works.
- Schema changes go through the project's migration mechanism.
- Update the docs the profile says must change with the code, in the same
  change, and the design doc's status if there is one.

## By kind

- **Bug:** make the reproducing test from gb-its-marijuana pass with the chosen
  fix. Don't change that test to make it pass; if it turns out to be wrong,
  stop and say so. Fix the other places diagnosis found with the same
  mistake, if the user agreed to.
- **Refactor:** behaviour stays the same; the tests pinned in the previous
  step must pass unchanged.

## Rules

- When there are several reasonable ways to build something and you
  aren't sure which the user would pick, stop and ask, with the options,
  trade-offs and your recommendation. Don't ask what the project already
  answers (its conventions, the design, an earlier decision); follow that.
  If it's a real design question, that's escalation to gb-would-an-idiot-do-that (see
  /gangbang). Write the answer into the design doc or the state file.
- Don't add what wasn't asked for: no extra endpoints, options or refactors
  on the side. Mention them instead.
- Don't edit a path listed in the state file's `Excluded` (the user's own
  changes) before asking: they commit theirs first, or it becomes part of
  the work (see /gangbang, "Excluded").
- Before handing off, make sure everything builds and type-checks (the
  profile's commands), so gb-how-the-turntables starts from code that compiles.
- End with a short list of what changed, by file.
