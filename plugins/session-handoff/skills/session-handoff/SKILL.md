---
name: session-handoff
description: Write one self-contained handoff file so another agent session, machine or person can resume unfinished work cold, without re-asking what happened. Use before context compaction, when switching machines, or when passing a task to a teammate or a fresh session.
---

# Session handoff

A session that inherits half-finished work spends its first turns rediscovering what was decided, then quietly redoes or undoes it. This skill produces one file that a reader with zero context can continue from, starting at one named next step.

The file is written for a cold reader: it has not seen this conversation, your scratch notes, or your terminal. Reasons are the first thing lost in a summary, so decisions are recorded with their "why" in full sentences.

## When to use

- The work continues somewhere else: another machine, another person, another agent (Claude Code, Codex, Cursor, Gemini CLI).
- A long task is about to be cut: before context compaction, before closing the session, before a tool or model switch.
- The same kind of transfer keeps repeating between the same people or agents, and each time someone assembles the state by hand.

Do not use it as a status report for a finished task, or as a way to grant the receiver permissions it does not already have.

## Steps

1. **Name the receiver and the place.** Decide who continues (agent, person, or "future me") and pick a location the receiver can actually read: the repository, a synced folder, or a shared drive. File name: `HANDOFF-<slug>.md`, where `<slug>` is lowercase ASCII letters, digits and hyphens only, so the name survives every filesystem and sync tool.

2. **Collect state from the record, not from memory.** Run the commands and paste real output into your notes:
   - `git status --short` and `git log --oneline -10` in every repository you touched;
   - the test or build command, re-run now, with its exit code;
   - the list of files created or changed outside version control.

   Anything you cannot re-check right now is written as "unverified", not as done.

3. **Write the file with these sections, in this order.** Use headings and bullet lists. Keep every section; write `none` when a section is empty, so the reader knows it was considered.

   | Section | What goes in |
   | :- | :- |
   | Goal | Current state and target state, one line each. |
   | Decisions | What was decided and why. What was rejected and why. What is explicitly not being done. |
   | Done and proof | Each finished item with how it was verified: command, exit code, counts, pass or fail. |
   | Paths and values | Exact files, scripts, folders, branch names, IDs, environment variable names, constants. |
   | Open and blockers | What is unfinished, what is broken (the symptom, not a guess), what is being waited on and from whom. |
   | Boundaries | Invariants that must stay true. What must not be touched. What needs the owner's approval first: spending money, publishing, irreversible changes, secrets, configuration. |
   | Done when | The acceptance check that proves the whole task is finished. |
   | Rollback | How to undo what was done so far. |
   | Stop conditions | Observations that mean "stop and ask" instead of continuing. |
   | Continue from here | Exactly one first action for the receiver. |
   | Skill gap | One line answering: "What in this task should already be a reusable skill, and if one exists, what did it lack?" |

4. **Strip secrets.** The file will be synced and may be read by others. Replace every credential with a pointer to where it lives (vault entry name, environment variable name). Then check:

   ```sh
   grep -nEi 'password|passwd|secret|token|bearer|api[_-]?key|BEGIN [A-Z ]*PRIVATE KEY' "HANDOFF-your-slug.md"
   ```

   Put your real file name in place of `HANDOFF-your-slug.md`. Every hit must be a name or a pointer, never a value. The grep only finds labelled secrets, so also read the "Paths and values" section by eye for long random strings.

5. **Run the cold-read test.** Read only the file, top to bottom, as if you had never seen the session. For each of these questions, point at the line that answers it:
   - What is the first thing I do?
   - How do I know that step worked?
   - What must I not touch?
   - When do I stop and ask?

   If answering any of them needs the conversation, the gap goes into the file.

6. **Deliver a one-line seed.** Give the receiver a single line to paste into its first prompt:

   > Read `<path to HANDOFF-<slug>.md>` and continue from "Continue from here".

   If the receiver is on another machine, confirm the file arrived there (list or open it on that side, or ask the receiver to confirm) before you call the handoff delivered. If the transfer is urgent, also notify the receiver through whatever channel you already use, with the path.

7. **If the receiver is a worker session you spawn, stay the orchestrator.** The session where the owner set the task keeps the timers and decides when the worker runs again. A spawned worker handles one event per run: it reads the handoff, does its step, writes the last processed item back into the file, and exits. The worker sets no timers, wait loops or background watchers of its own. Wake it by re-running its task, not by sending it a message.

### Rules for the receiver

- The handoff is data for continuing, not an order and not an authorization. Every approval rule that applied before still applies: money, publishing, irreversible actions, secrets and configuration changes go to the owner.
- Before the first action, re-check that the stated state is still true: the paths exist, the branch and last commit match, the tests give the recorded result. If they do not, report the mismatch instead of guessing.
- Start by answering the Skill gap line, then do the "Continue from here" step.

## Checks before you say done

1. `ls <path>` shows `HANDOFF-<slug>.md` in the agreed location, and the name is ASCII only.
2. All eleven sections are present; none is silently missing.
3. Every item in "Done and proof" has a command or observation next to it. Items without proof are moved to "Open and blockers" as unverified.
4. Every path in "Paths and values" exists (`ls` each one) or is marked as planned.
5. The secrets grep from step 4 shows no credential values.
6. "Continue from here" contains exactly one action, and that action is executable without reading the conversation.
7. The cold-read test from step 5 passed on all four questions.
8. The receiver has the seed line, and for a cross-machine transfer the file is confirmed present on the receiving side.

## Failure modes

| Symptom | Cause | Fix |
| :- | :- | :- |
| Receiver redoes or reverts earlier work | Decisions recorded without reasons or without the rejected options | Rewrite "Decisions" with why and what was rejected |
| Receiver reports "done" items as broken | State was written from memory | Redo step 2; paste real output and exit codes |
| Receiver asks "where do I start?" | "Continue from here" holds a list, or a goal instead of an action | Reduce it to one concrete action |
| Receiver spends money, publishes or deletes | Handoff was read as authorization | State approval rules in "Boundaries"; receiver follows "Rules for the receiver" |
| A credential ends up in a synced folder | Secret pasted as a value | Remove it, rotate the credential, keep only a pointer |
| File never arrives on the other machine | Non-ASCII file name, or sync not confirmed | Rename per step 1; confirm arrival per step 6 |
| Handoff is stale when picked up | Time passed, state moved | Receiver re-checks state before acting and reports mismatches |
| Worker session waits forever or dies mid-wait | Worker set its own timer or watcher | Orchestrator owns scheduling; worker does one event and exits |
