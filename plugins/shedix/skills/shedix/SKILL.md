---
name: shedix
description: Starts and drives the person's shedix cloud Linux machines over curl and ssh. Use when the person asks to move or fork this session to a shedix machine (so they can close the laptop), split tasks across shedix machines, "use shedix for this", run or test their project on a machine, work on a fresh Ubuntu machine, connect a machine to GitHub, use the browser on a machine, or start, sleep, stop or delete a shedix machine; and for any prompt that names shedix.dev/SKILL.md or shedix.dev/agents.md.
license: Proprietary. Free to use for working with shedix.
compatibility: bash or zsh, curl, ssh and python3, on macOS or Linux, with network access to shedix.dev and to the machine (SSH, port 22).
metadata:
  version: "2026-11-06"
---

# shedix for agents

shedix gives the person cloud Linux machines of their own: Ubuntu 24.04 with sudo, Docker, Node 22, Python 3, git and
gh, and Claude Code and Codex installed. You drive them with `curl` and `ssh`; there is nothing to install. Every
command here works as written in bash or zsh, on macOS and Linux.

**How to use this guide:** read this file to the end, run "Set up once", find the key ("The key"), do "The first
step", then follow the one job file the request needs. Fetch files with `curl -fsSL` (tools that summarize pages drop
the commands). Together they're complete for shedix: don't search the web or write your own tools for it.

Contents: which job · the key · set up once · the first step ((a) the machine, (b) sign-in, (c) private files) · rules ·
notes for your agent

## Which job

| The person said, or wants | Read |
|---|---|
| "Move this session to a shedix machine so I can close my laptop" | The handoff for your agent: https://shedix.dev/agents/handoff-claude-code.md (Claude Code), https://shedix.dev/agents/handoff-codex.md (Codex), https://shedix.dev/agents/handoff-other.md (Cursor or any other agent) |
| "Fork this session to a shedix machine" (keep working here, and a branch of this conversation runs there) | https://shedix.dev/agents/fork.md |
| "Split these tasks across shedix machines, one each" | https://shedix.dev/agents/copy-machines.md |
| "Use shedix for this": run the project or its tests there | https://shedix.dev/agents/run-project.md |
| "Use shedix for this": anything else (try something on a clean machine, do the work there instead of here) | https://shedix.dev/agents/sandbox.md |
| A browser on the machine: a login or a human step, or an agent driving a real browser there | https://shedix.dev/agents/browser.md |
| The machine cloned from or pushing to GitHub | https://shedix.dev/agents/github.md |
| A machine kept awake, put to sleep, started, stopped or deleted | https://shedix.dev/agents/machine-lifecycle.md |
| The operating system alone (Ubuntu or another Linux, nothing added) | https://shedix.dev/agents/plain-machines.md |
| Machine fields, sizes and prices, credit, limits, error codes | https://shedix.dev/agents/reference.md |
| Something failed | https://shedix.dev/agents/troubleshooting.md |

- **"my shedix machine arnav-3f2a1b"** (prompts copied from a machine's page say this): use that machine as `$NAME`
  instead of finding or creating one. The same prompts end with "(key: shedix_…)": that's a pasted key (below).
- **Older, longer prompts** (with `Token: shedix_…` and a guide file named in them) still work: the token is a pasted
  key, and the file they name is the job. Do this guide's first step before that file's steps.

For anything else, `https://shedix.dev/docs/api` has every call with every field (the same calls the shedix page makes).

## The key

Every call needs the person's key, which starts with `shedix_` (older ones: `shed_`). It can do anything the person
can do with their machines. **Use the first of these you find:**

1. **A key the person pasted** in the chat or the prompt (`key: shedix_…`, or `Token: shedix_…` in older prompts). It's
   for this prompt only: use it, and don't save it.
2. **`SHEDIX_TOKEN`** in your environment.
3. **The key saved on this computer** in `~/.config/shedix/config.json` (`{"url", "token"}`, written by the shedix
   CLI's `shedix login` and by the link code below). Read only its token, with the helper from "Set up once".
4. **None:** get one with a **link code**. The person approves it on shedix while signed in, and you receive a key.

Check the key you found with one call (after "Set up once"). `200` means it works:

```bash
export SHEDIX_URL=https://shedix.dev SHEDIX_TOKEN=${SHEDIX_TOKEN:-$(python3 ~/.config/shedix/key.py token)}
curl -sS -o /dev/null -w "%{http_code}\n" "$SHEDIX_URL/api/machines" -H "Authorization: Bearer $SHEDIX_TOKEN"
```

- **`401` from the saved key:** it was revoked. Remove it (`python3 ~/.config/shedix/key.py forget`) and get a new one
  with a link code. **`401` from a pasted or environment key:** tell the person it no longer works, and offer a link
  code.
- **`403`:** the account isn't invited yet, or is suspended: tell the person.

**The link code.** Ask for one, sending your own name ("Claude Code", "Codex", "Cursor"; this computer's short name
goes with it), and tell the person the line it prints, word for word:

```bash
python3 ~/.config/shedix/key.py link "Claude Code"
```

> Open https://shedix.dev/link/BCDF-2345 and approve (code BCDF-2345)

Then wait for the approval. Each run waits up to 100 seconds, polling as often as shedix allows, which fits Claude
Code's 2-minute limit on a command (if your commands stop sooner, give it fewer: `wait 25`). Run it again while it
says it's still waiting; the code lasts 10 minutes:

```bash
python3 ~/.config/shedix/key.py wait 100
```

- **`Linked to shedix as a***@gmail.com`:** the key is saved for next time. Tell the person, once: "Linked to shedix
  as a***@gmail.com. If that isn't you, say so and I'll remove the key. It's saved in ~/.config/shedix/config.json,
  readable only by your user (programs you run can read it too); you can revoke it at https://shedix.dev/profile#access."
  If they say it isn't them, remove it (`python3 ~/.config/shedix/key.py forget`) and stop.
- **Still waiting** (exit code 2): run it again. **Denied, expired or used** (exit code 1): tell the person, and ask
  before starting a new code.
- A new person signs up on that page first; you never handle their shedix sign-in.

**Keep the key secret.** It lives only in `~/.config/shedix/config.json` (or the person's environment, or this
prompt). Never write it into any other file, a commit, a log or output you show, and never type it into a program's
prompt. Never ask for the person's shedix sign-in or password.

## Set up once

shedix's SSH key, and two small helpers: `show.py` prints shedix's answers without their secrets (a machine's JSON
carries `access.password` and secret addresses), `key.py` reads and saves the key and runs the link code, and both only
ever print what's safe. Agents can't type passphrases and the person's own SSH keys aren't yours to change, so use the
key kept for shedix (the shedix CLI uses the same one). Running this again does no harm:

```bash
mkdir -p ~/.config/shedix/ssh && chmod 700 ~/.config/shedix
[ -f ~/.config/shedix/id_ed25519 ] || ssh-keygen -q -t ed25519 -N "" -C shedix -f ~/.config/shedix/id_ed25519
cat > ~/.config/shedix/show.py <<'EOF'
# Prints shedix's answers without their secrets. "login": only user@IP of a ready machine, failing otherwise.
# "signed claude [codex]": the agents' sign-in; exit 0 all signed in, 1 one isn't, 2 the machine doesn't say. 3: an error.
import json, sys
d = json.load(sys.stdin)
if "error" in d: print("error: " + d["error"], file=sys.stderr); sys.exit(3)
ms = d.get("machines", [d])
if sys.argv[1:] == ["login"]:
    if not ms[0].get("ready"): sys.exit(f'{ms[0].get("name")} is not ready: {ms[0].get("status")}, {ms[0].get("setup")}')
    a = ms[0]["access"]; print(f'{a.get("user") or "dev"}@{ms[0]["ip"]}'); sys.exit()
if sys.argv[1:2] == ["signed"]:
    ag = ms[0].get("agents")
    if not ag: print(f'{ms[0].get("name")} does not report sign-in: check over ssh'); sys.exit(2)
    print("  ".join(f"{k}={ag.get(k)}" for k in sys.argv[2:]))
    sys.exit(1 if any(ag.get(k) in (None, "off") for k in sys.argv[2:]) else 0)
for m in ms:
    print("  ".join(f"{k}={m[k]}" for k in "name status setup ip size stack project label sleepTimer".split() if m.get(k) not in (None, "")) + ("  key added" if m.get("added") else ""))
if not ms: print("no machines")
if "flushed" in d: print("flushed:", d["flushed"])
EOF
cat > ~/.config/shedix/key.py <<'EOF'
# shedix's key on this computer, kept only in ~/.config/shedix/config.json ({"url", "token"}, as `shedix login` writes it).
# get: says whether one is saved. token: prints it, for SHEDIX_TOKEN=$(...) only. forget: removes it.
# link [agent]: asks shedix for a link code and prints what to tell the person.
# wait [seconds]: waits for the approval, then saves the key; exit 0 saved, 2 still waiting, 1 failed.
import json, os, socket, sys, time, urllib.error, urllib.request
D = os.path.expanduser("~/.config/shedix"); CFG = D + "/config.json"; LINK = D + "/link.json"
URL = os.environ.get("SHEDIX_URL") or "https://shedix.dev"
def read(f):
    try:
        with open(f) as h: return json.load(h)
    except (OSError, ValueError): return {}
def write(f, data):
    os.makedirs(D, exist_ok=True); os.chmod(D, 0o700)
    fd = os.open(f + ".new", os.O_WRONLY | os.O_CREAT | os.O_TRUNC, 0o600); os.fchmod(fd, 0o600)
    with os.fdopen(fd, "w") as h: json.dump(data, h, indent=2); h.write("\n")
    os.replace(f + ".new", f)
def drop(f):
    try: os.remove(f)
    except OSError: pass
def post(path, body):
    # The API refuses Python's default User-Agent (error 1010), so name this helper
    req = urllib.request.Request(URL + path, json.dumps(body).encode(), {"Content-Type": "application/json", "User-Agent": "shedix-agent/1 (key.py)"})
    try:
        with urllib.request.urlopen(req, timeout=30) as r: return r.status, json.load(r)
    except urllib.error.HTTPError as e:
        try: return e.code, json.load(e)
        except ValueError: return e.code, {}
    except (OSError, ValueError) as e: return 0, {"error": str(e)}
cmd, arg = (sys.argv[1:] + ["get"])[0], (sys.argv[2:] + [""])[0]
token = read(CFG).get("token")
if cmd == "get": print("a key is saved" if token else "no saved key"); sys.exit(0 if token else 1)
if cmd == "token":
    if not token: sys.exit("no saved key")
    print(token)
elif cmd == "forget":
    c = read(CFG)
    if c.pop("token", None): write(CFG, c)
    print("the saved key is removed")
elif cmd == "link":
    l = read(LINK)
    if l.get("expiresAt", 0) < time.time() + 30:
        s, l = post("/api/link", {"agent": (arg or "a coding agent")[:60], "computer": socket.gethostname().split(".")[0][:60]})
        if s != 201: sys.exit(f'shedix answered {s}: {l.get("error", "no link code")}')
        l["expiresAt"] = time.time() + l.get("expiresIn", 600); write(LINK, l)
    print(f'Open {l["url"]} and approve (code {l["code"]})')
elif cmd == "wait":
    l = read(LINK)
    if not l.get("poll"): sys.exit("no link code waiting: run link first")
    end, gap = time.time() + float(arg or 100), l.get("interval", 5)
    while True:
        s, r = post("/api/link/poll", {"poll": l["poll"]})
        st = r.get("status") if s == 200 else None
        if st == "approved":
            c = read(CFG); c["token"] = r["token"]; c.setdefault("url", URL); write(CFG, c); drop(LINK)
            print(f'Linked to shedix as {r.get("email")}. The key is saved in {CFG}.'); sys.exit()
        if st in ("denied", "expired", "used") or s == 404:
            drop(LINK); sys.exit({"denied": "The person denied it.", "expired": "The code expired.", "used": "The code was already used."}.get(st, "shedix no longer knows this code."))
        if s == 429: gap = r.get("interval") or gap + 5
        elif st != "pending": print(f'shedix answered {s}: {r.get("error", "")} (trying again)')
        if time.time() + gap > end: print("Still waiting for the person to approve: run wait again."); sys.exit(2)
        time.sleep(gap)
else: sys.exit("use: key.py get | token | forget | link [agent] | wait [seconds]")
EOF
```

Each command you run may start a fresh shell, so begin every command that calls shedix with this line. With a pasted
key, write it in place of the `${…}` part (`SHEDIX_TOKEN=shedix_…`):

```bash
export SHEDIX_URL=https://shedix.dev SHEDIX_TOKEN=${SHEDIX_TOKEN:-$(python3 ~/.config/shedix/key.py token)}
```

## The first step: every job, before moving anything

Three checks, in this order: (a) the machine is ready and you're connected; (b) the agent that will work there is
signed in, when the job runs Claude Code or Codex there; (c) the person has said which private files may go, when the
job copies files from this computer. Nothing is copied before all three are done.

### (a) The machine is ready

Name the project so you (or the next agent) can find its machine again: the repository's name, `owner/repo`, or the
folder's name when there's no remote (up to 80 letters, digits, dots, dashes, underscores or slashes; stored
lower-case). The person sees each project as a group on the shedix page. Skip this when the prompt names the machine.

**Find this project's machine, or create one, in one command, without waiting.** The printer shows one line per
machine, or "no machines":

```bash
FOUND=$(curl -sS "$SHEDIX_URL/api/machines?project=acme/web" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 ~/.config/shedix/show.py)
echo "$FOUND"
if [ "$FOUND" = "no machines" ]; then curl -sS -X POST "$SHEDIX_URL/api/machines" \
  -H "Authorization: Bearer $SHEDIX_TOKEN" -H "Content-Type: application/json" \
  -d "{\"size\": \"m\", \"stack\": \"web\", \"project\": \"acme/web\", \"label\": \"api and web\", \"sshKey\": \"$(cat ~/.config/shedix/id_ed25519.pub)\"}" \
  | python3 ~/.config/shedix/show.py; fi
```

- **Size** by the heaviest step: `s` (2 vCPU, one shared core, 4 GB) for scripts and small projects, `m` (4 vCPU,
  8 GB) for installing and running an app, `l` or `xl` for heavy builds or several services. A machine's size can't
  change later. Prices: https://shedix.dev/agents/reference.md.
- **Stack:** `web` (Node, pnpm, Yarn, Bun), `python` (uv) or `systems` (Go, Rust). Every stack has Docker, Node,
  Python, Claude Code and Codex. **Label:** a name the person will recognize, up to 60 characters.
- **A `402`** means there isn't enough credit, **a `409`** a limit (3 machines running, 10 in all): tell the person the
  error's words.

An existing machine that's `stopped` needs a start (its files and installed dependencies are still there); one that's
`stopping` needs to finish first. On a machine you didn't just create, add shedix's key (harmless if it's there):

```bash
curl -sS -X POST "$SHEDIX_URL/api/machines/$NAME/start" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 ~/.config/shedix/show.py   # same name, new IP
curl -sS -X POST "$SHEDIX_URL/api/machines/$NAME/ssh-keys" -H "Authorization: Bearer $SHEDIX_TOKEN" \
  -H "Content-Type: application/json" -d "{\"key\": \"$(cat ~/.config/shedix/id_ed25519.pub)\"}" | python3 ~/.config/shedix/show.py
```

A new machine is ready in about a minute and a half: read the project meanwhile. Then wait for it (`?wait=N`, up to
300, answers as soon as it's ready) and write its SSH settings, one file per machine, which every later `ssh`,
`scp` and `rsync` uses. It shares one connection for 15 minutes, so only the first command pays for connecting. The
IP changes every start, so write the file again after one:

```bash
NAME=arnav-3f2a1b
LOGIN=$(curl -sS "$SHEDIX_URL/api/machines/$NAME?wait=240" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 ~/.config/shedix/show.py login) &&
cat > ~/.config/shedix/ssh/$NAME.conf <<EOF
Host $NAME
  HostName ${LOGIN#*@}
  User ${LOGIN%@*}
  IdentityFile ~/.config/shedix/id_ed25519
  IdentitiesOnly yes
  HostKeyAlias $NAME
  UserKnownHostsFile ~/.config/shedix/known_hosts
  StrictHostKeyChecking accept-new
  ControlMaster auto
  ControlPath ~/.config/shedix/cm-%C
  ControlPersist 15m
  ServerAliveInterval 30
  LogLevel ERROR
EOF
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'echo connected; nproc; free -g | head -2'
```

From here: `ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME '<command>'`. You log in as the machine's user, with
passwordless `sudo` and `docker`. Work in `~/work` (the page's Files panel shows it). Inside quotes, write `$HOME`, not
`~`. The person can use the same `ssh` line on this computer; it changes nothing on their own keys.

### (b) The agent there is signed in

Jobs that run Claude Code or Codex on the machine (handoffs, fork, split tasks) need it signed in there first. Read
`agents` from the machine (`agents.claude` for a Claude Code job, `agents.codex` for a Codex job): `off` is not
signed in; `ready`, `allow` and `on` are signed in.

```bash
curl -sS "$SHEDIX_URL/api/machines/$NAME" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 ~/.config/shedix/show.py signed claude   # or: signed claude codex
```

**Not signed in: stop, and give the person this one link** (`codex` in place of `claude` for Codex). It opens the
machine's page straight into the agent's own sign-in, in the browser:

> Claude Code isn't signed in on arnav-3f2a1b yet. Open https://shedix.dev/m/arnav-3f2a1b/claude#signin and sign in (it's
> Claude Code's own sign-in; shedix never sees it). It then offers to turn on Remote Control for the Claude app, which
> is up to you. I'll carry on by myself as soon as it shows you're signed in.

Then wait until the machine says so, in rounds that fit your command time limit, and carry on by yourself:

```bash
end=$((SECONDS+100)); while :; do
  curl -sS "$SHEDIX_URL/api/machines/$NAME" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 ~/.config/shedix/show.py signed claude; s=$?
  [ $s = 1 ] && [ $SECONDS -lt $end ] || break; sleep 5
done
```

Exit code 0: signed in, go on. 1 after the round: run it again (the person may still be signing in). 3: shedix
answered with an error, shown above. 2: an older machine that doesn't report it; check with `ssh … 'claude auth status'` (Codex: `codex login status`) instead.
Never handle the person's Claude or Codex sign-in yourself (don't start it in a terminal or pass on its code), and
never ask them to SSH in for it: the link does it.

### (c) The private files: ask before copying

Before copying the project, find the files that won't travel with git and are private: `.env` files, `.dev.vars`,
`.npmrc`, keys and certificates, an untracked or ignored `CLAUDE.md`, `AGENTS.md` or `.claude/`, and the person's own
`~/.claude/CLAUDE.md` (for Codex, `~/.codex/AGENTS.md`). This lists them into a file the copy step reads, without
showing what's in them:

```bash
L=~/.config/shedix/leave-out-$NAME.txt
find . \( -name node_modules -o -name .venv -o -name venv -o -name __pycache__ -o -name target -o -name dist -o -name build \
  -o -name .next -o -name .turbo -o -name .cache -o -name coverage -o -name .git \) -prune -o \( -name '.env*' -o -name '.dev.vars*' \
  -o -name .npmrc -o -name .pypirc -o -name .netrc -o -name '*.pem' -o -name '*.key' -o -name '*.p12' -o -name '*.pfx' -o -name '*.crt' \
  -o -name 'id_rsa*' -o -name 'id_ed25519*' -o -name '*credentials*.json' -o -name 'service-account*.json' -o -name 'secret*' \
  -o -name CLAUDE.md -o -name CLAUDE.local.md -o -name AGENTS.md -o -path '*/.claude/*' \) -print | sed 's|^\./||' | sort \
  | while read -r f; do git ls-files --error-unmatch "$f" >/dev/null 2>&1 || echo "/$f"; done > "$L"
cat "$L"; [ -f ~/.claude/CLAUDE.md ] && echo "~/.claude/CLAUDE.md"
```

If the list is empty, say so and go on. Otherwise ask the person **one question**, listing the files, with your own way
of asking (Claude Code's AskUserQuestion, Codex's equivalent, or plainly in the chat): **copy all of them, choose
which, or none.** If you're already waiting for the person (the sign-in in (b)), ask it in the same message. Nothing
private is copied without a yes. Then make the file hold only what stays behind (one `/path` per line, as listed):

```bash
: > ~/.config/shedix/leave-out-$NAME.txt                                    # all may go
printf '%s\n' /.env /api/.env.local > ~/.config/shedix/leave-out-$NAME.txt  # these stay behind; leave it as it is for none
```

The person's own `~/.claude/CLAUDE.md` goes only if they said yes, after the project, to the same place on the machine
(`scp -F ~/.config/shedix/ssh/$NAME.conf ~/.claude/CLAUDE.md ${NAME}:.claude/CLAUDE.md`; for Codex, append theirs to
the machine's `~/.codex/AGENTS.md`, which holds shedix's own notes). After copying, tell the person exactly which
private files were sent, and which stayed behind.

## Rules that save the most time

- **Reuse the project's machine** when there is one: its dependencies are already installed.
- **One SSH connection, and plain checks:** a file count, a port, an HTTP status. Never hide an error behind
  `| grep` or `; echo done`.
- **zsh (macOS):** put braces round a variable that a colon follows (`${NAME}:work/`, `${PORT}:localhost`): zsh can
  read `:w` or `:l` as a modifier. A glob that matches nothing (`README*`) aborts the whole command.
- **Ask first** before you share a public link to the person's app, and never sleep, stop or delete a machine they're
  using without being asked.
- **Tell the person** the machine's name and where to find what you started (the job files say where), in your last
  message.

## Notes for your agent

- **Claude Code:** a command stops after 2 minutes unless you give it a longer timeout (up to 10) or run it in the
  background. If one timed out, check the machine (`tmux ls`, the logs) before running it again: the first run may
  still be going.
- **Codex:** its sandbox blocks the network unless the person allows it, so ask once, at the start, for network access
  to shedix and the machine for the whole task. Permissions last one turn, so finish the task in that turn rather than
  stopping to report on the way. If the person's message ends your turn, carry on from where you were.
- **Cursor and others:** if your commands have a time limit, run anything long in tmux on the machine (run-project.md
  shows how) and check on it with short commands.
