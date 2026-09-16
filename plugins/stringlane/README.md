<p align="center">
  <a href="https://stringlane.app">
    <img src="https://stringlane.app/readme/plugin.png" alt="StringLane for coding agents — localization on autopilot, locally. Flutter ARB, iOS .strings / .xcstrings, Android XML, i18next JSON." width="960">
  </a>
</p>

<p align="center">
  <a href="https://stringlane.app">Website</a> ·
  <a href="https://stringlane.app/docs/cli/agent-setup">Agent setup</a> ·
  <a href="https://stringlane.app/docs/reference/mcp-tools">MCP tools</a> ·
  <a href="https://stringlane.app/product/desktop">Desktop app</a> ·
  <a href="https://stringlane.app/release-notes">Release notes</a>
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@stringlane/cli"><img alt="stringlane CLI on npm" src="https://img.shields.io/npm/v/%40stringlane%2Fcli?label=stringlane%20cli&color=00C896&labelColor=0C0E13"></a>
  <img alt="MIT licence" src="https://img.shields.io/badge/licence-MIT-00C896?labelColor=0C0E13">
</p>

<p align="center">
  <sub>Free of charge · no account · no import step, no export step — your files stay where they are</sub>
</p>

---

Localization for coding agents: an MCP server over your locale files, plus a
write guard that asks before an agent hand-edits them.

Flutter ARB, iOS `.strings` and `.xcstrings`, Android `strings.xml`, i18next
JSON. Your files stay where they are — there is no import step, no export step,
no account, and no copy of your strings anywhere but your repository.

## What your agent gets

**Ten MCP tools.** `inspect_project` for the state of a project,
`read_translations` and `describe_keys` for what a key says and means,
`read_key_usage` for where the code calls it, then the three that do the work —
`prepare_translation_plan`, `validate_translations`, `apply_translations` — plus
`add_key`, `add_locale` and `set_project_context`.

Four of the five formats escape, prefix or nest their values, so the bytes on
disk are not the string. Every read goes through the parser and every write
through the placeholder, ICU-plural and length checks, which is the difference
between a translation that compiles and one that ships a literal `%1$s` to a
user.

**A write guard**, and it is the reason this is a plugin rather than a bare MCP
entry. An MCP server sees only its own tool calls. It has no idea an agent just
wrote a locale file with `Write`, or with `Bash`, and this project has twice
watched that happen ([#132][issue]). The guard is a hook, so it sees both: it
asks a human before a direct edit, and reports one that already happened.

It **asks**; it never refuses on its own. Anything it cannot decide is allowed,
because a guard that blocks when it is confused gets turned off — and then it
guards nothing.

**Four commands** — `/stringlane:setup`, `/stringlane:init`,
`/stringlane:translate <locale>` and `/stringlane:review`. On hosts that load
skills rather than commands they arrive under the same names with a
`stringlane-` prefix.

**Three skills**, for the parts no tool call can finish on its own.

*The other half of adding a locale.* `add_locale` creates the locale file and
stops — correctly, since that is the half StringLane owns. The rest is in your
app: a `supportedLocales` literal, an `Info.plist`, an Xcode project, a
`locale_config.xml`, a `supportedLngs` array, a language picker. Every one of
them is optional, so none can be assumed, and missing one produces a fully
translated locale the app never offers. The `locale-wiring` skill carries that
per platform; `/stringlane:init` scans for yours and, if you let it, records
what it found — the absences included — in `.stringlane/locale-wiring.md`.

*The other half of adding a string.* `add_key` writes the base locale and stops,
which is right and is not finished: every other language is behind the moment it
lands, and a successful tool call says nothing about that. The `new-strings`
skill carries the stop rule, including at the one moment no hook can see — while
the string is still a literal in a component and no locale file has been
touched.

*What each format can and cannot say.* `localization-formats` is the reference
for plurals, metadata, placeholders and ICU across the five.

## Install

Two ways in, and which one you get is the host's decision rather than ours.
Three hosts load this repository directly. The rest have no plugin format this
directory fits, so they install the CLI and take a config entry printed for
them — same server, same version, one extra paste.

| Host | Install | MCP server | Write guard |
|---|---|---|---|
| Claude Code | plugin | ✅ | ✅ installed with it |
| Codex CLI | plugin | ✅ | ✅ ships with it |
| Gemini CLI | extension | ✅ | paste † |
| Cursor | CLI + paste | ✅ | paste † |
| VS Code Copilot | CLI + `code --add-mcp` | ✅ | paste, but inert ‡ |
| GitHub Copilot CLI | CLI + paste | ✅ | paste, but inert ‡ |
| Windsurf | CLI + paste | ✅ | none — see below |
| Zed | CLI + paste | ✅ | permission block |
| OpenCode | CLI + paste | ✅ | permission block |

**† "paste" is a real guard, not a placeholder.** `stringlane setup hooks --host
cursor` prints the hook document for that host, you paste it into the file it
names, and `stringlane hook --host cursor` answers Cursor's protocol from then
on — refusing an edit to a locale file and telling the agent which tool to use
instead. The dialects are implemented and tested; what they are missing is a
*recording*, which is a captured session proving what the host actually sends.

Until a host has one, `--apply` prints instead of writing, for that command and
for `setup mcp`. StringLane will not merge into a file format nobody has run,
because a config a host silently ignores is worse than no config. It will gladly
show you the bytes: a paste writes nothing you did not read first, and
`--remove` still works, so the guard is never something you cannot take back off.

**‡ VS Code is the exception, and the command says so.** Its edit-tool names
appear in documentation rather than as hook values, so StringLane has no
recorded list of them and the hook allows every write. Pasting it is harmless
and guards nothing yet — a guessed tool name would be worse, because it would
look like a guard while never firing. `setup hooks --host vscode` prints that
warning under the document.

Today `--apply` writes for Claude Code. Everywhere else the answer is a paste,
and a print that could not become a write exits non-zero — the entry is still
there to copy, and the exit code is the command telling you it changed nothing.

Every host also gets `stringlane setup rules`, which writes three localization
rules into your `AGENTS.md` between two markers it can take back out.

### Claude Code

```
/plugin marketplace add thebedcoder/stringlane-plugin
/plugin install stringlane@stringlane
```

That is the whole install: the MCP server, the four commands, the skills and the
write guard, which registers on `PreToolUse`, `PostToolUse`, `PostToolUseFailure`
and `SessionStart`.

### Codex CLI

```sh
codex plugin marketplace add thebedcoder/stringlane-plugin
codex plugin add stringlane@stringlane
```

Codex reads its own manifest (`.codex-plugin/plugin.json`) and its own
marketplace index (`.agents/plugins/marketplace.json`); both are in this
repository, generated from the same source as the Claude Code ones. The four
slash commands arrive as skills — `/stringlane:setup` is `stringlane-setup`
here — because Codex does not load a `commands/` directory.

`hooks/hooks.json` is the file Codex's own discovery is documented to find, and
it is the same document Claude Code installs. Which paths that discovery
actually scans is described rather than enumerated in the spec, so if the plugin
install does not pick the guard up, `stringlane setup hooks --host codex` prints
the same document and the file to put it in.

### Gemini CLI

An extension, and it installs from a path rather than the repository URL:

```sh
git clone https://github.com/thebedcoder/stringlane-plugin
gemini extensions install ./stringlane-plugin/plugins/stringlane
```

`gemini extensions install <repository-url>` expects `gemini-extension.json` at
the repository ROOT. This repository keeps the plugin in a subdirectory so that
one copy serves every host, so the documented local-path form is the one to
use — same bytes, same version.

The extension brings the MCP server and `GEMINI.md`. It declares **no hooks** —
an extension manifest is not where Gemini reads them from — so the guard is one
more command:

```sh
stringlane setup hooks --host gemini-cli
```

That prints the document and the `settings.json` to paste it into. Gemini's hook
protocol has no `ask`, so StringLane's refusal is a `deny` that names the tool to
use instead and carries
`stringlane setup hooks --host gemini-cli --remove` in the same message — a
refusal you cannot lift is one people turn off for good.

### Cursor, Windsurf, Zed, OpenCode, VS Code Copilot

No plugin, one command. Install the CLI, then ask it for the entry:

```sh
npm install -g @stringlane/cli
stringlane setup mcp --host cursor
```

`--host` takes `cursor`, `windsurf`, `zed`, `opencode` or `vscode`, and
`--scope project` prints the committed, per-repository entry where the host has
one. Each report names the exact file, gives the entry in that host's own shape
— these five hosts want the entry under four different container keys, and one
under the wrong key is not an error on any of them: the host starts, lists no
StringLane tools, and says nothing about why — and states what the host needs
before it will read the file at all.

Two of them have a faster path, printed with the entry:

```sh
code --add-mcp '{"name":"stringlane","command":"stringlane","args":["mcp","."]}'
```

writes the VS Code user profile, and Cursor prints a
`cursor://anysphere.cursor-deeplink/mcp/install` link. The link is built from
the documented shape and has not been run; the paste above it is the part that
is known to be right.

**Cursor gets the same guard as Gemini**, from its own hook document:

```sh
stringlane setup hooks --host cursor
```

Paste that into `~/.cursor/hooks.json` or `<project>/.cursor/hooks.json`. Cursor
has no `ask` either, so the same `deny`-with-a-way-out applies.

**VS Code prints too**, into `~/.copilot/hooks/stringlane.json` or
`<project>/.github/hooks/stringlane.json` — see the ‡ note above for why it does
not guard anything yet.

**Zed and OpenCode get a guard anyway**, from their own permission tables rather
than from a hook:

```sh
stringlane setup guard --host zed
```

That prints an `always_confirm` block (Zed) or a `permission` block (OpenCode)
covering this project's locale files, in that host's own pattern language — Zed's
patterns are Rust regexes, OpenCode's are globs, and a locale path emitted
verbatim is not a path but a pattern that looks like one. It is a **snapshot**:
a locale file added later is uncovered until you run it again. Paste it in
yourself — both files are JSONC, and StringLane will not rewrite a file whose
comments its parser would discard.

**Windsurf gets no guard, and that is a decision rather than a gap.** Its
pre-write hook is exit-code only, with no message back to the model. A silent
block teaches the agent nothing, so it routes around it through the shell —
which is worse than not blocking, because you believe you are guarded.

### GitHub Copilot CLI

No plugin install, and that is not an oversight either. Copilot CLI's plugin
format is its own: a root `manifest.json`, TypeScript `agents/`, `commands/` and
`skills/`, and a marketplace that is a JSON index served over HTTP with a
`downloadUrl` per entry. It shares nothing with the directory here, so there is
no command that would work. Install the CLI and take the MCP entry by hand:

```sh
npm install -g @stringlane/cli
stringlane setup mcp --host vscode --scope project
```

Which file Copilot CLI reads a `servers` entry from is not recorded here, so
nothing writes it for you. Its hooks directory is `~/.copilot/hooks/`, and
`stringlane setup hooks --host vscode` prints the `stringlane.json` that goes in
it — with the ‡ warning above attached, because that dialect has no recorded
tool names and so allows every write.

## Start here

```
/stringlane:setup
```

The plugin brings the MCP server and the write guard with it, but both of them
call a `stringlane` binary that has to exist on your machine. `/stringlane:setup`
finds out whether it does — and whether the copy it found is new enough to
answer `stringlane hook`, which is the failure that looks exactly like working.

It reads and reports. The one thing it can write is a package install, and it
asks first.

Then `/stringlane:init` for the project itself.

On a host that took the CLI path there is no slash command to run: `stringlane
inspect` is the same first question, and `stringlane init` the same next step.

## Turning the guard off

`/plugin` → disable, or take just the hooks back out:

```sh
stringlane setup hooks --host claude-code --remove
```

`--remove` works on **every** host with a hook, recorded or not — deliberately.
`--apply` is StringLane proposing a change to a file it has never seen your host
write; `--remove` is you revoking something you pasted in yourself. Refusing to
install is cautious. Refusing to uninstall is a trap, and on the hosts whose
refusal is a `deny` it is the message's own way out.

Prevention covers `Write`, `Edit`, `MultiEdit` and `NotebookEdit`. A `Bash` write
cannot be prevented — nothing in the tool call says which file it will touch — so
it is covered after the fact instead, by comparing your locale files against a
baseline taken when the session started. No completeness is claimed for
prevention, and none should be read into it.

[issue]: https://github.com/thebedcoder/StringLane/issues/132

## Windows

The MCP server and the hooks work. The **resolver shim does not** — `bin/stringlane`
is a POSIX shell script, and Windows will not run it.

In practice that costs you one convenience, not the plugin. The shim exists only
to find StringLane when it is not already on `PATH`, so on Windows you install it
yourself once and everything else behaves identically:

```sh
npm install -g @stringlane/cli
```

Once `stringlane` is on `PATH` — a global install, or your project's own
`node_modules/.bin` — the hooks and the MCP server call it directly and never
reach the shim.

What you lose without that step is the zero-install `npx` fallback: the hook
command will not resolve, and the guard stays silent rather than blocking
anything. That is the designed failure, not a broken one — but it does mean the
guard is not protecting you, so the install is worth doing.

A `bin/stringlane.cmd` sibling would close this. It is not shipped because it
could not be tested here: shipping an untested resolver for the platform we
cannot run is how "supported" becomes a claim rather than a fact. The CLI already
gates on a three-OS matrix, so there is a place to do it properly.
