# Reference

Contents: a machine's fields · sizes and prices · stacks · credit and limits · labels and projects · errors · more

## A machine's fields

| Field | Meaning |
|---|---|
| `name` | The machine's name, like `arnav-3f2a1b` (the person's name and a random end; older machines: `shed-3f2a1b`). Use it in every later call. |
| `ready` | `true` once it's up. |
| `setup` | While it runs: `booting`, then `ready`. `copying` for a copy waiting for its snapshot. |
| `ip` | Its public IPv4 address. It changes every time the machine starts. |
| `access.user` | The user to log in as. |
| `status` | `starting`, `running`, `stopping`, `stopped` or `deleting`. |
| `size`, `stack` | Its size (`s`, `m`, …) and stack (`web`, `python`, `systems`, `plain`). |
| `image` | What it runs, for people: shedix's image, or a Plain machine's OS (like "debian-12, plain"). |
| `project` | The project you named, or null. |
| `label` | The name the person (or you) gave it, or null. |
| `access.sshKeys` | The keys that can log in. |

Other answers: a list is `{"machines": [...]}`; an error is `{"error": "..."}`. A machine's JSON also carries secrets
(`access.password`, and inside the `access.terminal` and `access.box` addresses): pipe answers through the printer
from agents.md, which leaves them out.

## Sizes and prices

Charged by the second while running, and a few cents a day while asleep (its disk, and memory above 8 GB). Choose by the heaviest step; a machine's
size can't change later (a copy can have another size: https://shedix.dev/agents/copy-machines.md).

| Size | Hardware | Running | Asleep | For |
|---|---|---|---|---|
| `s` | 2 vCPU (one shared core), 4 GB RAM, 30 GB disk | $0.09/hr | $0.006/hr | Scripts, small projects, experiments |
| `m` | 4 vCPU, 8 GB RAM, 45 GB disk | $0.21/hr | $0.009/hr | Installing and running an app: big `node_modules`, a few services, Docker |
| `l` | 8 vCPU, 16 GB RAM, 60 GB disk | $0.42/hr | $0.015/hr | Several heavy installs or services at once, parallel agents |
| `xl` | 16 vCPU, 32 GB RAM, 75 GB disk | $0.84/hr | $0.024/hr | Heavy builds |

## Stacks

`web` (Node, pnpm, Yarn, Bun, TypeScript, Playwright with Chromium), `python` (uv, JupyterLab, Ruff) and `systems`
(Go, Rust). Every stack has Docker, Node, Python, Claude Code and Codex. `plain` is the operating system alone:
https://shedix.dev/agents/plain-machines.md.

## Credit and limits

Machines are paid from the person's prepaid credit. A `402` from creating or starting means the credit is too low for
an hour of that machine, a size above `s` needs the account's first top-up, or the month's spending cap is reached:
tell the person the error's words. They add credit at `$SHEDIX_URL/profile#credit`, or with `shedix topup 20` on their
computer. A `409` is a limit (3 machines running, 10 in all): stop or delete one first. `GET $SHEDIX_URL/api/billing`
shows the balance, what the running and sleeping machines cost an hour, and when the credit runs out.

**Data transfer.** Downloading is free, always, any amount: `npm install`, `git clone`, `docker pull`, model downloads.
Only data the machine sends out counts (git push, uploads, traffic served from a preview link): 10 GB a month per
account is included, then $0.10/GB from credit, and the person is emailed at 7 and 10 GB, before it starts. Never avoid a
download to save data. Before sending many GB out in one go (over 5 GB), tell the person once.

## Labels and projects

Give a machine you create a name the person will recognize with `"label"` (up to 60 characters). To change it or the
project later: `PATCH $SHEDIX_URL/api/machines/$NAME` with `{"label": "…"}` or `{"project": "acme/web"}`.

## Errors

Errors come back as `{"error": "..."}` with:

- **400:** a bad request; the error says what.
- **401:** a bad or revoked key. A saved one: remove it and get a new one with a link code (agents.md, "The key").
- **402:** not enough credit (above). Tell the person.
- **403:** the account isn't invited yet, or is suspended.
- **404:** no such machine.
- **409:** the wrong state, like starting a machine that's running, or a limit.
- **502:** the cloud failed; try again.

## More

- The whole API, with every field: `https://shedix.dev/docs/api` (the same calls the shedix page makes).
- The person can see every call made with a key on their profile (https://shedix.dev/profile#access), and revoke it there.
- Optional, for people: a CLI with the same powers, `curl -fsSL https://shedix.dev/install.sh | sh`, then `shedix help`.
