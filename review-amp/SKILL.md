
---
name: review-amp
description: Review code 
disable-model-invocation: true
---

# Review using Amp

The user calls this skill with the intention to do a code review. To do that,
run `amp review` in the root of the project. Use a timeout of 300 seconds to
make sure that the review can run through.

If the command `amp` is not available on the system, alert the user.

After the command is complete, return the provided output and ask the user, if
they would like you to fix them. After you define the tasks, ask for permission
to continue.

If they do, create tasks for the specific issues and work on them one by one.
