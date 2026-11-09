# The shared browser on a machine

Every shedix machine (Web, Python, Go & Rust) has a real Chromium that the person and the agents on the machine use
together: the person watches it on the machine's page and logs in or clicks where a person must, and agents carry on in
that same browser. It's off until someone starts it, and turns itself off when nobody uses it. Logins stay on the
machine's disk while it's off.

**Done when:** the browser is running, the work in it is done, and the person had the link whenever a human step was
needed. Seconds to start.

**Before this:** the key, "Set up once" and the first step's (a) in https://shedix.dev/agents.md: the machine ready and
connected, as `$NAME`.

## Start it, check it, stop it

On the machine, `shedix browser` does all three (no key needed there):

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'shedix browser start'    # starts it, or says it's already on
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'shedix browser status'   # on or off
ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME 'shedix browser stop'
```

It won't start when the machine is short of memory, and says so: tell the person, and stop something first. Never
start a second browser beside it; use this one.

## Drive it

Agents **on the machine** (a session you started there) connect to it over the Chrome DevTools Protocol at
`http://127.0.0.1:9222`: Playwright's `chromium.connectOverCDP("http://127.0.0.1:9222")`, or a DevTools MCP server
pointed at that address. Tell such a session in its task that the browser is there.

From **this computer**, forward the port over SSH and connect to `http://127.0.0.1:9222` here (it's reachable only
through SSH, never from the internet):

```bash
ssh -F ~/.config/shedix/ssh/$NAME.conf -S none -f -N -o ExitOnForwardFailure=yes -L 9222:127.0.0.1:9222 $NAME
curl -sS http://127.0.0.1:9222/json/version | head -n 5
```

Close the forward when you're done: `pkill -f "9222:127.0.0.1:9222 .*$NAME"`.

## When a person is needed

A login, a CAPTCHA, a payment, a consent screen or anything you mustn't do yourself: stop and give the person the
browser's link on the machine's page, then carry on in the same browser once they say it's done:

> Please open https://shedix.dev/m/arnav-3f2a1b#browser, log in to the dashboard there, and tell me when you're done.

Never type the person's passwords or codes into it yourself, and don't ask for them. The view is private to the
person's shedix account, like the terminal.
