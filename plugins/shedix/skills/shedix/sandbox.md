# Do the work on a machine instead of here

For "Use shedix for this" when the work isn't running the project (that's https://shedix.dev/agents/run-project.md): trying
an install script, a tool or a risky command on a clean machine, an experiment that needs root or Docker, or carrying
on with the person's next requests on a machine instead of this computer.

**Done when:** the work is done on the machine, you've shown the person what you did and found, and the machine is
deleted, asleep or kept, as they want. Minutes, plus the work.

**Before this:** the key, "Set up once" and the first step in https://shedix.dev/agents.md: (a) the machine ready and
connected as `$NAME`; (c) only if you'll copy files from here; (b) only if an agent will work on the machine.

1. **Which machine.** For an experiment that has nothing to do with the project, create a fresh one with a label
   saying what it's for, size `s` unless the work is heavy, and no project (skip the lookup in the first step). For
   work on the project, use the project's machine. If the person named a machine ("my shedix machine …"), use it.
2. **Do the work there, not here.** Run every command over SSH (`ssh -F ~/.config/shedix/ssh/$NAME.conf $NAME '…'`),
   in `~/work`. It has passwordless `sudo` and Docker. Run anything long in tmux with a log (run-project.md, "Run
   commands"). Copy only what the work needs from this computer, after the first step's private-files question.
3. **Show the person** what you did and what you found: the commands, what they changed, the output that matters.
4. **Afterwards:** delete a machine you created for a one-off experiment, unless the person wants to keep it (ask);
   put the project's machine to sleep rather than deleting it (https://shedix.dev/agents/machine-lifecycle.md). Tell the
   person the machine's name and what you did with it.
