---
name: gb-how-dare-you
description: Review step of the /gangbang workflow — run /code-review on the changes plus a check against the project's conventions, and decide whether the build loop is clean or goes back to implement. Use when /gangbang reaches its review step, or when the user types /gb-how-dare-you.
argument-hint: '<what to review, default: the changes>'
---

# gb-how-dare-you

Review the changes and decide: clean (on to the next step) or back to
gb-dont-know-where-this-is-going with a list of what to fix. Review finds and judges; it
doesn't fix. Under /gangbang this runs in a fresh subagent, so it reads the
code without the author's reasoning: judge what the code does, not what it
was meant to do. Fixes go through implement and test like any other change, so
nothing reaches the user untested.

## 1. What to review

Run every git command below from the git root (`cd <git root> &&` in the
same command): the paths in `Excluded` are repo-relative.

Under /gangbang the brief gives a snapshot; use it. Otherwise take a
snapshot of the code as it is now (the recipe is in
`~/.claude/skills/gangbang/SKILL.md`, "Snapshots"). Call it `S`. The working tree doesn't change while you review. Then the
range depends on the mode the brief gives:

- **Pass mode** (inside the build loop): if the state file has no
  `Reviewed` snapshot yet, `<Start>..S`. Otherwise (later passes, and
  later rounds too), only what changed since the last review: `<Reviewed>..S`, unless the brief
  gives another range (acceptance sent the code back for cleaning: review
  that range for what the user rejected). Also check, in the
  code, that the previous review's findings were really fixed: the full
  list is in its report (the brief gives the path), the log has one line
  each.
- **Final mode** (the final check): everything since Start, `<Start>..S`,
  whatever earlier passes covered. A fix can break something elsewhere.

Without /gangbang, review what $ARGUMENTS names, or the uncommitted
changes.

Only this work's changes are in scope. Code from before Start is out of
scope, even on the same branch: don't report findings about it. So are
the paths in the state file's `Excluded` (the user's own edits): leave
them out of the range (`-- . ':!<path>'`). If other changed files look
unrelated to this work, don't review them; list them in your report so the
main session can ask about them. The one
exception is old code this work breaks (a caller that now gets a different
value, a lock order the change introduces); that's this work's finding.

## 2. Review

**The generic review.** Run the built-in `code-review` skill, always
naming the level `high` and `--max-findings default` (it otherwise
reuses the last level and findings cap used). Its
default target is the uncommitted diff, which is not this range, so don't
leave it on the default: give it the files the range touches (`git diff
--name-only <from> S`, with the same `Excluded` pathspec) as its target,
and tell it the changes to review are `git diff <from> S` (with that
pathspec too), not the whole files. If it can't be pointed at that range, or can't run here (as a
subagent you may have no Skill tool, or be unable to start reviewers of
your own), review that diff yourself for correctness: logic errors, edge
cases, error handling, concurrency, security, and callers the change
breaks. Say in the report which way it ran. `code-review` starts
reviewers of its own: don't do a second general review on top. Your own
part is the next one.

**The project's conventions**, on the same range, which a generic review
doesn't know (from the project profile):

- Each piece sits in the right layer; the module boundaries hold.
- The API follows the project's conventions, or the break is written down.
- Security and validation are applied as in the rest of the project; no
  sensitive data goes where the profile says it must not.
- The docs that must change with the code did.
- Every acceptance criterion has its proof (all criteria, in every
  mode): a test on possible data, a measurement (command, baseline,
  result, runs) that meets the target for a measured criterion, or "manual" with exact steps
  for gb-fire-drill. Manual and measured criteria have no test by design;
  that's not missing proof.
- A criterion marked "code-read" really holds: read the file:line it cites
  and check the reasoning, or send it back as missing proof.

## 3. Triage

Judge every finding on its merits; a reviewer's comment is a claim, not an
order. Many aren't worth acting on: style the project's conventions don't
ask for, hypothetical problems that can't happen with how the code is
called, "consider" suggestions that add code without fixing anything. Drop
those without guilt; only real problems keep the loop going. The
exception is what acceptance sent back: the user already decided it's
worth fixing, so it's a Fix (or an Ask about how), never a Drop.

- **Fix**: real bugs, convention breaks, missing access checks or
  validation, missing proof. Each with the file, what's wrong, and what the
  fix should be.
- **Ask**: findings with more than one reasonable answer (a simpler
  design, a different shape for the API) where you can't tell which the
  user would want. Give the options and your recommendation; the answer
  becomes a Fix or a Drop. If the project already answers it, it's not an
  Ask. As a subagent, you can't ask the user: return these at the top of
  your report, and the main session asks.
- **Drop**: not worth fixing, with one line why. They're listed at
  acceptance, so the user can disagree.

## 4. Verdict

- **Fix list not empty** → back to gb-dont-know-where-this-is-going.
- **Asks open** → not clean yet: the main session asks the user, and each
  answer becomes a fix or a drop.
- **Fix list empty, no Asks** → clean: in pass mode, on to the final
  check; in final mode, on to checks and acceptance.

Report: the mode, the range reviewed and the snapshot `S`, the verdict,
then Asks, fixes requested, and dropped findings, one line each. Under
/gangbang, write it to the path the brief gives (under
`~/.claude/work/<project>/<slug>/`) and return a short summary. Don't
write the state file: the main session logs the fix list and sets
`Reviewed` to `S` either way.
