---
name: gb-would-an-idiot-do-that
description: Design step of the /gangbang workflow — discuss a feature, write its design doc with acceptance criteria, sketch its screens if it has any. Use when /gangbang reaches its design step, or when the user types /gb-would-an-idiot-do-that.
argument-hint: <feature, or what to change in its design>
---

# gb-would-an-idiot-do-that

Turn a feature idea into a design the user can approve and the later steps
can build and prove against. Input: $ARGUMENTS, or the feature from the
/gangbang state file. Work from the project profile (gb-what-i-do-here-every-day); if it
isn't loaded, load it first.

## 1. Understand before writing

Read what exists: the docs for this area, the code the feature touches, the
conventions the profile points to. Then talk with the user: restate the
feature in a few lines, give your opinion on the approach and its
trade-offs, and ask the questions whose answers change the design. Don't
ask what the code or the docs already answer. Keep this a conversation; a
design built on a guess costs a whole loop later.

## 2. Write the design doc

Put it where the profile says design docs go, in the project's style; add
to the area's existing doc if there is one. With no convention, use
`docs/<area>.md`. Write down:

- **Decisions**, each with why, and alternatives you rejected
- **Data**: schema changes and the migration
- **API**: endpoints or interfaces with their shapes, following the
  project's API conventions. If something must break a convention, say so
  here explicitly, and plan to update the conventions doc.
- **Security and validation**: who may do what, what outsiders see, what
  input is checked where, following the project's way
- **UI**: what changes, if the project has one
- **Acceptance criteria**: start from triage's draft in the state file
  and refine it. A numbered list, each one observable and
  provable by a test ("an owner can archive an item; archived items leave
  the default list"), including the edge cases you discussed: access,
  empty states, invalid input, concurrency where it matters. Acceptance
  checks exactly this list, with a test as proof for each, so write each
  criterion so that a test can demonstrate it.

## 3. Sketch the screens

If the UI changes and the project keeps mockups (the profile says where),
draw them there, in the same style, and open them for the user. Designs
live in the repo. With no mockups in the project, describe the screens in
the design doc instead of inventing a format.

## 4. Hand off

Give the user the doc path, the decisions in brief and the acceptance
criteria, and stop for approval. Under /gangbang, record the design doc
path in the state file and point its criteria section at the doc.
