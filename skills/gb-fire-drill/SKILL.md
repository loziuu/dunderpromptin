---
name: gb-fire-drill
description: Simulated manual testing step of the /gangbang workflow — with the user's permission, launch the app and check the criteria only a person could check (how a screen looks and behaves), capturing screenshots as proof. Use when /gangbang has manual criteria left before acceptance, or when the user types /gb-fire-drill.
argument-hint: '<criteria to check, default: the manual ones in the state file>'
---

# gb-fire-drill

Some criteria can't be proved by an automated test: a screen shows the
right thing, a flow feels right on the page. Instead of leaving them as
"check it yourself", run the app and check them the way a person would,
and bring back screenshots as proof.

## 1. Ask first

Launching the app starts servers, may seed data, and opens a browser or a
simulator, so ask the user before doing any of it. Say what you would run
(the commands from the profile), against which database, and which
criteria you'd check. If they say no, the criteria stay "manual" with exact
steps for them, and acceptance shows them that way.

## 2. Run and check

- Launch the app the way the profile describes, using the built-in `run`
  skill, or the browser skills to drive the UI.
- Use a development or test database, never production. If only
  production is configured, stop and say so.
- Set up the data each criterion needs through the app or the API, as a
  user would, so it is data that can really occur.
- For each criterion: do what a user would do, then take a screenshot of
  the result. Note what you did and what you saw.

## 3. Clean up and report

Stop whatever you started. Save screenshots under
`~/.claude/work/<project>/<slug>/` (same names as the /gangbang state
file), and list per criterion: the steps, the screenshot path, and ✅ or ❌
with what was wrong. Under /gangbang, update each criterion's line in the
state file's Proof section with that, plus a snapshot of the code you
checked (recipe in `~/.claude/skills/gangbang/SKILL.md`, "Snapshots"), so
a later change to that code sends it back to you. A ❌ goes back into the build loop
like a failing test.
