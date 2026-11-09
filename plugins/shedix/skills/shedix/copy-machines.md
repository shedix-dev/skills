# Split tasks across machines, or copy a machine

For "Split these tasks across shedix machines, one each", and for copying a machine.

**Done when:** each task has a Claude Code session working on it on a machine of its own, each session will put its
machine to sleep when its task is done, and the person has every machine's name. About 5 minutes, then the tasks'
own time. For a plain copy: the copies are ready and you've told the person their names.

**Before this:** the key, "Set up once" and the whole first step in https://shedix.dev/agents.md, for the project's machine
(the source; size `s` unless the tasks need more): ready and connected as `$NAME`, Claude Code signed in there
(`signed claude`), and the person's answer about private files. Every copy then starts signed in, with the project.

## Split tasks, one machine each

1. **The tasks.** Use the ones the person listed; if they haven't, ask for them. Give each a short name in lower-case letters,
   digits and dashes (`dark-mode`, `flaky-login`): it names its branch, its tmux session and its entry in the Claude app.
2. **Put the project on the source machine,** with its history, as https://shedix.dev/agents/run-project.md ("Move a
   project onto it") says: commit and push what you have first, then one `rsync` with `.git`, leaving out the private
   files the person kept back (`--exclude-from="$HOME/.config/shedix/leave-out-$NAME.txt"`). Tell the person which
   private files were sent.
3. **Copy the source once per task but one** (the source takes the first task), as below, with each copy labelled by
   its task. An account runs at most 3 machines at once: if a `409` stops you, put the remaining tasks in git worktrees
   on the machines you have (`git worktree add ~/work/app-<task> -b <task>`), one session each. Tell the person each
   machine's name as it starts. Wait for each, and write its SSH settings (agents.md, first step (a)).

**4. On each machine, start the task's session,** with `NAME` set to that machine. Write the task into a file there
(it keeps quoting simple), make its branch, and start Claude Code in tmux with Remote Control, named after the task.
The task ends by putting the machine to sleep, so each one stops when its task is done:

```bash
T=dark-mode
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "cat > ~/work/task-$T.md" <<'EOF'
Add a dark mode to the settings page. (The task, in the person's words, and how to check it's done.)
When it's done and checked, commit it on this branch (and push it, if this machine is connected to GitHub), say what
you did, then run `shedix sleep` as your very last step.
EOF
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "cd ~/work/app && git switch -c $T && tmux new -d -s $T -c ~/work/app 'claude --remote-control \"$T\" \"Read ~/work/task-$T.md and do it.\"'"
sleep 5; ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME "tmux capture-pane -p -t $T | grep -v '^\s*$' | tail -n 8"
```

The first time Claude Code runs in a folder it may ask a colour theme and whether to trust the folder: give the
person the session's link, `https://shedix.dev/m/<machine>/claude?t=<task>`, and don't answer the trust question
yourself. From Codex, start `codex` the same way after letting it trust the folder (handoff-codex.md, step 3).

**5. Tell the person** every machine's name with its task; where to find each session (the Claude app's list, by the
task's name, or its link above); that each machine goes to sleep when its task is done, keeping everything; and
which private files were sent.

## Copying a machine

A machine that's set up (code, installed packages, Claude Code or Codex signed in) can be copied, to give several
agents the same starting point or to try something without touching the original. The copies start from its whole
disk as it is now, so nothing needs installing or signing in again: every sign-in kept on the machine comes along
(Claude Code, Codex, `gh`, Docker, npm, cloud CLIs, git credentials, SSH keys), root's too. They keep its stack,
project and keys (and its size, unless you ask for another), and get their own names, addresses, SSH host keys and
secrets; programs that were running don't carry over, so start them on each copy.

- **Sign in first.** If the copies are for agents, the first step's sign-in check (b) on the source comes before
  copying it (and anything else the work needs: GitHub, https://shedix.dev/agents/github.md); every copy then starts signed in.
- **One identity per machine** (a Tailscale node, a named tunnel) is copied too, so the copies share it; sign it out on
  a copy, or give each its own, when that matters. Claude Code's phone access is the exception: each copy shows up in
  the Claude app as a machine of its own.
- **A new machine in a project** starts the way the shedix page starts one: as a copy of the project's oldest machine,
  so it has the project's setup. Use a fresh machine only when the person asks for one.
- **Size:** copies keep the source's size unless you send another: `"size"` (`s`, `m`, `l`, `xl`, or `custom` with
  `"cpus"` and `"ramGb"`) and `"diskGb"`. That's also how to move to a bigger or smaller machine, since a machine's
  own size can't change. The whole disk is copied, so `diskGb` can't be smaller than the source's (a 400 says which
  sizes fit). The source's `diskUsedGb` says how full its disk is, and `memoryUsedGb` how much memory is in use while
  it runs; check them before copying to a smaller machine, and tell the person if the project won't fit.

```bash
curl -sS -X POST "$SHEDIX_URL/api/machines/$NAME/copy" -H "Authorization: Bearer $SHEDIX_TOKEN" \
  -H "Content-Type: application/json" -d '{"count": 3, "label": "api worker"}' | python3 ~/.config/shedix/show.py
# the same machine on an L with a 90 GB disk:
curl -sS -X POST "$SHEDIX_URL/api/machines/$NAME/copy" -H "Authorization: Bearer $SHEDIX_TOKEN" \
  -H "Content-Type: application/json" -d '{"size": "l", "diskGb": 90}' | python3 ~/.config/shedix/show.py
```

The call answers within about 30 seconds with the new machines, up to 5 at a time. One copy is a clone of the disk,
ready in about a minute; several wait for a snapshot first (`setup=copying`) and are ready in three to six minutes.
Wait for each with `?wait=300`, then write its SSH settings (agents.md, first step (a)). Before copying, shedix has the
source write everything in memory to its disk, so files written up to that moment come along (`flushed: True`); an
older machine can't (`flushed: False`), so check that what matters arrived. Labels are numbered ("api worker", "api
worker 2"…); without one, copies are called after the source ("acme web copy"). If the call was cut off, list the
project's machines before trying again: the copies may exist already.
