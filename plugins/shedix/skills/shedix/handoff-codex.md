# Hand off a Codex session, so the person can close the laptop

For "Move this session to a shedix machine so I can close my laptop" from Codex. To keep working here as well, fork it
instead: https://shedix.dev/agents/fork.md.

**Done when:** Codex is running this conversation on the machine in tmux, at its prompt or working, not stopped at a
question, and the person knows the machine's name and where to find it. About 5 minutes.

Contents: move the work · copy the conversation · trust the folder · resume · check · tell the person · don't · if it
fails

**Before this:** the key, "Set up once" and the whole first step in https://shedix.dev/agents.md: the machine ready and
connected as `$NAME`, Codex signed in there (`signed codex`), and the person's answer about private files. Do the
whole handoff in this turn: if your sandbox needs the person's approval for the network, ask once, at the start, for
shedix and the machine together.

## 1. Move the work there, with its history

**Commit and push first,** so the machine starts from exactly this state: make a new branch (`git switch -c <name>`),
add the changed and new files by name (never one from the private list in the first step: check `git status --short`
before committing), commit, and push it when there's a remote. Then copy the project with its `.git`, which needs
nothing else. The private files the person kept back stay behind (`--exclude-from`, the list from the first step):

```bash
rsync -azL --exclude-from="$HOME/.config/shedix/leave-out-$NAME.txt" \
  --exclude=node_modules --exclude=.venv --exclude=venv --exclude=__pycache__ --exclude=target \
  --exclude=dist --exclude=build --exclude=.next --exclude=.turbo --exclude=.cache --exclude=coverage \
  --exclude='*.log' -e "ssh -F $HOME/.config/shedix/ssh/$NAME.conf" ./ ${NAME}:work/app/
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'cd ~/work/app && git log -1 --oneline && git status --short | head'
```

Name the folder after the project instead of `app`, and use that name in the steps below. **Tell the person exactly
which private files were sent** (the first step's list, less those left out), and which stayed behind.

A git worktree's `.git` is only a file pointing at the main checkout, so for a worktree, push the branch and clone it
into `~/work/app` on the machine instead (a private repository needs the machine connected to GitHub:
https://shedix.dev/agents/github.md), then send the private files the person approved, by name:

```bash
printf '%s\n' .env api/.env.local | rsync -azL --files-from=- -e "ssh -F $HOME/.config/shedix/ssh/$NAME.conf" ./ ${NAME}:work/app/
```

## 2. Copy the conversation

Codex keeps each conversation in `~/.codex/sessions`, in a folder for its day (`<year>/<month>/<day>`), as
`rollout-<time>-<id>.jsonl`. This one's id is in `$CODEX_THREAD_ID`; if that's empty, it's the newest file there whose first line has this project's folder as its
`cwd`. A conversation that was compacted or continued keeps its beginning in earlier files (`history_base` in the
first line names them), and Codex needs those too, so copy every file the conversation builds on, to the same place on
the machine:

```bash
ID=$CODEX_THREAD_ID
FILES=$(python3 - "$ID" <<'EOF'
import json, pathlib, sys
root = pathlib.Path.home() / ".codex" / "sessions"
todo, seen, files = [sys.argv[1]], set(), set()
while todo:
    tid = todo.pop()
    if tid in seen: continue
    seen.add(tid)
    for f in root.rglob(f"*{tid}*.jsonl"):
        files.add(str(f.relative_to(root)))
        base = (json.loads(f.open().readline()).get("payload") or {}).get("history_base") or {}
        if base.get("thread_id"): todo.append(base["thread_id"])
print("\n".join(sorted(files)))
EOF
)
echo "$FILES"
(cd ~/.codex/sessions && rsync -azR --rsync-path="mkdir -p ~/.codex/sessions && rsync" \
  -e "ssh -F $HOME/.config/shedix/ssh/$NAME.conf" $FILES ${NAME}:.codex/sessions/)
```

## 3. Let Codex trust the project folder

Codex stops at "Trust this folder?" the first time it runs in a git project, and in a terminal nobody is watching that
stops the whole handoff. The person asked you to move their work there, so add the folder to the machine's Codex
settings (running it again does no harm):

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'D=$HOME/work/app; mkdir -p ~/.codex; grep -qF "[projects.\"$D\"]" ~/.codex/config.toml 2>/dev/null || printf "\n[projects.\"%s\"]\ntrust_level = \"trusted\"\n" "$D" >> ~/.codex/config.toml'
```

## 4. Resume the conversation there

In a tmux session named after the project. When the person uses Codex in the ChatGPT app on this machine, Codex's
background service is running (the machine page turns it on: Agents, Codex in the ChatGPT app), and the session must
run inside it for the app to see it; the command below does that when it's there:

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "R=\$(pgrep -u \$USER -f '[c]odex app-server --remote-control' >/dev/null && echo '--remote unix://'); tmux new -d -s codex-app -c \$HOME/work/app \"codex resume \$R -C \$HOME/work/app $ID\""
```

## 5. Check it's running

```bash
sleep 5; ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'tmux capture-pane -p -t codex-app | grep -v "^\s*$" | tail -n 8'
```

You should see Codex's header with the folder, and its prompt. If it shows:

- **"Trust this folder?"**: the path in step 3 isn't the folder Codex runs in. Fix the path, end the session
  (`tmux kill-session -t codex-app`) and run step 4 again. Don't press keys in it for the person.
- **A sign-in screen:** Codex isn't signed in after all. End the session, do the first step's sign-in check (b), then
  run step 4 again.
- **No session:** Codex exited at once. Run step 4 again with `; sleep 300` after the `codex resume …` command, inside
  its quotes, so the session stays open, and look at it as above to read the error.

## 6. Tell the person

- The machine's name. On the shedix page: the machine, then Terminals, then `codex-app`, or straight from any browser at
  `https://shedix.dev/m/$NAME/codex?t=codex-app`. It's this conversation, and they can type into it.
- In the ChatGPT app, when it's set up for this machine: Codex, then the machine.
- Which private files were sent, and which stayed behind.
- To stop the laptop's copy now, so the two don't drift apart; then they can close the laptop.

## Don't

- Research how Codex stores or resumes conversations, or write your own scripts for it: these steps are complete.
- Type the key into a program's prompt, or write it anywhere but `~/.config/shedix/config.json`.
- Answer Codex's questions on the machine for the person.

## If it fails

| You see | Do |
|---|---|
| `curl: (6) Could not resolve host` on this computer | Your sandbox blocks the network: ask the person once for network access for the whole handoff. |
| `no rollout found for thread id` | The conversation's files aren't under `~/.codex/sessions` on the machine: check step 2's list, and that each file kept its `<year>/<month>/<day>` folders. |
| `ssh` exits with 255 | The machine went to sleep: start it, wait, and write its SSH settings again (agents.md). |
