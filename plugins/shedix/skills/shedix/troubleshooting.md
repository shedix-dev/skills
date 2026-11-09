# When something fails

Find what you see, then do what the row says. If nothing here fits, the error's own words usually say what's wrong:
tell the person, quoting them.

## Reaching shedix and the machine

| You see | Do |
|---|---|
| `curl: (6) Could not resolve host` | Your sandbox blocks the network (Codex's does by default). Ask the person once for network access for the whole task, not per command. |
| `401` | The key is wrong or revoked. A saved key: remove it (`python3 ~/.config/shedix/key.py forget`) and get a new one with a link code; a pasted one: tell the person (agents.md, "The key"). |
| `403` | The account isn't invited yet, or is suspended: tell the person. |
| `key.py wait` keeps saying it's still waiting | The person hasn't approved yet: run it again until the code expires (10 minutes), then ask before starting a new one. |
| `The person denied it.` or `The code expired.` | Tell the person; start a new link code only if they want one. |
| `shedix answered 404` from `key.py link` | This shedix doesn't have link codes yet: ask the person to paste a key from https://shedix.dev/profile#access (Tokens, New token). |
| `402` or `409` from creating or starting | Credit or a limit: https://shedix.dev/agents/reference.md, "Credit and limits". |
| `ssh` exits with 255, or connecting times out | The machine stopped (it sleeps after an hour unused), or started again with a new IP. Look it up, start it if it's stopped, wait with `?wait=240`, and write its SSH settings again (agents.md). |
| `Permission denied (publickey)` | shedix's key isn't on the machine: add it with `POST …/ssh-keys` (agents.md, first step (a)). |
| `<name> is not ready: …` from the printer | It's still booting: wait with `?wait=240`. |

## Running things

| You see | Do |
|---|---|
| A command stopped after 2 minutes (Claude Code) | Give it a longer timeout (up to 10 minutes) or run it in the background. Check `tmux ls` and the logs before running it again: the first run may still be going. |
| An install hangs | It's waiting for an answer nobody can give in tmux: add `< /dev/null` so it fails at once, then use its non-interactive flag. |
| `port … never opened` | Read the service's log (`tail -n 40 ~/work/<session>.log`): the service crashed or listens on another port (`ss -Hltnp` shows which). |
| A forward fails at once | The port is taken on this computer: ask the person to stop the local app, or forward to another local port (`-L 13000:localhost:3000`). |
| `zsh: no matches found` or a path cut at a colon | zsh: quote globs, and put braces round a variable before a colon (`${NAME}:work/`). |
| `rsync: unrecognized option` on macOS | macOS's rsync is old: use only the flags in run-project.md. |

## Handing off or forking a session

| You see | Do |
|---|---|
| Codex on the machine shows "Trust this folder?" | Add the project's folder to the machine's `~/.codex/config.toml`, then start the session again (handoff-codex.md, step 3). |
| Claude Code on the machine asks for a theme or to trust the folder | The person answers at `https://shedix.dev/m/<machine>/claude?t=<tmux session>`, which opens that session; don't answer the trust question yourself. |
| `signed claude` still says `claude=off` after the person signed in | Give it a few seconds (the machine reports every couple of seconds) and poll again; if the sign-in page showed an error, send the person the `#signin` link again (agents.md, first step (b)). |
| `duplicate session` from `tmux new` | A session by that name is already there (the person may have opened one on the page): use another name, like `claude-handoff-2`. |
| The person chose an option on a trust screen but nothing happened | Those screens need the keyboard: the person presses Enter in the terminal (a tap on the text doesn't choose it). |
| `no rollout found for thread id` (Codex) | The conversation's files aren't in `~/.codex/sessions` on the machine with their dated folders: copy them as handoff-codex.md, step 2, says. |
| A sign-in screen in the session | The agent isn't signed in on the machine: end the session, do the first step's sign-in check (b) with its `#signin` link, then start the session again. |
| The phone opens the laptop's conversation, not the machine's (Claude Code) | The copy kept `bridgeSessionId`: copy it again without that field (handoff-claude-code.md, step 2). |
| The machine went to sleep while waiting for the person | An agent waiting at a question isn't activity. Start the machine, then run the start step again. |
| A private file arrived that the person said no to | Remove it on the machine (`rm`), tell the person, and check the copy used `--exclude-from` with the first step's list (paths start with `/`, relative to the project's folder). |
