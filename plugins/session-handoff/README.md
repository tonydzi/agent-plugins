# session-handoff

A session that inherits half-finished work spends its first turns rediscovering what was decided, then quietly redoes or undoes it. `session-handoff` makes the agent write one file that a reader with zero context can continue from.

Instructions only: one markdown skill, no scripts, no network, no dependencies.

## What the agent produces

A file `HANDOFF-<slug>.md` with eleven fixed sections:

1. Goal: current state and target state
2. Decisions: what, why, and what was rejected
3. Done and proof: each finished item with the command and result that verified it
4. Paths and values: exact files, branches, IDs, variable names
5. Open and blockers
6. Boundaries: invariants, do-not-touch, what needs the owner's approval
7. Done when: the acceptance check
8. Rollback
9. Stop conditions
10. Continue from here: exactly one first action
11. Skill gap: what in this task should already be a reusable skill

Plus a one-line seed for the receiver:

> Read `<path>/HANDOFF-<slug>.md` and continue from "Continue from here".

## What it enforces

- State is collected from the record (`git status`, `git log`, a fresh test run), not from memory. Unverifiable items are marked unverified.
- Credentials are replaced with pointers, then the file is grepped for secrets.
- A cold-read test: four questions that must be answerable from the file alone.
- The handoff is data, not authorization: the receiver keeps every approval rule and re-checks the stated state before acting.
- For spawned worker sessions: the original session stays the orchestrator; a worker handles one event per run and sets no timers of its own.

## When to use

- Before context compaction or closing a long session
- Switching machine, tool or model mid-task
- Passing a task to a teammate or to another agent

## Install

Claude Code:

```
/plugin marketplace add tonydzi/agent-plugins
/plugin install session-handoff@agent-plugins
```

Codex:

```
codex plugin marketplace add tonydzi/agent-plugins
codex plugin add session-handoff@agent-plugins
```

Any other agent (Cursor, Gemini CLI, and so on): copy `skills/session-handoff/SKILL.md` into the place your agent reads skills or rules from.

Then ask: "prepare a handoff for this session".

## License

MIT

## Credit

The contract-style sections (current state to target, invariants, do-not-touch, done-when, proof, rollback, stop conditions) grew out of the ANSS handoff standard, published at https://t.me/ClawRus/16237.
