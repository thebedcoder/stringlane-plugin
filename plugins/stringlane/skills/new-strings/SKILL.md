---
name: new-strings
description: What to do with a user-visible string you are about to write into code — where the key goes, and why the base locale is only half of it. Use when adding a new UI string, label, button, error or empty-state text, when your code references a key the project does not have, when you are about to leave a hardcoded string in a component, or after add_key returns.
---

# A new string is not finished when the base locale has it

You are writing a component, and it needs a sentence a user will read. Three
things can happen to that sentence, and only one of them is right.

**It stays a literal in the component.** The project is localized; every other
string in it is a key. This one is now invisible to every locale, every check and
every translator, and nothing will report it — the guard watches locale files,
and you never touched one.

**You type it into the locale file.** `add_key` exists because that hand edit
skips the key-existence check, the lock, and the place *this format* keeps a
description — and key names are not equally legal across formats.

**You call `add_key`.** Right, and half done.

## The half that gets dropped

`add_key` writes the **base locale only**. That is correct and it is the whole of
what it claims. Every other locale now reports the key as missing, and until they
have it, your users see English — with a green build, a passing test run, and a
tool call that returned success.

So the completion rule is not "the key exists". It is:

> The string is done when every locale has it, or the user has said not to
> translate it yet.

The result you just got back says how many are waiting: `localesToTranslate`.
Read it, and say the number out loud to the user rather than reporting "added the
string".

## Do not go looking for the sequence here

Translating those locales is the StringLane server's own three-step workflow, and
it is described in the server instructions your session already has, in
`/stringlane:translate <locale>`, and in the tool descriptions themselves. It is
deliberately not repeated in this file: two copies of one workflow means one that
nobody updates.

What this file adds is the part no tool description can reach — the moment
*before* any of them, when the string is still a literal in your editor.

## While you still know what it means

How to call `add_key` — everything in one call, each key with a description — is
in the server's own instructions and in the tool's description, and is not
repeated here for the same reason the sequence is not. What belongs here is
*when*: you are the only one who will ever know whether `Close` is a button or a
verb, and that decides the word in German. Write the description now, in the
call, not as a later pass that never happens.

## Renaming and removing

A rename is a new key plus a removal, not an edit — the old key is still
referenced by every locale file that has it, and translations do not follow a
rename. Removing a key that other locales still carry leaves them holding a
string nothing displays; that is a hand edit across every file, so ask before
doing it rather than doing it quietly.
