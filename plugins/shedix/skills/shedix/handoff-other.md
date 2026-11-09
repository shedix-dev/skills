# Hand off from Cursor or any other agent, so the person can close the laptop

For "Move this session to a shedix machine so I can close my laptop" from Cursor or another agent whose conversation
can't move ("Fork this session" from such an agent works the same way: write HANDOFF.md, and keep working here too).

**Done when:** Claude Code or Codex is working on the machine from a written handoff, in tmux, and the person knows the
machine's name and where to find it. About 5 minutes.

Your conversation can't move to another computer, so write down what it knows, and an agent on the machine continues
from that. The machine has Claude Code and Codex: **ask the person which one to use first** (Claude Code if they don't
mind), since the first step checks that one is signed in there.

**Before this:** the key, "Set up once" and the whole first step in https://shedix.dev/agents.md: the machine ready and
connected as `$NAME`, the chosen agent signed in there (`signed claude` or `signed codex`), and the person's answer
about private files.

## 1. Write HANDOFF.md in the project

Write it for an agent that has seen nothing of this conversation:

- **Goal:** what the person wants, in their words.
- **Done so far:** what's finished, and how you checked it.
- **Decisions:** what was decided and why, and what was ruled out.
- **Next:** the remaining steps, in order, and how to check each one.
- **Run and test:** the commands, ports and test data.
- **Open questions** for the person.

Commit it with the rest of the work (it's for the machine, so leave secrets out of it).

## 2. Move the work there, with its history

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

## 3. Start the agent there

**Claude Code**, in tmux, with Remote Control on so the person can follow it in the Claude app:

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "tmux new -d -s claude-handoff -c ~/work/app 'claude --remote-control \"app (from laptop)\" \"Read HANDOFF.md and continue from Next. Tell me what you are doing.\"'"
```

The first time Claude Code runs in this folder it may ask a colour theme and whether to trust the folder: the person
answers them at `https://shedix.dev/m/$NAME/claude?t=claude-handoff`, which opens this session in their browser.

**Codex.** Let Codex trust the project folder, then start it in tmux, inside Codex's background service when the
ChatGPT app is set up for this machine:

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'D=$HOME/work/app; mkdir -p ~/.codex; grep -qF "[projects.\"$D\"]" ~/.codex/config.toml 2>/dev/null || printf "\n[projects.\"%s\"]\ntrust_level = \"trusted\"\n" "$D" >> ~/.codex/config.toml'
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "R=\$(pgrep -u \$USER -f '[c]odex app-server --remote-control' >/dev/null && echo '--remote unix://'); tmux new -d -s codex-app -c \$HOME/work/app \"codex \$R -C \$HOME/work/app 'Read HANDOFF.md and continue from Next. Tell me what you are doing.'\""
```

## 4. Check it's running

```bash
sleep 5; ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'tmux capture-pane -p -t claude-handoff | grep -v "^\s*$" | tail -n 8'   # or -t codex-app
```

You should see the agent reading HANDOFF.md or working on it, or Claude Code's first questions waiting for the person.

## 5. Tell the person

- The machine's name, and where it is: `https://shedix.dev/m/$NAME/claude?t=claude-handoff` (or
  `https://shedix.dev/m/$NAME/codex?t=codex-app`) from any browser; or the Claude app (by the name above) or the ChatGPT app
  (Codex, then the machine), when they use those.
- Which private files were sent, and which stayed behind.
- That it's a new session that has read HANDOFF.md, not this conversation.
- To stop the agent on the laptop, so the two don't work on the same thing.
