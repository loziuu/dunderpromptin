---
name: gb-diversity-day
description: Triage step of the /gangbang workflow — scan the code a request touches, decide whether it's a feature, bug, refactor or small change, and propose the route and acceptance criteria. Use when /gangbang starts on a new request, or when the user types /gb-diversity-day.
argument-hint: <the request>
---

# gb-diversity-day

Decide what the request actually needs before any work starts. A bug
doesn't need a design doc and a new feature can't skip one; telling them
apart takes looking at the code, not just at how the request is worded.
Input: $ARGUMENTS, or the feature from the /gangbang state file. Work from
the project profile (gb-what-i-do-here-every-day).

## 1. Scan for this request

The profile describes the whole project; this scan is about the request.
Find:

- the code it touches, in every layer (backend, frontend, mobile, data)
- the existing tests around it
- related things: similar features, past decisions in the docs, TODOs,
  earlier changes to the same code (`git log` on the files)

## 2. Classify it

From what the code shows, not from the request's wording. A reported "bug"
may be behaviour nobody built (a feature); a "small tweak" may cut across
layers and need decisions (a feature).

- **Feature**: new behaviour, or a change that needs decisions (data, API,
  access, UI).
- **Bug**: existing behaviour differs from what was intended (by the docs,
  the design, the obvious meaning of the code).
- **Refactor**: structure changes, behaviour must not.
- **Small change**: behaviour changes, but no decision is open (a label, a
  limit everyone agreed on, a missing field the design already lists).

## 3. Propose

Show the user, briefly:

- **Kind**, and the evidence from the scan that makes it that kind
- **What it touches**: layers and main files
- **Route**: the one for its kind, from the Routes table in the gangbang
  skill (`~/.claude/skills/gangbang/SKILL.md`)
- **Acceptance criteria**: a numbered list, each provable by a test, or,
  for a goal like "faster", by a measurement against a baseline (say the
  command and the target). For a
  feature this is a first draft that gb-would-an-idiot-do-that refines; for a bug it starts
  with "<the bugged behaviour> no longer happens" and "the rest of <area>
  behaves as before"; for a refactor, "behaviour unchanged" plus which
  behaviour exactly.

Then stop. The user approves, changes the kind or route, or edits the
criteria. Their choice wins.

Under /gangbang, write the approved kind, route and criteria into the
state file. Without a design doc, the state file is where the criteria
live.
