# StringLane

Localization for coding agents: an MCP server for your locale files, plus a write guard that asks before an agent hand-edits them.

Use the `stringlane` MCP server for anything about this project's
translations — what is missing, what a key says in a locale, adding a key or
a language. Do not read or hand-edit locale files: four of the five supported
formats escape, prefix or nest their values, so the bytes on disk are not the
string, and a hand edit skips the placeholder, plural and length checks.

@./skills/locale-wiring/SKILL.md
@./skills/localization-formats/SKILL.md
@./skills/new-strings/SKILL.md
