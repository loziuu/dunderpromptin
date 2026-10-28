---
name: gb-im-going-to-die
description: Environment step of the /gangbang workflow — work out what the project's tests need (services, config, dependencies), check it, and get it running, either by the user or by you using only existing resources. Use when /gangbang reaches its environment step or a test step reports something down, or when the user types /gb-im-going-to-die.
argument-hint: '<optional: "me" or "you", who sets it up>'
---

# gb-im-going-to-die

Tests can only prove anything if they actually run. This step makes sure
everything they need is up before the first test step, and decides who
does the setting up. Work from the project profile's "Running and testing"
section (gb-what-i-do-here-every-day).

## 1. Who sets it up

Settings for this project live in `~/.claude/work/<project>/env.md`
(`<project>` as in /gangbang). If $ARGUMENTS is `me` or `you`, use that.
Otherwise, if `env.md` records a choice, use it and say so in one line.
Otherwise ask the user:

- **The user sets it up**: you check and tell them what's missing.
- **You set it up**: you check and fix what you can, within the rules
  below.

Write the choice into `env.md`, so later runs don't ask again. The user can
change it any time by saying so or with `/gb-im-going-to-die me` / `/gb-im-going-to-die you`.

## 2. Work out what's needed

From the profile, list what the test suites need, each with a read-only
check:

- **Services**: databases, containers, caches, queues (`pg_isready`,
  `podman ps -a`, `docker ps -a`, `systemctl status`, a port check)
- **Config**: env files and variables the tests read (does `.env` exist,
  is `DATABASE_URL` set)
- **Dependencies**: toolchains and packages (`node_modules` present, the
  compiler version the project asks for)
- **Checks the user's CLAUDE.md asks for** before running tests

Run the checks. If everything is up, say so in one line and you're done.
`env.md` keeps the list and what worked last time, so the next run starts
there.

## 3. Get it running

**If the user sets it up:** list what's missing with the exact command for
each, and stop. When they say it's ready, check once more.

**If you set it up:** go through what's missing.

- **Use what exists, freely.** Start a stopped container that's already
  there, start an installed service, point at a database that already
  exists, run the project's own dev scripts for starting things.
- **Create nothing without permission.** A new container, volume or
  database, pulling an image, writing an env file (even copied from
  `.env.example`), installing packages: list exactly what you would create
  and why, and ask. Create it only after the user says yes, and only what
  they agreed to.
- **Never touch anything remote or production**, and never use credentials
  that aren't already configured for local use. If the only database
  configured looks like production, stop and say so.

Check again after. If something still won't come up, stop and tell the
user what you tried and what failed. Don't loop on it, and never fall back
to only compiling: untested code must not look tested.

## 4. Record

In `env.md`: what the tests need, the checks, and the commands that got it
running. In the /gangbang state file: anything you started or created this
run, so it can be offered for stopping at the end.
