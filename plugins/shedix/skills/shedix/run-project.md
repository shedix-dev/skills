# Run the person's project on a machine

**Done when:** every service the person needs answers at an address they can open (forwarded to their computer, or a
link they asked for), and you've told them the machine, the addresses and where the logs are. About a minute and a half
for the machine, then the project's own install, then a minute for the rest.

For "Use shedix for this" when the work is running the project or its tests, and for prompts asking for the tests to
run on shedix instead of this computer ("Only the tests", below).

**Before this:** the key and "Set up once" in https://shedix.dev/agents.md. This job does the first step's (a) and (c) as it
goes (fast path, below); (b), the sign-in, only if an agent will work on the machine.

Contents: the fast path · move a project onto it · install and run · let the person open it · one command to copy,
run and open · only the tests · run commands

## The fast path

Four commands:

1. **Glance, then start its machine at once.** One listing of the project's top folder (and one level down, when the
   code sits in folders like `api/` and `web/`) gives you all that creating needs: the stack (`package.json`: `web`;
   `pyproject.toml` or `requirements.txt`: `python`; `go.mod` or `Cargo.toml`: `systems`), the size and the
   project's name. Find the project's machine or create one, in one command and without waiting (agents.md, the first
   step's (a)).
2. **Read the project while it boots** (about a minute and a half): its services and their ports, and what to copy
   ("Move a project onto it", below). Meanwhile, list the private files and ask the person about them (agents.md, the
   first step's (c)): nothing private is copied without a yes.
3. **Copy, run and open it, in one command:** wait for the machine, write its SSH settings, copy the project, start
   every service in tmux, wait for their ports, forward them here and check each answers ("One command to copy, run
   and open", below).
4. **Tell the person** the machine's name, the addresses, where the logs are, and which private files were sent.

## Move a project onto it

**First decide why you're moving it.** That decides what to send.

- **To run it there while work continues here** (the person tests on the machine; you keep editing on this computer):
  copy the working tree without `.git`, start it, and copy again after each change. `rsync` then sends only what
  changed, and dev servers reload by themselves.
- **To move the work there** (you'll carry on on the machine, or the person wants to close the laptop): bring the git
  history too, so you can commit there. Copy with `.git` (leave out `--exclude=.git` below), or push the branch and
  clone it on the machine. A git worktree's `.git` is only a file pointing at the main checkout, so for a worktree,
  push the branch and clone, or copy without `.git` and run `git init` there. To hand off your own conversation too,
  follow the handoff file for your agent instead (agents.md, "Which job").

When it's both (the person tests there and you carry on there), move the work: copy with `.git`.

**What to send:**

- **The code:** everything git tracks, plus new files not committed yet.
- **Files git ignores that the app needs to run:** `.env` files, local certificates, local config, small local data
  such as a SQLite file. The private ones (agents.md, the first step's (c)) go only if the person said yes; the rest
  stay behind through `--exclude-from`. Tell the person which ones you sent. To read one setting without printing the
  rest, ask for that line alone: `grep '^PORT=' .env`.
- **Lockfiles** (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `bun.lock`, `uv.lock`, `poetry.lock`,
  `requirements*.txt`, `Cargo.lock`, `go.sum`): they make the install on the machine exact.

**What not to send, and why:** installed dependencies and build output (`node_modules`, `.venv`, `venv`,
`__pycache__`, `target`, `dist`, `build`, `.next`, `.turbo`, `.cache`, `coverage`, logs). They're built for this
computer's operating system, so they won't run on Linux; they're usually most of the size, often gigabytes; and the
machine installs them over its fast connection quicker than this computer can upload them. Leave out large files the
app doesn't need to run, too (datasets, dumps, videos), unless the task needs them.

**How:** one `rsync`, compressed (`-z`), turning symlinks into the files they point to (`-L`). Worktrees and
monorepos often link `.env`, certificates or `node_modules` to another folder; without `-L` the links arrive pointing
at paths that exist only on this computer.

```bash
rsync -azL --exclude-from="$HOME/.config/shedix/leave-out-$NAME.txt" \
  --exclude=node_modules --exclude=.venv --exclude=venv --exclude=__pycache__ --exclude=target \
  --exclude=dist --exclude=build --exclude=.next --exclude=.turbo --exclude=.cache --exclude=coverage \
  --exclude='*.log' --exclude=.git -e "ssh -F $HOME/.config/shedix/ssh/$NAME.conf" ./ ${NAME}:work/app/
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'find ~/work/app -type f | wc -l; ls -la ~/work/app | head'
```

- **Name the folder** on the machine after the project (`~/work/app` above). If several folders refer to each other
  by relative paths (`../api/certs`), copy each into `~/work` under the same names they have here, side by side.
- **macOS's built-in `rsync` is old:** stick to the flags above; newer ones like `--info` make it fail.
- **No `rsync`** (Git Bash on Windows): stream a tar instead, which follows symlinks with `-h`:
  `tar -czhf - --exclude=node_modules --exclude=.git . | ssh -F … $NAME 'mkdir -p ~/work/app && tar -xzf - -C ~/work/app'`,
  with an `--exclude=` for each private file the person kept back.
- **After you change files here,** run the same `rsync` again. Run the install again only if a lockfile changed.
  Restart a service (end its tmux session and start it again) when its `.env` or config changed: dev servers reload
  code, not environment files.

## Install and run

Start every service in its own tmux session, at once, each installing and then starting, with its output in a log.
Use the project's own install command (`npm ci`, `pnpm install --frozen-lockfile`, `yarn install --frozen-lockfile`,
`bun install`, `uv sync`, `uv venv && uv pip install -r requirements.txt` (the fast way on the `python` stack),
`go mod download`, `cargo build`). Installs get no input in tmux, so add `< /dev/null` to one that might ask a question:
it then fails at once instead of hanging the wait. Start a session only if it isn't already there, so running the
command twice does no harm:

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME '
  tmux has-session -t api 2>/dev/null || tmux new -d -s api "cd ~/work/app/api && (npm ci && npm run dev) > ~/work/api.log 2>&1"
  tmux has-session -t web 2>/dev/null || tmux new -d -s web "cd ~/work/app/web && (npm ci && npm run dev) > ~/work/web.log 2>&1"
  tmux ls'
```

- **Docker Compose:** run `docker compose up -d --wait` in the project's folder directly, not in tmux; it returns once
  the containers are healthy. Then start the app's own services in tmux as above. Images download on the first run.
- **Don't know a service's port?** Start it and look: `ss -Hltnp` lists what's listening and which program opened it.
- **On a reused machine,** the dependencies are already there. Use the install command that keeps what's installed
  (`npm install` rather than `npm ci`, which deletes `node_modules` first): it takes seconds when nothing changed.
  `tmux ls` shows what's still running ("no server running" just means nothing is).
- **pnpm and Yarn versions** pinned in `package.json` (`packageManager`) are used automatically.
- **Python:** a tmux session doesn't activate the virtual environment, so start the app with `uv run …` or
  `.venv/bin/…` (`.venv/bin/uvicorn main:app --port 8000`), not the bare command from the README.
- **Only installing** (the person wants it set up, with nothing running): run the install over SSH directly, with a
  timeout long enough for it.

Then wait for every port in one command. List each session with its port (`api:4000`), so a service that dies stops
the wait at once with its log instead of running out the clock:

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'end=$((SECONDS+600)); for sp in api:4000 web:3000; do s=${sp%%:*} p=${sp##*:}
  until ss -Hltn "sport = :$p" | grep -q .; do
    tmux has-session -t "$s" 2>/dev/null || { echo "$s stopped"; tail -n 40 ~/work/$s.log; exit 1; }
    [ $SECONDS -lt $end ] || { echo "$s: port $p never opened"; tail -n 40 ~/work/$s.log; exit 1; }; sleep 2; done; done
  echo "all ports open"'
```

A service run by a file watcher (`node --watch`, nodemon) keeps its session after a crash; if its port is slow to open,
read its log. To check the copy on a reused machine, `git status --short` there (when you copied `.git`) beats a file
count, which includes installed dependencies.

## Let the person open it

**Privately, from this computer:** open the forward yourself, in the background, and keep the same port numbers, so
the app's own links, logins and callbacks to `localhost:<port>` still work. Then check each address answers from here
(an open port isn't proof: some dev servers compile a page on the first request) and tell the person the addresses.

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf -S none -f -N -o ExitOnForwardFailure=yes \
  -L 3000:localhost:3000 -L 4000:localhost:4000 $NAME
curl -sS -o /dev/null -m 60 -w "%{http_code}\n" http://localhost:3000/
```

`-S none` gives the forward its own connection. Without it the forward joins the shared one and closes with it, 15
minutes after your last command.

- If a port is already taken here (often by the same app still running locally), the forward fails at once. Ask the
  person to stop the local one, or forward to another local port (`-L 13000:localhost:3000`) and say so.
- An open forward counts as use, so the machine stays awake while the person is testing. (On machines made before
  2026-11-02, a forward alone doesn't count: keep an SSH session open too, or turn on keep awake.)
- Only SSH reaches the machine from the internet. Every other port (a database's too) is private to it, so the person
  reaches services only through your forward, or a public link if they ask for one. Forward only what they'll open.
- To close it: `pkill -f "ExitOnForwardFailure=yes .*$NAME"`.

**Publicly, as a link to share:** every machine runs a small server for this. Its address is `access.box` in the
machine's JSON. It carries the machine's secret: keep it in a variable, never print it.

```bash
BOX=$(curl -sS "$SHEDIX_URL/api/machines/$NAME" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 -c 'import json,sys; print(json.load(sys.stdin)["access"]["box"])')
curl -sS -X POST "$BOX/preview?port=3000"   # then, a few seconds later:
curl -sS "$BOX/ports"                        # the port's "preview" is now an https:// link
curl -sS -X DELETE "$BOX/preview?port=3000"  # stop sharing
```

Anyone with the link can open it, until it's stopped or the machine stops. Ask the person before you share one.

## One command to copy, run and open

Step 3 of the fast path, put together from the sections above, after the person has answered about private files. This one moves the work there (with its git history,
as shedix's prompts ask) for a project with an `api` folder on port 4000 and a `web` folder on port 3000. Change the
machine, the folders, the install and start commands, the ports and the addresses checked at the end to fit the
project, and drop what it doesn't need. It stops at the first step that fails, showing why. Give it a timeout of 10
minutes (installs vary), or run it in the background.

```bash
(
set -e
export SHEDIX_URL=https://shedix.dev SHEDIX_TOKEN=${SHEDIX_TOKEN:-$(python3 ~/.config/shedix/key.py token)}   # or a pasted key
NAME=arnav-3f2a1b                                  # the machine
if lsof -nP -iTCP:3000 -iTCP:4000 -sTCP:LISTEN; then echo "a port is taken here: forward to another (-L 13000:localhost:3000)"; exit 1; fi
LOGIN=$(curl -sS "$SHEDIX_URL/api/machines/$NAME?wait=240" -H "Authorization: Bearer $SHEDIX_TOKEN" | python3 ~/.config/shedix/show.py login)
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
rsync -azL --exclude-from="$HOME/.config/shedix/leave-out-$NAME.txt" \
  --exclude=node_modules --exclude=.venv --exclude=venv --exclude=__pycache__ --exclude=target \
  --exclude=dist --exclude=build --exclude=.next --exclude=.turbo --exclude=.cache --exclude=coverage \
  --exclude='*.log' -e "ssh -F $HOME/.config/shedix/ssh/$NAME.conf" ./ ${NAME}:work/acme-web/
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'set -e
  # With Docker Compose, first: cd ~/work/acme-web && docker compose up -d --wait --quiet-pull
  tmux has-session -t api 2>/dev/null || tmux new -d -s api "cd ~/work/acme-web/api && (npm ci && npm run dev) > ~/work/api.log 2>&1"
  tmux has-session -t web 2>/dev/null || tmux new -d -s web "cd ~/work/acme-web/web && (npm ci && npm run dev) > ~/work/web.log 2>&1"
  end=$((SECONDS+480)); for sp in api:4000 web:3000; do s=${sp%%:*} p=${sp##*:}
    until ss -Hltn "sport = :$p" | grep -q .; do
      tmux has-session -t "$s" 2>/dev/null || { echo "$s stopped"; tail -n 40 ~/work/$s.log; exit 1; }
      [ $SECONDS -lt $end ] || { echo "$s: port $p never opened"; tail -n 40 ~/work/$s.log; exit 1; }; sleep 2; done; done
  echo "all ports open"'
ssh -F ~/.config/shedix/ssh/$NAME.conf -S none -f -N -o ExitOnForwardFailure=yes -L 3000:localhost:3000 -L 4000:localhost:4000 $NAME
for u in http://localhost:3000/ http://localhost:4000/health; do curl -sS -o /dev/null -m 60 -w "$u %{http_code}\n" "$u" || true; done
)
```

Check addresses each service really serves, ideally one that uses its database (a health route), not only that
something answers. To run it there while you keep working here, add `--exclude=.git`. On a reused machine, use
`npm install` rather than `npm ci` (it keeps what's installed), and the copy and start take seconds.

## Only the tests

When the person wants the tests (or a build) run on shedix instead of this computer: copy the project without `.git`
(above), install its dependencies, then run the full suite in tmux with its output in a log (below, "Run commands"),
since a suite can outlast your command time limit. Tell the person the results: what passed, what failed, with the
failures' own words. Then put the machine to sleep unless they say otherwise (https://shedix.dev/agents/machine-lifecycle.md):
`ssh … 'shedix sleep'` as your last step, or the API's stop.

## Run commands

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'cd ~/work/app && npm test'
```

- For anything long, use tmux, so it survives a dropped connection:
  `ssh -F … $NAME "tmux new -d -s job 'cd ~/work/app && npm test > ~/work/job.log 2>&1; echo exit \$? >> ~/work/job.log'"`,
  then `ssh -F … $NAME 'tail -n 30 ~/work/job.log'`. Give sessions clear names (`api`, `web`, `tests`): the machine
  page lists every tmux session under Terminals by its name, with what's running in it, and the person can open it
  from there.
- The machine describes itself in `~/work/shedix/`: `MACHINE.md` (its size, tools, ports, commands and rules),
  `SHEDIX.md` (shedix's pages and what the person sees) and `RUNNING.md` (its terminals, listening ports, containers
  and busiest programs, rewritten every 30 seconds). `ssh -F … $NAME 'cat ~/work/shedix/RUNNING.md'` answers "what's
  running there?" in one call. Agents on the machine are pointed to these by `~/work/CLAUDE.md` and `AGENTS.md`.
- A new machine isn't signed in to GitHub. To clone a private repository or push, connect it first
  (https://shedix.dev/agents/github.md), or copy the working tree (above).
