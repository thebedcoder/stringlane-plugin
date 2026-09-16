---
name: stringlane-init
description: "Set up StringLane for this project and reach a first validated translation"
---

Set this project up for StringLane, and stop at every point where a human should
decide. **Propose; never auto-apply.** Every write below is a separate yes.

You have no logic to write here. Each step is an existing command or MCP tool —
if a step seems to need something that does not exist, say so and stop rather
than improvising.

## 1. Find out what is there

Run `inspect_project`, or `stringlane inspect .` in a terminal.

If it reports no project: **stop.** Do not guess, do not create files. Say which
of these was looked for and not found, so the user knows what to fix:

- **Flutter/ARB** — a `pubspec.yaml`, and `.arb` files (usually `lib/l10n/`)
- **iOS** — `.strings` / `.stringsdict`, or a `.xcstrings` catalog
- **Android** — `res/values*/strings.xml`
- **i18next** — a `locales/` or `public/locales/` tree of `.json`

Then offer to run `stringlane init` in the right directory, or to point
StringLane at a path with `stringlane.yaml`. Show no stack trace.

## 2. Show the config before writing it

If there is no `stringlane.yaml`, show exactly what `stringlane init` would
write, and ask. It records the path and format so every later command agrees
about them.

## 3. Report the state plainly

From the same inspect result: the locales, the completion percentage for each,
and how many keys have no description. Do not editorialise — a 40%-complete
locale is a fact, not a problem, and the user may know why.

## 4. Descriptions come before translations

If `keysWithoutDescription` is high, say why it matters before offering to fix
it: a translator — human or model — given `"Cancel"` with no context cannot know
whether it is a button or a verb, and the two are different words in most
languages. Offer `describe_keys`, and ask first.

## 5. Context, proposed from what the repo already says

Read the README and the app's own metadata for the product name, tone, and any
terms that must not be translated (brand names, feature names). **Propose them;
do not write them.** The user confirms, then it goes through
`set_project_context` — or `stringlane context --from` in a terminal.

## 6. Find out what a NEW locale would cost this project

`add_locale` creates a locale file. It does not make the app **offer** the
language, and that second half lives in files StringLane does not own. Find them
now, once, while you are already reading this codebase — the alternative is
re-deriving them under time pressure on the day someone adds a language.

The `locale-wiring` skill describes what to look for per platform. What matters
here is the fork it opens with: every platform's list of admitted languages is
*optional*, so record **whether this project has one**, not what one would look
like in general.

Report what you found as file and line, with the line's current text as an
anchor. Record the absences too — "no picker", "no `supportedLngs`" — because an
absence stops the next session searching, and a blank entry does not.

Then offer to write it to `.stringlane/locale-wiring.md`, and **ask**. It is a
file in their repository and it will be committed. If they say no, say it
plainly and move on; the finding is still in this conversation.

## 7. Offer one concrete piece of work

Not "you could translate things". Name the locale with the most missing strings
and offer to start there, then stop and let them answer.
