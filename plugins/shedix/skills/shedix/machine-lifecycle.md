# Sleep, keep awake, start, stop and delete

**Done when:** the machine is in the state the person asked for, and you've told them. Seconds.

**Before this:** the key and "Set up once" in https://shedix.dev/agents.md, and the machine's name, as `$NAME`. Only do any of
this when the person asks; never sleep or stop a machine they're using without being told to.

## Machines go to sleep after an hour unused

A machine counts as in use while an SSH session or port forward is connected, a terminal or VS Code tab is open,
Claude Code is in the middle of a turn or Codex is writing its conversation, or its CPU or network is busy (an install
counts). After an hour with none of that it goes to sleep by itself. A sleeping machine keeps everything: on start, what was running (agents, dev servers, terminals) carries on where it was. Only for
long work nobody is watching (a job in tmux with nobody connected and little CPU), keep it awake, and turn that off
when you're done:

```bash
curl -sS -X PATCH "$SHEDIX_URL/api/machines/$NAME" -H "Authorization: Bearer $SHEDIX_TOKEN" \
  -H "Content-Type: application/json" -d '{"keepAwake": true}' | python3 ~/.config/shedix/show.py
```

## Put it to sleep, now or later

When the person tells you to put a machine to sleep, now or later ("sleep it when you're done", "sleep it at
midnight"), set its **sleep timer**. It puts the machine to sleep at that time whatever is running, even with keep awake on,
within a minute; running programs end and files stay. From an agent **on the machine** (no key needed; machines
made before 2026-11-02 don't have this command, so use the API call below):

```bash
shedix sleep            # in 2 minutes: run it as your very last step, after telling the person you're done
shedix sleep 3h         # or 45m, 1h30m, 2d
shedix sleep at 23:30   # the machine's clock is UTC; "at 23:30 +0530" for another zone
shedix sleep cancel
```

From anywhere else, with the key: `{"sleepIn": minutes}` (0 for now), or `{"sleepTimer": "<ISO time>"}`; `null` for
either cancels it. The machine's `sleepTimer` field says when it's set for. Starting or stopping a machine clears it.

```bash
curl -sS -X PATCH "$SHEDIX_URL/api/machines/$NAME" -H "Authorization: Bearer $SHEDIX_TOKEN" \
  -H "Content-Type: application/json" -d '{"sleepIn": 120}' | python3 ~/.config/shedix/show.py
```

## Start, stop and delete

```bash
curl -sS -X POST "$SHEDIX_URL/api/machines/$NAME/start?wait=240" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 ~/.config/shedix/show.py  # same name, new IP
curl -sS -X POST "$SHEDIX_URL/api/machines/$NAME/stop" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 ~/.config/shedix/show.py            # puts it to sleep; everything kept
curl -sS -X DELETE "$SHEDIX_URL/api/machines/$NAME" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 ~/.config/shedix/show.py               # gone for good
```

A started machine has a new IP: write its SSH settings again (agents.md, first step (a)). A sleeping machine costs cents a
day ($0.006/hr for an `s`) and keeps everything, including what was running, so prefer putting a project's machine
to sleep to deleting it. Delete machines you started for a one-off job, unless the person asked to keep them.
