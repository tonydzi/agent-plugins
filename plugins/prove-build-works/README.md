# prove-build-works

An acceptance gate for coding agents. It stops the agent from saying "done" after one green happy-path run.

The skill makes the agent treat "done" as a claim that needs evidence. Right after a build or a fix the agent must:

1. take the scope from `git status`, not from memory;
2. run the artifact live with the consumer's interpreter;
3. break it on purpose;
4. show the test red on a broken copy and green on the original;
5. get a review of the live code from a different vendor's model;
6. prove the result is visible (log, counter, coverage N of M);
7. repeat the check by a different method;
8. end with `PASS`, `PARTIAL` or `FAIL`, with a command and an exit code behind every row.

It also covers the opposite claim. Before the agent writes "impossible", it must go through `--help`, subcommands, config and same-version docs.

## What makes it different from "run the tests"

- **Red proof.** A test that never failed on broken code proves nothing. The skill requires the red run, a junk-flag canary and, for fail-closed gates, a mutation pass.
- **Verdict ceilings.** A skipped second opinion, a missing test or an unshown second pass caps the verdict at `PARTIAL`. The agent cannot round up.
- **Conditional checks.** A trigger table adds one cheap check per risk: join keys that are mostly empty, in-progress states with no reaper, hooks that were built and never registered, public artifacts nobody can see anonymously, exit codes eaten by a pipe.
- **Named failure modes.** Fifteen ways a check reports green while the thing is broken, each seen in real agent work.

## Install

Claude Code:

```
/plugin marketplace add tonydzi/agent-plugins
/plugin install prove-build-works@agent-plugins
```

Codex:

```
codex plugin marketplace add tonydzi/agent-plugins
codex plugin add prove-build-works@agent-plugins
```

Other agents (Cursor, Gemini CLI and any agent that reads markdown skills): copy `skills/prove-build-works/SKILL.md` into the agent's skills or rules folder.

## Use

Ask for it after any build or fix:

```
/prove-build-works
```

or in plain words: "prove that what you just built works".

Example of the closing block:

```
| What                 | How                                         | Result |
| parser fix           | ran on 3 real files, output shown           | ok     |
| red proof            | broken copy: exit 1, original: exit 0       | ok     |
| second opinion       | other-vendor CLI, 2 findings: 1 accepted    | ok     |
| scheduled entry      | not registered in the scheduler yet         | gap    |

checked: parser, test, review | not checked: run from the scheduler (entry missing)
PARTIAL: works when called by hand, the scheduler entry is not registered.
```

## What it needs

- A shell and `git`.
- For the second opinion: access to one model from a different vendor, through a CLI or a web chat. Without it the skill still runs and caps the verdict at `PARTIAL`.

Instructions only. The plugin ships markdown and runs no code.

## License

MIT
