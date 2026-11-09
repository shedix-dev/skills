# Plain machines

**Done when:** the plain machine is ready, you're connected, and what the work needs is installed. About a minute and a
half, plus your installs.

**Before this:** the key and "Set up once" in https://shedix.dev/agents.md.

`"stack": "plain"` gives you the operating system alone: Ubuntu 24.04 as Google ships it, reached over SSH as the
machine's user (`access.user`) with passwordless sudo. shedix adds one small service, `shed-activity`, which reports
when the machine was last used so it goes to sleep after an hour unused; nothing else (no web terminal, VS Code, Files,
Ports, agents, Docker, git, tmux, rsync or `~/work`). Install what the rest of this guide uses before following it,
for example `sudo apt-get install -y rsync tmux git` on Ubuntu or Debian. For another free Linux from Google, add
`"image": "project/family"`, like `"debian-cloud/debian-12"` or `"rocky-linux-cloud/rocky-linux-9"` (projects:
`ubuntu-os-cloud`, `debian-cloud`, `rocky-linux-cloud`, `almalinux-cloud`, `centos-cloud`, `opensuse-cloud`). Use it
when the person wants a clean OS; for running their project, the other stacks save you installing everything.

```bash
curl -sS -X POST "$SHEDIX_URL/api/machines" -H "Authorization: Bearer $SHEDIX_TOKEN" -H "Content-Type: application/json" \
  -d "{\"size\": \"s\", \"stack\": \"plain\", \"image\": \"debian-cloud/debian-12\", \"label\": \"clean debian\", \"sshKey\": \"$(cat ~/.config/shedix/id_ed25519.pub)\"}" \
  | python3 ~/.config/shedix/show.py
```

Then wait for it and connect as in agents.md, first step (a).
