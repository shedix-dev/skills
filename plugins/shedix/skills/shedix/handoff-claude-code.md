# Hand off a Claude Code session, so the person can close the laptop

For "Move this session to a shedix machine so I can close my laptop" from Claude Code. To keep working here as well,
fork it instead: https://shedix.dev/agents/fork.md.

**Done when:** Claude Code is running this conversation on the machine, in tmux, with Remote Control on, and the
person knows the machine's name and where to find the session. About 5 minutes.

**Before this:** the key, "Set up once" and the whole first step in https://shedix.dev/agents.md: the machine ready and
connected as `$NAME`, Claude Code signed in there, and the person's answer about private files.

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

This conversation's id is in `$CLAUDE_CODE_SESSION_ID`. Claude Code keeps it at
`~/.claude/projects/<folder>/<id>.jsonl` (if the variable is empty, it's the newest `.jsonl` in this project's folder,
whose name is the project's path with every character that isn't a letter or digit turned into `-`). On the machine
the project is `~/work/app`, so its folder is named after that path.

**Don't copy the file as it is.** A conversation started in the Claude desktop app, or with Remote Control on,
carries the ID of its claude.ai session (`bridgeSessionId`). A copy that keeps it takes over that link: the phone opens
the laptop's conversation, and the machine's never shows up. Copy it without that field:

```bash
ID=$CLAUDE_CODE_SESSION_ID
SRC=$(find ~/.claude/projects -maxdepth 2 -name "$ID.jsonl" | head -n 1); echo "$SRC"
DIR=$(ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'd=$(echo ~/work/app | sed "s/[^a-zA-Z0-9]/-/g"); mkdir -p ~/.claude/projects/$d; echo $d')
python3 -c 'import json, sys; [print(json.dumps({k: v for k, v in json.loads(l).items() if k != "bridgeSessionId"}, ensure_ascii=False)) for l in sys.stdin if l.strip()]' \
  < "$SRC" | ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "cat > ~/.claude/projects/$DIR/$ID.jsonl"
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "grep -c bridgeSessionId ~/.claude/projects/$DIR/$ID.jsonl"   # 0
```

## 3. Resume the conversation there

In tmux, with Remote Control on and a name the person will find in the Claude app (the project's name and "from
laptop"):

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "tmux new -d -s claude-handoff -c ~/work/app 'claude --resume $ID --remote-control \"app (from laptop)\"'"
```

The first time Claude Code runs in this folder it may ask a colour theme and whether to trust the folder. Don't answer
the trust question yourself: give the person the session's link, `https://shedix.dev/m/$NAME/claude?t=claude-handoff`, which
opens this tmux session in their browser.

## 4. Check it's running

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'tmux capture-pane -p -t claude-handoff | grep -v "^\s*$" | tail -n 8'
```

You should see the conversation and Claude Code's prompt, or the questions above waiting for the person. If tmux has
no session `claude-handoff`, Claude Code exited at once: check the conversation's file is in
`~/.claude/projects/$DIR/` on the machine under the same id, then run step 3 again.

## 5. Tell the person

- The machine's name, and that this conversation now runs there as a new session in the Claude app, separate from the
  laptop's: in the app's list of sessions under the name above (not under the machine in Devices, which lists only
  sessions started from its phone access), or from any browser at `https://shedix.dev/m/$NAME/claude?t=claude-handoff`.
- Which private files were sent, and which stayed behind.
- To stop the laptop's copy now, so the two don't drift apart; then they can close the laptop.
