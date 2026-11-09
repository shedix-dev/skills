# Fork this session to a machine

For "Fork this session to a shedix machine". A fork keeps this conversation going here, and a copy of it, with the
project as it is now, carries on separately on the machine: its own session ID, its own entry in the Claude app named
with "(fork)", its own git branch. To move the work instead (so the person can close the laptop), follow the handoff
file for your agent (agents.md, "Which job").

- **Claude Code:** sections 1 to 5 below.
- **Codex:** section 1, then "Codex" at the end (`codex fork`).
- **Cursor and other agents** can't fork a conversation to another computer: say so, and offer the handoff, which
  writes what you know into HANDOFF.md for an agent on the machine while you carry on here
  (https://shedix.dev/agents/handoff-other.md).

**Done when:** the fork is running on the machine in tmux, the conversation here is unchanged, and the person knows
the machine's name, the fork's name and where to find it. About 5 minutes.

**Before this:** the key, "Set up once" and the whole first step in https://shedix.dev/agents.md: the machine ready and
connected as `$NAME`, the agent signed in there (`signed claude`, or `signed codex` for Codex), and the person's answer
about private files.

## 1. Copy the project as it is, without touching it here

The work here goes on, so don't commit, switch branches or stash here. Copy the working tree with its `.git`
(uncommitted changes arrive as uncommitted changes), leaving out the private files the person kept back, then give the
fork its own branch on the machine:

```bash
rsync -azL --exclude-from="$HOME/.config/shedix/leave-out-$NAME.txt" \
  --exclude=node_modules --exclude=.venv --exclude=venv --exclude=__pycache__ --exclude=target \
  --exclude=dist --exclude=build --exclude=.next --exclude=.turbo --exclude=.cache --exclude=coverage \
  --exclude='*.log' -e "ssh -F $HOME/.config/shedix/ssh/$NAME.conf" ./ ${NAME}:work/app/
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'cd ~/work/app && b=$(git branch --show-current) && git switch -c "${b:-work}-fork" && git status --short | head'
```

Name the folder after the project instead of `app`. In a git worktree (`.git` is a file), add `--exclude=.git`, then
`git init` and commit on the machine, or push a branch and clone it there (https://shedix.dev/agents/github.md). Tell the
person which private files were sent, and which stayed behind.

## 2. Copy the conversation (Claude Code)

This conversation's id is in `$CLAUDE_CODE_SESSION_ID` (if it's empty: the newest `.jsonl` in this project's folder
under `~/.claude/projects`). Copy it without `bridgeSessionId`, the link to its claude.ai session: a copy that keeps
it would take over the laptop's entry in the Claude app. Only this computer's file is read; nothing here changes.

```bash
ID=$CLAUDE_CODE_SESSION_ID
SRC=$(find ~/.claude/projects -maxdepth 2 -name "$ID.jsonl" | head -n 1); echo "$SRC"
DIR=$(ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'd=$(echo ~/work/app | sed "s/[^a-zA-Z0-9]/-/g"); mkdir -p ~/.claude/projects/$d; echo $d')
python3 -c 'import json, sys; [print(json.dumps({k: v for k, v in json.loads(l).items() if k != "bridgeSessionId"}, ensure_ascii=False)) for l in sys.stdin if l.strip()]' \
  < "$SRC" | ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "cat > ~/.claude/projects/$DIR/$ID.jsonl"
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "grep -c bridgeSessionId ~/.claude/projects/$DIR/$ID.jsonl"   # 0
```

## 3. Start the fork

`--fork-session` makes Claude Code start a new session from the conversation instead of continuing it, and
`--session-id` gives that new session an ID chosen here, so you know it. Name it after the work, ending in "(fork)":
that's the name the Claude app shows (Remote Control's name), beside the original. `--fork-session` doesn't add
"(fork)" by itself, so the name must carry it. If the person said what the fork
should work on, add it in quotes at the end as its first message; otherwise it waits at its prompt for them.

```bash
FORK=$(python3 -c 'import uuid; print(uuid.uuid4())'); echo "$FORK"
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "tmux new -d -s claude-fork -c ~/work/app 'claude --resume $ID --fork-session --session-id $FORK --remote-control \"signup form (fork)\"'"
```

For a second fork of the same conversation, use another tmux name (`claude-fork-2`) and a new `FORK`. The first time
Claude Code runs in this folder it may ask a colour theme and whether to trust the folder: don't answer the trust
question yourself; the person answers at `https://shedix.dev/m/$NAME/claude?t=claude-fork`.

## 4. Check it's running

```bash
sleep 5; ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'tmux capture-pane -p -t claude-fork | grep -v "^\s*$" | tail -n 8'
```

You should see the conversation and Claude Code's prompt, or the questions above waiting for the person. If there's no
session `claude-fork`, Claude Code exited at once: run step 3 again with `; sleep 300` after the `claude …` command,
inside its quotes, and look again to read the error. If an older Claude Code refuses `--session-id` with
`--fork-session`, run step 3 without `--session-id $FORK`: the fork then picks its own ID (the newest file in
`~/.claude/projects/$DIR/`).

## 5. Tell the person

- The machine's name, and that the fork runs there as its own session: in the Claude app under the name above (with
  "(fork)"), separate from this one, or from any browser at `https://shedix.dev/m/$NAME/claude?t=claude-fork`.
- That this conversation is untouched and carries on here, on its own files; the fork works on the branch
  `<branch>-fork` on the machine (to bring its work back, push that branch from the machine once it's connected to
  GitHub, or fetch it over SSH).
- Which private files were sent, and which stayed behind.
- That the machine sleeps after an hour unused, and keeps everything.

## Codex

Codex forks too: `codex fork <id>` starts a new conversation, with its own id, from an existing one (`/fork` does the
same inside Codex). The fork's file holds only a pointer into the original's history, so the original's files must
stay on the machine, and the fork works in the folder it's started in (`~/work/app`, not the laptop's path). After
section 1:

1. **Copy the conversation's files** to the machine exactly as https://shedix.dev/agents/handoff-codex.md, step 2, says
   (`ID=$CODEX_THREAD_ID`), and **let Codex trust the folder** as its step 3. Only this computer's files are read.
2. **Start the fork** in tmux, inside Codex's background service when the ChatGPT app is set up for this machine. If the
   person said what the fork should work on, add it in quotes after `$ID`:

   ```bash
   ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "R=\$(pgrep -u \$USER -f '[c]odex app-server --remote-control' >/dev/null && echo '--remote unix://'); tmux new -d -s codex-fork -c \$HOME/work/app \"codex fork \$R -C \$HOME/work/app $ID\""
   sleep 5; ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'tmux capture-pane -p -t codex-fork | grep -v "^\s*$" | tail -n 8'
   ```

   What you see, and what to do if it's a trust screen, a sign-in or nothing, is as in handoff-codex.md, step 5.
3. **Tell the person** the machine's name; that the fork is a new conversation, at
   `https://shedix.dev/m/$NAME/codex?t=codex-fork` from any browser (whether and under what name it shows in the ChatGPT
   app isn't confirmed yet: don't promise it); that this conversation carries on here untouched; and which private
   files were sent.
