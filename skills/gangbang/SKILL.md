---
name: gangbang
description: Runs a request through a full workflow — project profile, triage, design or diagnosis, a build loop of implementation, tests and code review, simulated manual checks, acceptance — looping back until the user accepts. Only when the user types /gangbang.
argument-hint: '<what to build or fix> | resume <slug> | drop <slug> | clean | (nothing: list open work)'
disable-model-invocation: true
---

# gangbang

Arguments: **$ARGUMENTS**

You drive the request through steps. Each step is its own skill; load it
with the Skill tool and follow it (or hand it to a subagent, see below),
then come back here. The steps hand work to each other through files, not
through chat, so the workflow survives a new session and every step can
see what the others decided.

If the work may change this workflow's own skills (`gangbang` or `gb-*`
under `~/.claude/skills/`), the steps must not follow half-edited
instructions. That's decided before triage, from the request alone, and
scope can grow later, so do this for every run whose git root is
`~/.claude`. At the very start, before the clean-tree check (see "Start
commit": a stash there would roll them back), copy all of these skills as
they are on disk now, the version you're running (`mkdir -p
~/.claude/work/<project>/<slug>/skills`, then `cp
~/.claude/skills/<name>/SKILL.md
~/.claude/work/<project>/<slug>/skills/<name>.md`), and read those
copies instead of loading the skills or their files: in the main session,
in every brief, and wherever one skill points to another's `SKILL.md`. Record
that in the state file's `Skills` line. On `resume`, see Arguments.

## Where progress lives

Progress lives outside the repo, in `~/.claude/work/<project>/`:
`<project>` is the git root's absolute path with every `/` replaced by `-`
(e.g. `-home-me-repositories-shop`). In it:

- `<slug>.md`: a state file, one per request. A state file is a top-level
  `*.md` with a `Next:` line; `env.md` (gb-im-going-to-die's settings) is
  not one.
- `<slug>/`: everything else the request produces: test and review
  reports, screenshots, the temporary "before" worktree. Nothing else goes
  at the top level.

## Arguments

- **nothing**: list this project's unfinished work (name, kind, next step,
  round, last log line) and ask which to resume. If there's none, ask what
  to work on.
- **`resume <slug>`**: continue `<slug>.md` from its `Next`; say which step
  you're resuming at. First, in this order:
  1. If `Skills` is set (or `<slug>/skills/gangbang.md` exists), read the
     copy of gangbang and follow it instead of this file. If `Next` is
     `done`, refresh the copies from the installed skills first (the
     accepted version is what's installed now) and log it.
  2. If Decisions says the user declined the profile file, analyze the
     project again without asking (gb-what-i-do-here-every-day, "No").
  3. Check what changed while no session was running: take a fresh
     snapshot and list `git diff --name-only <Reviewed, or Start if
     empty> <new>`, leaving out `Excluded`. A path alone can't tell the
     work's changes from the user's (both may edit one file), so show the
     whole list, marking the paths a step logged after that snapshot
     touched (with `Next: done`, none: nothing of the work's comes after
     the final check), and ask which ones the user edited. Handle those as
     for a dirty tree at the start (commit, which moves Start as under
     "Excluded"; leave as unrelated; or count as part of the work), except
     a file the work itself changed since Start (`git diff --name-only
     <Start> <new>` and the Log): excluding or committing it would take
     the work's own changes out of scope with the user's, so such a file
     can only count as part of the work. If the list is empty, say so and
     go on. If anything was counted in or Start moved, and `Next` is the
     final check, checks or acceptance, set `Next` back to the build loop
     (test in area mode, then review) and log why: that code hasn't been
     tested or reviewed yet.
  4. If `Running` is set, a subagent's result may have been lost with
     the old session. Check whether the code changed since, leaving out
     `Excluded` (`git diff --quiet <old> <new> -- . ':!<path>'…`). For a
     review, `<old>` is the snapshot in `Running`; for a test report that
     exists, the end snapshot it names (test steps write tests, so the
     tree moves under them). Unchanged: log each report `Running` names
     that exists, as usual, and run again only the subagents whose report
     is missing. Changed: discard those reports and run the whole step
     again (both halves, for the final check) on the new snapshot.
  5. If `Next` is `done`, the user is reopening finished work: set
     `Reviewed` to the fresh snapshot, so the round's reviews start from
     there, ask what's wrong, then start a new round as when acceptance
     fails (see Acceptance), with the same Start.
- **`drop <slug>`**: abandon it. Show its last log lines, ask to confirm,
  then remove a leftover "before" worktree if there is one (`git worktree
  remove --force ~/.claude/work/<project>/<slug>/before; git worktree
  prune` in the repo), and
  delete its state file and its folder. Code changes stay; the user
  decides about those.
- **`clean`**: list the finished state files (`Next: done`) with their
  dates, ask to confirm, delete them and their folders.
- **anything else**: a new request. If an open state file already looks
  like the same request, ask whether to resume it instead.

## Routes

Not every request needs every step: triage decides the route.

| Kind | Route |
|---|---|
| Feature | gb-would-an-idiot-do-that (design) → ✋ → build loop → final check → checks → ✋ acceptance |
| Bug | gb-its-marijuana (diagnose) → ✋ → build loop → final check → checks → ✋ acceptance |
| Refactor | gb-how-the-turntables (test), pin mode → build loop → final check → checks → ✋ acceptance |
| Small change | build loop → final check → checks → ✋ acceptance |

The other steps: gb-what-i-do-here-every-day (profile), gb-diversity-day
(triage), gb-im-going-to-die (environment), gb-dont-know-where-this-is-going
(implement), gb-how-dare-you (review), gb-fire-drill (manual checks).

## The state file

`~/.claude/work/<project>/<slug>.md`, `<slug>` a short kebab-case name for
the request. Create it at the start:

```markdown
# <request>
Project: <git root>
Start: <commit the work started from, see below>
Kind: <set by triage>
Route: <set by triage>
Design doc: <path, if gb-would-an-idiot-do-that writes one>
Round: 1
Build pass: 0
Reviewed: <snapshot gb-how-dare-you last reviewed up to, see below>
Running: <subagents running now, each with its report path, plus the snapshot of the code they run on, e.g. "<slug>/test-r1p2-final.md, <slug>/review-r1p2-final.md @ 3f2a9c1, 14:05"; empty otherwise>
Excluded: <paths in the tree that aren't this work's, see below; empty otherwise>
Skills: <if the work edits this workflow: "<slug>/skills/, follow these copies, not the installed skills"; empty otherwise>
Next: gb-what-i-do-here-every-day

## Acceptance criteria
<set by triage; gb-would-an-idiot-do-that may refine them into the design
doc, then this says "see design doc">

## Proof
<one line per criterion, kept current from every test report:
criterion → test (name, file) or manual steps or code-read file:line →
data and why it's realistic → before: fails / passes (weak, why) / passes
as required (unchanged behaviour) / new API / couldn't run.
A measured criterion: command → target → baseline at Start → now → runs each.
A gb-fire-drill check: steps → screenshot path → ✅/❌ → snapshot checked>

## Decisions
<answers the user gave along the way, so nobody asks again>

## Log
- <date> round 1: started
```

Update `Next` and append to the log after every step: one line saying what
the step did or found.

**Only the main session writes the state file.** Subagents read it, write
their full report into `<slug>/`, and return a summary; you copy what
matters into the state file (log line, `Reviewed`, Proof, the fix list).
Two writers on one file lose each other's changes, and the final check
runs two subagents at once.

**Start commit.** This work's scope is everything that changed since it
started, and nothing from before. If the project is `~/.claude`, first set
up `.git/info/exclude` as "Excluded" below describes, so Claude Code's own
files never show up as dirty (never ask to commit or stash those), and
take the skill copies (see above). Then,
before creating the state file, check the working tree (`git status
--porcelain`). If it isn't clean, say which files and ask, for each,
whether to commit it, stash it, count it as part of this work, or leave
it as unrelated (it goes into `Excluded`); unanswered, it would be in the
scope. Don't offer stash for this workflow's own skills (`gangbang`,
`gb-*`): it would swap the installed workflow back to an older version
mid-run. After a commit or stash, check again. Only then set `Start` to
`git rev-parse HEAD`, so a commit they just made stays out of the scope. A
repo with no commits yet has no HEAD, so ask the user to make one first.

The work's changes are `git diff <Start>` (commits since Start, uncommitted
and staged changes) plus `git ls-files --others --exclude-standard` for new
files. Start stays the same across build passes and rounds (the one
exception is under "Excluded"). If it's no
longer an ancestor of HEAD (`git merge-base --is-ancestor <Start> HEAD`
fails, e.g. after a rebase), stop and ask the user which commit to use
instead. A state file from before this rule has `Base:` and no `Start:`:
ask the user which commit the work started from.

**Excluded.** The user may keep editing the tree while the work runs.
Files changed since Start that this work didn't touch (the user's own
edits, unrelated WIP) go into `Excluded`, after asking the user when it
isn't obvious. Every step leaves them out: no tests for them, no findings
about them, not in the acceptance diff stat. A pathspec hides a whole
file, so before the work edits a path in `Excluded`, stop and ask the
user to commit their own changes to it first, or to count it as part of
the work; either way, take it out of `Excluded`. If they committed, check
that the new commits touch only paths in `Excluded` (`git diff
--name-only <Start> HEAD`; if not, ask), then set `Start` to the new HEAD
and log it, so their commit stays out of the scope as at the start.
Also clear `Reviewed`: the old snapshot holds their file as it was then,
and a review from it would count their later edits as this work's. Whenever this workflow
creates or updates the profile (`.claude/gangbang.md`), add it to
`Excluded` too: the user reviews it, not code-review.

If `~/.claude/work/<project>/` is inside the git root (the project is
`~/.claude`), snapshots would hash files that aren't code at all: the
workflow's own reports and worktrees, and what Claude Code itself keeps
writing (`projects/`, `sessions/`, `history.jsonl`, `.credentials.json`,
…), which may hold secrets and change every turn. Before the clean-tree
check, add `/work/` to `.git/info/exclude` (local, never committed; from
the git root), ask the user which other paths Claude Code writes aren't
the work's target, top-level or not (e.g. `/projects/`,
`/skills/synced/`), and add those too, repo-relative. Nothing of the
user's or the work's goes there: the profile and the user's own files
stay visible to git, and `Excluded` keeps them out of the work by
pathspec.

**Snapshots.** To name the code exactly as it is right now, uncommitted and
new files included, make a snapshot commit. It touches neither the
branch, the index nor the working tree:

```
tmp=$(mktemp -u); cp "$(git rev-parse --git-path index)" "$tmp" \
  && GIT_INDEX_FILE=$tmp git add -A \
  && GIT_INDEX_FILE=$tmp git -c user.name=gb -c user.email=gb@local \
     commit-tree "$(GIT_INDEX_FILE=$tmp git write-tree)" -p HEAD -m gb-snapshot; rm -f "$tmp"
```

Run it, and every other git command in this workflow, from the git root
(`cd <git root> &&` in the same command, or `git -C <git root>`): paths in
`Excluded` are repo-relative, and from a subdirectory `.`, `':!<path>'`
and `git ls-files --others` silently match the wrong files. It prints the snapshot's sha. Starting from a
copy of the real index keeps tracked files that `.gitignore` matches. `git diff <A> <B>` between Start and
snapshots then shows exactly what changed, new files included. gb-how-dare-you
records the snapshot it reviewed in `Reviewed`, so the next pass reviews
only what changed since. Snapshots aren't on any branch and git may clean
them up after a couple of weeks; if one is gone, review from Start.

## Steps

1. **gb-what-i-do-here-every-day**: makes sure you know the project before anything else.
   Every later step works from that profile.
2. **gb-diversity-day**: scans the code the request touches, classifies it and
   proposes the route and acceptance criteria. **Always stops** for the
   user's approval.
3. **gb-im-going-to-die**: works out what the tests need and gets it running, by the
   user or by you, as the user chose for this project. Runs in the main
   session, since it may need to ask. If a test step later reports
   something down, run gb-im-going-to-die again before going on.
4. **The route**, step by step. Stop where the route has ✋: after gb-would-an-idiot-do-that
   (approve the design) and after gb-its-marijuana (approve the failing test,
   cause and fix). The build loop runs without stopping, unless a step
   itself needs the user (a service that is down, a choice only they can
   make). Tell the user in one line when each step starts.
5. **Final check**, below: the full test suite and a review of everything
   since Start.
6. **Checks**: after the final check, run gb-fire-drill for every
   criterion still "manual", and for every one gb-fire-drill already
   checked whose code changed since the snapshot its Proof line names
   (`git diff --name-only <that snapshot> <final check's snapshot>`). It asks the user's permission before launching
   anything.
7. **✋ Acceptance**, below.

## The build loop

```
implement ──→ test (area) ──→ review (pass) ──clean──→ final check ──clean──→ checks → ✋ acceptance
    ↑              │                │                       │
    └──────────────┴──failures, findings────────────────────┘

implement = gb-dont-know-where-this-is-going, test = gb-how-the-turntables,
review = gb-how-dare-you
```

The user should only ever see work that is already tested, reviewed and
fixed. So implement → test → review repeats on its own until review comes
back clean. Each pass is kept fast; the full, slow check runs once at the
end:

- **gb-how-the-turntables** (area mode) runs the tests for the area the change touches
  (the profile says which), the proof tests, and the quick checks. It
  fails because the code is wrong (not the test) → back to gb-dont-know-where-this-is-going
  with the failures.
- **gb-how-dare-you** (pass mode) reviews everything since Start while
  `Reviewed` is empty; after that (later passes and later rounds), only
  what changed since `Reviewed`, plus whether
  the previous findings were really fixed. Findings to fix → back to
  gb-dont-know-where-this-is-going with exactly those, then test and review again.
- **gb-how-dare-you** returns questions (Asks): ask the user before
  anything else, record the answers in Decisions, and turn each into a
  fix or a drop. Any fix → back to gb-dont-know-where-this-is-going.
- **gb-how-dare-you** clean (nothing left to fix, no open questions) → the
  final check.

Increment `Build pass` each time gb-dont-know-where-this-is-going starts a
pass. Log each pass in the state file with the files it changed (`build
pass 2: fixed missing access check on PUT /items/{id}; items/api.py,
tests/test_items.py`), and the files each test step added or changed, so
a resume can tell the work's changes from the user's. **Limit: 3 passes** per round (counted from the round's
start, or from the last restart if later, see "Scope added along the
way"). If a fourth would be needed for any
reason (tests still failing, review or the final check still finding
things, a gb-fire-drill ❌), stop and show the user what keeps coming back instead of
going around again: it usually means the approach is wrong, not the
details.

## The final check

Right after the build loop comes back clean, and before gb-fire-drill and
acceptance, check the whole work once, in full. Take a snapshot first and
put it in both briefs: review reviews exactly that, so files the test run
generates can't slip into it. Then start two subagents at the same time
(neither changes the working tree):

- **gb-how-the-turntables, final mode**: every suite, linter and type check the profile
  lists. It writes no tests.
- **gb-how-dare-you, final mode**: everything since Start, not just the last
  pass.

Both clean → checks, then acceptance. Anything to fix (a failing test
anywhere, a finding) → back to gb-dont-know-where-this-is-going as the next build pass; it
counts toward the limit. The loop runs as usual, then the final check runs
again, in full.

Never present acceptance without a clean final check. If it can't run
(a service is down and gb-im-going-to-die can't bring it up), stop and say so; skip it
only if the user explicitly says to, and then acceptance says plainly that
it was skipped.

## Subagents for test and review

Run **gb-how-the-turntables** and **gb-how-dare-you** each in a fresh subagent (Agent tool,
general-purpose), every time they run. Two reasons:

- **An independent review.** You wrote the code and know why every line is
  the way it is; a reviewer who knows that too will agree with it. The
  subagent sees the code, not your reasoning.
- **A context that lasts.** Test output and review findings are long; in a
  subagent they stay out of the main session, which only gets the report.

Brief the subagent with: the skill to load and follow (`gb-how-the-turntables` or
`gb-how-dare-you`) and its file (`~/.claude/skills/<name>/SKILL.md`, to
read if it can't load skills), the git root (and that every git command
runs from it), the state file's path, the profile (the path to
`.claude/gangbang.md`, or, if the user declined the file, the profile text
itself, pasted into the brief), the Start commit, the round and build
pass, the mode (gb-how-the-turntables: pin, area or final;
gb-how-dare-you: pass or final), for gb-how-dare-you a snapshot you take
right before starting it (the final check's shared one there), and where
to write its report
(`<slug>/test-r<round>p<pass>-<mode>.md` or
`<slug>/review-r<round>p<pass>-<mode>.md`). For a pass-mode review
whenever `Reviewed` is set, also give the previous review's report path, so it can check
those findings were fixed. When a subagent runs again in the same pass
and mode (after gb-im-going-to-die, or after the user answered its
questions), add `-a2`, `-a3`… to the report name, so an old report is
never mistaken for the new one. Tell it not to write the state file. Don't pass your notes on why the code is the way it is.

Take a snapshot and set `Running` (report paths and snapshot) in the
state file before starting a subagent, and clear it
once its report is logged. From each test report, update the state file's
Proof section; from each review report, set `Reviewed` to the snapshot it
names and log its fix list. If the session dies in between, `resume` sees
`Running` still set and picks up the report, or runs the step again.

**Don't touch the working tree while gb-how-the-turntables or gb-how-dare-you runs.** Wait for
the report before editing, building or running anything else in the repo.
Tests and review must see one fixed version of the code; results from a
tree that changed under them prove nothing about any version. If the user
asks for a change meanwhile, note it and make it after the report, as the
next build pass.

A subagent can't talk to the user. Tell it to return, instead of asking:
questions for the user, a service that is down, a design problem. You ask
the user, write the answer into the state file's Decisions, and pass it on.

gb-dont-know-where-this-is-going, and the steps that stop for the user, run in the main
session.

## Rules for every step

**When there's more than one way, ask.** In any step, when there are
several reasonable ways to handle something and you aren't sure which one
the user would pick (two data models, a minimal fix or a deeper one, which
of two conventions to follow), stop and ask. Give the options, the
trade-offs and your recommendation. Don't ask when the project already
answers it (its conventions, the design, an earlier decision); pick that
one and mention it. Write each answer into Decisions, so later steps and
rounds don't ask again.

**Escalation.** If any step hits a decision nobody made, or finds the
request is bigger than triage thought (a bug whose cause is a design
flaw, a small change that needs a new endpoint), stop, say so, and propose
switching the route to include gb-would-an-idiot-do-that. Triage will sometimes
underestimate; this is the safety net.

**Scope added along the way.** When the user adds to the work mid-way (fix
this other thing too), add it to the criteria as new numbered items marked
`(added pass N)` or `(added round N)`, and record it in Decisions. Restart
the pass limit: log `pass limit restarts after pass N` and why, and count
the 3 passes from there. Don't reset `Build pass` itself; report names
depend on it staying unique. The pass limit is there to catch an
approach that keeps failing, not a scope that grew. If the addition needs
decisions of its own (data, API, access), that's escalation, above.

## Acceptance

The user decides whether the work is done, and they decide from proof, not
from your say-so. Go through the acceptance criteria one by one:

- ✅ met, **with proof**: the test that demonstrates it (name and file),
  preferably, or a screenshot from gb-fire-drill, or a command and its
  output. "I implemented it" is not proof. For a bug, the reproducing test
  from gb-its-marijuana, now passing, is the first proof. For a measured
  criterion (faster, smaller), the command, the baseline at Start, the
  result now, and how many runs each.
- 🔎 met **by reading the code**: only where no test can prove it, or a
  project rule forbids the test that would (e.g. no assertions on message
  strings). Give the file:line and the reasoning; gb-how-dare-you must have
  checked it. The user decides whether that's enough.
- ❌ not met, and why
- ⚠️ changed during the work, and how

For each test you cite as proof, say in a few words what data it uses and
why that data can really occur: allowed by the business rules and reachable
through the actual code paths (no state the API could never produce, no
values validation would reject). A test passing on impossible data proves
nothing. Also say how it did on the code before the change (gb-how-the-turntables checks
this): **fails before** (it catches the problem), **passes before** (weak
proof, with why), **passes as required** (an unchanged-behaviour
criterion or a refactor's pinned test, which must pass before and after),
or **new API** (couldn't compile before).

Below that: the final check's test results (one line per suite, the full
suite) and its review verdict, or plainly that the user had it skipped;
how many build passes it took and what review caught along the way (one line each), review findings
dropped as not worth fixing (one line each, so the user can disagree), and
the diff stat since Start: `git diff --stat <Start> <snapshot>` with a
fresh snapshot, so new files count, leaving out `Excluded`.

Build all of this from the state file (Proof, Log, Decisions) and the
reports in `<slug>/`, not from memory: after a `resume`, that's all there
is.

Then **ask the user whether the work is done correctly**, and stop.

If it isn't, discuss what's not okay: understand the problem first, then
offer alternative approaches (two or three, with trade-offs and your
recommendation) rather than one fix. Once you agree on a fix, route back:

| What's wrong | Back to |
|---|---|
| behaviour missing or wrong | gb-dont-know-where-this-is-going (the build loop) |
| works but proof is missing, weak, or uses unrealistic data | gb-how-the-turntables (then review) |
| code works but needs cleaning | gb-how-dare-you, pass mode over `<Start>..S`, briefed with what acceptance rejected (then the build loop from review) |
| a screen is wrong | gb-dont-know-where-this-is-going, then gb-fire-drill again |
| the bug's cause was misread | gb-its-marijuana, then ✋ again |
| a criterion was wrong, or the scope changed | gb-would-an-idiot-do-that (or gb-diversity-day, if the kind changed), then ✋ again |

The user's choice wins over your suggestion.

**Going back always runs every step after it again**, in the route's
order: back to implement means the whole build loop again, until review is
clean, then the final check, checks and acceptance. A late fix should never
reach the user untested or unreviewed. On later rounds, implement, test and
pass reviews work only on what acceptance sent back; the final check still
covers all the work's changes since Start. Write what was rejected into
the log so the next round fixes exactly that. A new round increments
`Round` and sets `Build pass` to 0.

**Round limit.** If round 3's acceptance still fails, stop looping and say
so: repeated rejections usually mean the design (or the diagnosis) is
wrong, not the code. Suggest going back to it.

## Done

When the user says it's done:

1. Mark the state file `Next: done`.
2. If `.claude/gangbang.md` exists, update it through gb-what-i-do-here-every-day (update
   mode) with what this work taught about the project, and say in the
   summary that it changed: it shows up in the diff next to the work.
3. If gb-im-going-to-die started or created anything this run (the state file lists
   it), ask whether to stop or keep it.
4. Summarize: what was built or fixed, where, the proof, anything left
   open.

The user handles commits and pull requests. Don't commit, push, or open a
PR.
