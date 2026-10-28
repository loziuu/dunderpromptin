---
name: gb-what-i-do-here-every-day
description: Profile step of the /gangbang workflow — load, generate or update the project profile (.claude/gangbang.md) describing the stack, architecture, API, conventions, security, validation and tests. Use when /gangbang starts or finishes, or when the user types /gb-what-i-do-here-every-day.
argument-hint: '<optional: "update">'
---

# gb-what-i-do-here-every-day

The later steps design, build and test in the project's own way, so they
need to know that way first. The profile is where it's written down:
`.claude/gangbang.md` at the git root.

## Start of a run

- **The file exists:** read it, then spot-check it against the code (a few
  paths and commands it names still exist). If something is stale, tell the
  user and fix it as in "Update" below. If it lacks something the sections
  below ask for (it was written before they did), fill that in the same
  way. Use it.
- **The file doesn't exist:** ask the user whether to generate it. Say what
  it is in a sentence: a checked-in summary of the project the workflow
  works from, updated after each finished feature.
  - **Yes:** analyze the project and write the file.
  - **No:** analyze the project just the same, keep the result in context,
    and write nothing. Under /gangbang, record the "no" in the state
    file's Decisions, so a resumed session re-analyzes without asking.
    Ask again on a later run, not during this one.

## Analyzing the project

Read before concluding: `CLAUDE.md` files, READMEs, docs folders, package
and build files, CI config, migrations or schema, a couple of complete
features end to end (from the route or entry point down to storage), and
the tests. Where the project already documents something, the profile
**points to it** and summarizes in a line or two, rather than copying it:
the project's own docs stay the source of truth, and a copy only goes
stale. Write down what you find, not what's usual for the stack; if
something isn't there (no auth, no validation layer), say so.

Write these sections, each with file paths as evidence:

```markdown
# Project profile
Generated <date> from <commit>. Updated after each /gangbang feature.

## Stack
Languages, frameworks, versions that matter, database, key libraries.

## Running and testing
How to run it; every test suite with its command; services the tests need
(database, containers) and how to check and start them; linters,
type checks, formatters. How long the full run takes, and which tests
cover which area (module or package → test classes or directories, with
the command to run just those), so a change can be tested quickly
before the full run at the end.

## Architecture
The parts (backend, clients, workers...), how they talk, the module
layout, and where a new feature's code goes.

## Layers of abstraction
The layers a request passes through (e.g. HTTP → command → domain →
repository → SQL), what each may and may not do, and how dependencies
between modules are kept in check.

## API
Style (REST, RPC, GraphQL), URL and naming patterns, request and response
shapes, status codes, the error format, pagination, ids exposed, and
where the conventions are written down.

## Validation
Where input is validated and normalized, how failures are reported, which
layer owns which rules.

## Security
Authentication, authorization (how access is checked, what outsiders
see), secrets, sensitive data and where it must not go, CSRF/CORS, upload
handling — whatever applies.

## Data
Schema and migrations: where they live, naming, how to add one; ids,
timestamps, soft deletes, audit or event logs.

## Docs and design
Where design docs go and their style; where UI mockups go, if the project
keeps any; which docs must be updated with a change.

## Rules
Project rules from CLAUDE.md and the docs that a change must respect, and
things that look wrong but are deliberate.
```

Keep it short enough to read in a minute or two: a map of the project, not
a manual.

## Update

After a finished /gangbang feature, or with `$ARGUMENTS` = "update": add
what the feature taught you (a new convention, a new module, a decision
that applies beyond this feature), fix anything found stale, and move the
"Updated" line. Change only what changed. Show the user what you changed in
a few lines; they review and commit it with the rest.
