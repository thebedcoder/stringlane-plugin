---
description: Translate one locale: plan, validate, apply
argument-hint: <locale>
---

Translate `$ARGUMENTS` for this project.

Three steps, in order:

1. **`prepare_translation_plan`** for that locale. If it comes back empty, the
   locale is complete — say so and stop. **Do not write the file yourself.**
   An empty plan means there is no work, not that the tool failed; treating it
   as failure and hand-writing the file is exactly how 472 strings were once
   written into a `uk.arb` that skipped every check.

2. **`validate_translations`** on what you produced. Fix what it reports and
   validate again. A clean validate is not permission to write — it is the
   check that the write would pass.

3. **`apply_translations`**, which takes no authorization argument — your host
   asks the user before it runs, and the write lands as one atomic commit under
   a per-project lock. Say what you are about to write before you call it, and
   point them at `git diff` afterwards; that is where they review it.

If the locale does not exist yet, use `add_locale` first — the path, the file
format and the plural categories a locale needs all differ per project, and a
hand-made file gets them wrong silently.

If your code references keys the project does not have yet, use `add_key` —
all of them in ONE call, each with a description of what the string is for. One
call is one atomic write, and the description goes in at the only moment you
still know the answer. It writes the base locale only, so the keys then show up
in the next plan as missing everywhere else.
