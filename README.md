# shedix skills

Cloud Linux machines for Claude Code, Codex and other coding agents: full Ubuntu with root and Docker, keeps working
after your laptop closes, sleeps when idle. [shedix.dev](https://shedix.dev)

This repo is the **shedix skill**: it teaches your agent to start and drive your shedix machines with `curl` and
`ssh`. Ask it to "move this session to shedix so I can close my laptop", "run these three tasks on separate machines"
or "test this on a fresh Ubuntu box", and it knows what to do.

## Install

**Any agent** (Claude Code, Codex, Cursor, GitHub Copilot, Gemini CLI, OpenCode and others), with the
[skills](https://github.com/vercel-labs/skills) installer:

```bash
npx skills add shedix-dev/skills
```

Add `-g` to install it for every project, or `-a codex` (`-a cursor`, `-a claude-code`, …) to pick the agent.

**Claude Code, as a plugin:**

```
/plugin marketplace add shedix-dev/skills
/plugin install shedix@shedix
```

**Nothing to install:** paste this into any agent and it reads the same guide from the site:

```
Use shedix for this: shedix.dev/SKILL.md
```

The first time, your agent shows you a short code to approve at shedix.dev, so it never sees your password.

## What's inside

```
plugins/shedix/
  .claude-plugin/plugin.json   the Claude Code plugin manifest
  skills/shedix/SKILL.md       the skill: set-up, the key, the first step, rules
  skills/shedix/*.md           one file per job: hand off a session, fork, run a project, copy machines, GitHub,
                               the browser, plain machines, sandboxing, the API reference, troubleshooting
.claude-plugin/marketplace.json  makes this repo a Claude Code marketplace
```

The files are copied from what [shedix.dev/SKILL.md](https://shedix.dev/SKILL.md) serves, so the repo and the site
always match.

## Links

- [shedix.dev](https://shedix.dev): start a machine
- [Install for your agent](https://shedix.dev/skills)
- [The guide for agents](https://shedix.dev/docs/agents) and the [API](https://shedix.dev/docs/api)
- Questions: [hello@shedix.dev](mailto:hello@shedix.dev) · Security: [security@shedix.dev](mailto:security@shedix.dev)

The skill is free to use for working with shedix (see the `license` line in `SKILL.md`).
