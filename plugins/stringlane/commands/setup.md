---
description: Check that StringLane actually works here — the CLI, the MCP server, the write guard — and fix what does not
---

Find out whether StringLane is actually working in this session, and fix what is
not. **Run the checks in order.** Each failure explains the next one, and
diagnosing them out of order is how you get a confident wrong answer.

You install nothing without being told to. Propose the exact command, wait for a
yes, then run it once.

## 1. Read the session-start warning, if there is one

Before running anything, check whether this session already told you. StringLane
registers a `SessionStart` hook, and when the resolver behind it finds no
StringLane — or finds one too old to run the guard — it says so once, in a
`systemMessage`:

> StringLane: no CLI found, so the write guard and the MCP server are not
> running. Run /stringlane:setup.

> StringLane: the installed CLI is too old for this plugin, so the write guard
> is not running. Run /stringlane:setup.

Either one is a **diagnosis, not a symptom** — it names which of the two failures
happened, so skip to the fix. The first is step 2, the second is step 3.

**No warning is also an answer**, and it is the useful one: the resolver ran, it
found a StringLane, and that StringLane answered. Steps 2 and 3 are then already
known to pass and you can go straight to step 4.

If you cannot see the start of this session — it was compacted, or you were
handed it mid-way — run the two checks below rather than guessing.

## 2. Is StringLane reachable at all?

```sh
stringlane --version
```

Four outcomes. Only the first is success, and the second is the one that gets
misread:

- **A version number.** Go to step 3.
- **Nothing at all, and exit 0.** StringLane is **not** installed. The plugin
  puts a resolver on this session's `PATH`, and when it finds no StringLane
  anywhere it deliberately succeeds in silence: a resolver that failed loudly
  would look to the hook protocol like a guard refusing an edit, so every user
  without StringLane would find their file writes blocked. Silence is the
  designed way of saying "found nothing". **Do not read exit 0 as installed.**
  (This is the case the `SessionStart` warning above exists to announce, because
  silence here is indistinguishable from success.)
- **An `npm` or `npx` error** — a 404 for the package, or no network. Nothing is
  installed here and the registry could not supply it either. Say which, and
  stop; retrying changes neither.
- **`command not found`.** The resolver is not on `PATH`, which almost always
  means this session started before the plugin was enabled. Ask the user to
  restart Claude Code, and stop — nothing below is meaningful until it is back.

For the second outcome, offer both and let the user choose. They are different
decisions, not two spellings of one:

```sh
npm install -g @stringlane/cli
```

is per machine and covers every project. A dev dependency in this repository
pins one version for everyone who clones it, and the resolver finds a project's
own copy without any global install. Then run the version check again. If it is
still silent, report that and stop rather than installing a second time.

## 3. Is the StringLane you found new enough?

```sh
stringlane --help
```

The command list must include `hook` and `add-key`. If either is missing, what
resolved is older than this plugin, and the write guard is dead: every hook here
calls `stringlane hook`, and a build without it answers "unknown command".

**That failure is silent to the edit that triggers it, by design.** The resolver
guarantees an old CLI cannot block your edits — otherwise upgrading the plugin
would freeze every session running an old CLI — so from the edit's side an
unusable guard and a working one look exactly alike. The `SessionStart` warning
in step 1 is what makes it visible at all; this step is how you confirm it when
that message is not in front of you.

The fix is an upgrade, at whichever of the two places step 2 describes the old
copy came from.

## 4. Is the MCP server connected?

Look at your own tools. If `inspect_project` is among them, the server is
running and there is nothing to do.

If it is not: this plugin registers the server, so the usual cause is again a
session older than the plugin — restart. If it is still absent after a restart,
the server could not start, and step 2 is why.

To register it somewhere this plugin is not installed — another host, or a
Claude Code without it:

```sh
stringlane setup mcp --host claude-code
```

prints the entry and writes nothing. `--apply` merges it in, backing the file up
first. The JSON is the same wherever your agent keeps its MCP servers.

## 5. Is the write guard live?

```sh
stringlane setup hooks --host claude-code
```

prints the hooks and writes nothing.

**`--host` is the host you are actually running in**, and it is not always
`claude-code`. This plugin ships one hooks document, in Claude Code's schema,
which Claude Code loads and Codex's discovery is documented to find — so on both
of those `--host claude-code` is the right question to ask. **On Gemini CLI it
is not**: the extension declares no hooks at all, so the guard is not installed
by the plugin and `--host gemini-cli` is what to run. Same for anyone here
through the CLI rather than the plugin — `--host cursor`, `--host vscode`.

Every one of those prints. Only `--host claude-code` writes with `--apply`
today; the rest print and you paste, which is the same bytes. `--remove` works
everywhere.

**With this plugin installed, do not apply them.** The plugin registers exactly
these already, and a second copy in the settings file makes every one of them
fire twice — two prompts on every edit, from a guard whose whole value is that
people leave it on. `--apply` is for a project that wants the guard *without*
the plugin, or a team that wants it committed:

```sh
stringlane setup hooks --host claude-code --scope project --apply
```

## 6. Then the project, which is a different question

Everything above asks whether StringLane works on this machine. Whether *this
project* is set up — a config, a base locale, what is translated — is
`/stringlane:init`, and it is described there.

Do not repeat any of it here. Run the checks in order, report what you found in a
short list with the fix for anything that failed, and hand off.
