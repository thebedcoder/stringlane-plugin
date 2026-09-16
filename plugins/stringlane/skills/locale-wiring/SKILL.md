---
name: locale-wiring
description: What adding or removing a locale means OUTSIDE the locale files — the supportedLocales list, Info.plist, the Xcode project, locale_config.xml, supportedLngs, and the language picker. Use when adding a language to an app, after add_locale, when a translated locale does not appear at runtime, or when removing a locale.
---

# Adding a locale is two jobs, and StringLane does one of them

`add_locale` creates the locale file, in the right place, in the right format,
with the right plural categories. That is the whole of what it does and the
whole of what it claims. **It does not make the app offer the language.**

Between a correct locale file and a user who can read it there is a second set
of edits, in files StringLane does not own and will never write: the app's own
source, its platform manifests, its picker. Every one of them is optional, so
none of them can be assumed — and skipping one produces the same symptom as
having no translation at all, with a fully translated file sitting on disk.

Never report a locale as added until this side is done or confirmed unnecessary.

## The one question that decides whether there is work

Every platform here has some form of **allowlist of languages the app admits**,
and on every platform that allowlist is *optional*. That single fork decides
everything:

- **No allowlist** — the platform discovers the locale from the file itself.
  There is nothing to edit, and hunting for something to edit wastes the user's
  time and risks inventing a list that then has to be maintained forever.
- **An allowlist exists** — the new locale is rejected until it is added there.
  The file is correct, the build is green, and the language silently does not
  exist.

So the first move is never "edit `supportedLocales`". It is: *does this project
have an allowlist, and of which shape?*

## Read the project's recorded wiring before scanning for it

If `.stringlane/locale-wiring.md` exists, read it first. `/stringlane:init`
writes it after a scan, and it records both what exists and what does **not** —
the "Not applicable" entries are the valuable half, because they stop every
later session re-hunting for a picker the project has never had.

It is a note, not a lock. Each entry carries an **anchor**: the line as it read
when it was recorded. Before you act on a pinned `file:line`:

1. Read that file and check the anchor still matches.
2. **Matches** → the pin is good, use it.
3. **Moved, or gone** → the pin is stale. Re-detect that one entry, do the work
   against what you actually found, and then offer to update the recorded line.
   Say the pin was stale; do not silently fix it, because a note that drifts
   without anyone noticing is worse than no note.

No such file, or it does not cover the platform you are on → detect it now, and
offer to record what you found. Ask before writing it: it is a file in the
user's repository and it will be committed.

## Flutter / ARB

`flutter gen-l10n` globs `arb-dir`, so **`l10n.yaml` needs no change** — there is
no list of locales in it, and people lose time looking for one. Regenerate after
adding the file, and `AppLocalizations.supportedLocales` will include the new
locale on its own.

Whether the *app* does is the fork, and it is one line:

- `supportedLocales: AppLocalizations.supportedLocales` — auto-tracks. Nothing
  to do.
- `supportedLocales: const [Locale('en'), Locale('de')]` — a literal. The
  generated class knows about the new locale and the app does not. **Add it
  here or nothing else matters.**

Then the two native sides, which `gen-l10n` does not touch:

- **iOS** — `ios/Runner/Info.plist`, `CFBundleLocalizations`. Flutter's own
  generated `app_localizations.dart` carries a doc comment saying this list
  should be kept consistent with `AppLocalizations.supportedLocales`, which is
  a comment nobody reads in a generated file nobody opens. A Flutter app's
  strings are not in `.lproj` folders, so iOS has nothing to infer the list
  from — without this key the system does not consider the app localized into
  that language, and the App Store listing will not show it.
- **Android** — per-app language, below. Identical to a native Android app.

And the picker, if the app has one: a `LocaleProvider`/`ChangeNotifier` with a
persisted `Locale`, plus a hardcoded code → display-name map. A locale missing
from that map is untranslatable-into by the only means the user has.

## iOS — `.strings` / `.stringsdict`

A locale is a new `xx.lproj/` directory holding `Localizable.strings`, and
`Localizable.stringsdict` beside it if any key has plurals. The two are one
unit: editing one without the other leaves the plural and the singular
disagreeing.

The gate is **Xcode's project file**. Localizations are recorded in
`project.pbxproj` — in `knownRegions`, and as a child reference inside each
`PBXVariantGroup`. An `.lproj` directory that exists on disk with no entry there
is simply not in the build: the file is present, correct, and never read.

**Do not hand-edit `project.pbxproj`.** It is a generated, reference-counted
format where a wrong UUID corrupts the project, and it is the file most likely
to conflict in a team's history. Tell the user to add the language in Xcode —
Project → Info → Localizations → **+** — which writes both halves and creates
the `.lproj` folders it expects.

`CFBundleLocalizations` in `Info.plist` is a separate mechanism: for an
`.lproj`-shaped app the bundle derives its localization list from the folders,
and the key *overrides* that. Add the locale there only if the project already
sets the key — if it does, the automatic list is not being used and an omission
here is the bug.

## iOS — `.xcstrings`

One catalog for every locale, so `add_locale` adds entries **inside the existing
file**. There is no new file and no new `.lproj` — creating either is wrong.

Shipping is still gated on the project's Localizations list. Check Project →
Info → Localizations in Xcode: a catalog entry for a language the project does
not list gets translated and does not ship. That is the check to hand the user,
not something to fix from here.

## Android XML

The basic case needs nothing. `res/values-de/strings.xml` — the directory
qualifier *is* the locale, and the resource system resolves it with no list
anywhere. Android is the one platform where system-language following works the
moment the file exists.

Three things can still swallow it:

- **`resourceConfigurations`** (older AGP) or **`localeFilters`** (newer), in
  `defaultConfig`. An allowlist of what gets *packaged*. If it is set and the
  new locale is not in it, the strings are stripped from the APK — correct file,
  green build, missing language. The nastiest failure here, because nothing
  reports it.
- **Per-app language (Android 13+)** — needed for the app to appear in
  Settings → Apps → *App* → Language. Two mutually exclusive routes, and which
  one this project uses must be established before touching either:
  - *Hand-written*: a `res/xml/…` locale config referenced by
    `android:localeConfig` on `<application>`. Add a
    `<locale android:name="de"/>` entry.
  - *AGP-generated*: `androidResources { generateLocaleConfig = true }` (AGP 8.1+,
    `compileSdk` 33+) with a default locale in `res/resources.properties` via
    `unqualifiedResLocale`. The list is derived from the resource folders, so
    **there is nothing to edit** — and **a manually created LocaleConfig file
    must be removed or the build fails.** Adding one "to be safe" breaks the
    build.
- **The in-app picker**, if there is one:
  `AppCompatDelegate.setApplicationLocales(LocaleListCompat.forLanguageTags(...))`
  over a hardcoded list of tags.

## i18next JSON

`supportedLngs` defaults to `false`, meaning every language is accepted. So:

- **Not set** — nothing to do. The locale loads.
- **Set** — a language not in the array is *rejected* and falls back to
  `fallbackLng`. The JSON file is on disk and i18next ignores it, silently. Add
  the code. (`nonExplicitSupportedLngs: true` relaxes this to the language part
  only, so `de-AT` passes on a list containing `de`.)

Then how resources actually reach the runtime, which is one of three shapes and
the reason recording it is worth more than re-deriving it:

- A **backend** with `loadPath: '/locales/{{lng}}/{{ns}}.json'` — path-driven,
  nothing to edit. Confirm the file is served, not just present: a locale
  outside the bundler's copied assets 404s at runtime.
- A **static `resources` object** — needs a new `import` and a new entry. This
  is the one that gets forgotten.
- A **glob/`resourcesToBackend`** — nothing to edit.

Also per-locale and separate from i18next: `preload` if it is set server-side,
and any date library's own locale data — `date-fns`, `dayjs` and `luxon` each
need their locale imported and registered. `Intl` is built in and needs nothing.

Plus the language switcher's list, which is almost always hardcoded.

## Removing a locale is the same list, reversed — and it fails louder

Delete the locale file and every entry above becomes a dangling reference. An
allowlist naming a language whose resources are gone is worse than one missing a
language: depending on the platform it yields blank strings, a fallback the user
did not ask for, a picker entry that selects nothing, or a build failure. Walk
the same list and remove, do not just delete the file.

## What this skill does not do

It writes nothing. Every edit above is in a file StringLane does not own — app
source, platform manifests, build config — outside the write guard and outside
`classifyPath`. Propose the changes, make them the way you would make any other
source edit, and let the user's normal review apply.

And do not do this work by hand-writing the locale file itself. That is the
opposite trade: the wiring is source code and belongs in source control review;
the locale file is a checked format and belongs to `add_locale`, which gets the
path, the escaping and the plural categories right where a hand-written file
gets them wrong silently.
