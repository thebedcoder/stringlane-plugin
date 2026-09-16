---
name: localization-formats
description: How each localization format actually stores strings, plurals and descriptions — the gotchas that make a hand-written file wrong. Use when editing or reasoning about ARB, iOS .strings/.stringsdict/.xcstrings, Android XML, or i18next JSON.
---

# What each format actually does

Four of the five escape or nest their values, so **the bytes on disk are not the
string**. That is the single sentence behind most of what follows.

## The order that matters: describe → plan → validate → apply

Descriptions first, and not as a nicety. A translator — human or model — handed
`"Cancel"` with no context cannot tell a button from a verb, and those are
different words in German, French and most other languages. `describe_keys`
before `prepare_translation_plan` changes the output, not just the paperwork.

Then `validate_translations`, then `apply_translations`. Validate is not
permission to write: apply re-runs every check itself and also verifies nothing
changed since the plan was made.

## ARB (Flutter)

- Metadata lives beside the string, in an `@key` sibling: `"@greeting": {"description": ...}`.
- **Key order is significant** to `flutter gen-l10n` output. Do not re-sort a file.
- Plurals are ICU inside the value: `{count, plural, =0{...} one{...} other{...}}`.
  A locale's required categories come from CLDR, not from English — Polish needs
  `few` and `many`; Japanese needs only `other`. Getting this wrong produces a
  file that compiles and renders the wrong string.
- `@@locale` is the locale, and it must match the filename's suffix.

## iOS

Three formats, and a project may use two at once.

- **`.strings`** — `"key" = "value";` with a trailing semicolon. Escaping is
  C-style: `\"` and `\n`. A missing semicolon breaks the file from that line on.
- **`.stringsdict`** — a plist SIDECAR holding the plurals for keys whose
  `.strings` entry is a placeholder. It is a *separate file with the same
  basename*, and editing one without the other leaves them inconsistent. Tools
  that list "the localization files" routinely omit it.
- **`.xcstrings`** — one JSON catalog for EVERY locale. Adding a locale means
  adding entries inside the existing file, not creating a new one.

## Android XML

- `res/values/strings.xml` is the default; `res/values-de/` and friends are the
  translations. The directory qualifier is the locale.
- **Apostrophes and quotes must be escaped** (`\'`), or the resource compiler
  fails. `&` becomes `&amp;`.
- Plurals are `<plurals>` with `<item quantity="one">` children, and the
  quantities must be in **CLDR order**.
- `<string name="x">` may contain markup, which must survive translation intact.

## i18next JSON

- Nested objects are namespaces; the key is the dotted path.
- Interpolation is `{{name}}` — double braces, unlike every other format here.
- Plurals are separate keys with suffixes: `key_one`, `key_other`.
- **Descriptions are NOT in the file.** The format has nowhere to put them, so
  StringLane keeps them in `.stringlane/metadata.yaml`. Adding a `"_comment"` key
  to the JSON does not work — it becomes a translatable string.

## Why hand-editing goes wrong

Placeholders must survive exactly: `{name}` dropped from a translation is a
crash, not a typo. Plural categories differ per locale. Escaping differs per
format. Length changes — German runs 30% longer than English and overflows
layouts. Every one of these is checked by `validate_translations` and by
`stringlane apply`, and skipped entirely by a `Write`.
