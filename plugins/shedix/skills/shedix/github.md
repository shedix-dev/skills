# Connect GitHub, so the machine can clone and push

**Done when:** `gh api user` on the machine answers with the person's GitHub account, and git has an identity there.
A minute, most of it the person approving on github.com.

**Before this:** the key, "Set up once" and the first step's (a) in https://shedix.dev/agents.md: the machine ready and
connected, as `$NAME`.

Before work on the machine needs to clone a private repository, push or open a pull request, check whether it's signed
in to GitHub. It answers with the account's name when it is:

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'gh api user --jq .login 2>/dev/null || echo "not connected"'
```

If it isn't, use one of two ways. Both are GitHub's own device sign-in: the person approves it on github.com with a
short code, the sign-in is made on the machine and stays there (shedix never sees it), and afterwards `git` uses it for
`https://github.com`. Never ask for or handle the person's GitHub password or tokens.

1. **The person connects it on the shedix page:** open the machine; under GitHub, "connect GitHub" shows a code and a
   link to github.com/login/device. Tell them that, and wait for them to say it's done.
2. **You start it and pass them the code**, when they'd rather not leave the conversation:

   ```bash
   ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'tmux has-session -t github 2>/dev/null || tmux new -d -s github "BROWSER=false gh auth login --hostname github.com --git-protocol https --web --scopes repo,workflow > ~/work/github.log 2>&1 && gh auth setup-git"; sleep 5; grep -o "one-time code: [A-Z0-9-]*" ~/work/github.log'
   ```

   Tell the person: "Open https://github.com/login/device and enter the code `XXXX-XXXX`" (it works from a phone, and
   expires after 15 minutes). Then check with the command above until it answers with the account's name.

Once connected, if the machine has no git identity yet (`git config --global user.email` prints nothing), set one so
commits show who made them: the person's own name and email if they've given them, or GitHub's private address,
`<id>+<login>@users.noreply.github.com` (`gh api user --jq '"\(.id)+\(.login)@users.noreply.github.com"'`). The shedix
page's button does this by itself. Copies of a machine stay signed in, like everything else on its disk.
