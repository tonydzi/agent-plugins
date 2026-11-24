---
name: simplest-build-first
description: "Pick the simplest, cheapest and least fragile way to build something before you write code, and check it again before you say done. Use before building a script, pipeline, hook, skill, scheduled job or integration, and when asked to simplify, cut cost or ship an MVP."
---

# simplest-build-first

Agents over-build. They write a new service where one command would do, add a database where a file would do, and wire a tool protocol where a plain file exchange would do. Each extra part is rent paid in every future session, in tokens and in repair time. This skill makes the agent choose the build that its owner can see break and fix in a minute, and that costs the fewest tokens.

Order of importance: 1) it does not break silently, 2) it is cheap, 3) it is simple.

## When to use

- Before you build anything: code, a script, a scheduled job, a skill, a hook, a link between two systems.
- When asked to "simplify", "make it cheaper", "build an MVP" or "do the smallest thing".
- Before you say "done" on something you built.

Most builds are a pipe: data moves from point A to point B, or from B to C. Treat every build as one pipe between two named points.

## Steps

### 1. Before building: five questions, stop at the first "yes"

1. **Can I not build it?** A one-off pain is solved by hand or by one existing command.
2. **Do we already have it?** Search the repo, scripts, notes and past work. Take what exists and fix it.
3. **Is there a mature standard or ready tool?** Take it. Borrow its model and vocabulary; do not drag a heavy implementation in.
4. **Is a file, a table or one command enough?** Take that.
5. **Only now write your own:** one file, one job, no new services, databases or dependencies.

Name the rung you stopped at in your report.

### 2. Several working paths: take the cheapest in tokens

Among paths that fail loudly and are fixed with one command, take the cheapest and name it. Reliability comes before price: a cheap path that lies silently is not an option. The ladder, cheap to expensive:

1. a ready result, a queue, a job with no model call;
2. a command-line tool, a script, SQL, grep;
3. a model on a subscription you already pay for;
4. your own model on one chunk of text;
5. a browser driven by screenshots, or waking a large old session.

Take rung 5 only with the reason stated in one line, and in the same pass order the cheaper path for next time.

Rules people forget:

- **Tool protocol vs. API and files.** A tool server's schema sits in the context of every session, and each call routes its answer through the model. An API, a CLI or files on disk cost zero tokens. They break a bit more often, but they break loudly and are fixed with one command. Default: file or API; a tool server only for a one-off live action, started for the job and removed after.
- **Someone else's ready tool beats your own code.** A mature CLI or binary (ffmpeg, yt-dlp, gh, rclone, sqlite3) comes before your own script.
- **Messages to people go through one sending backend** that owns dedup, rate limits and anti-ban budget. A direct API call that bypasses it is cheaper in tokens but loses those guards.

### 3. Added complexity: decide it yourself and own it

Complexity means a new field, table, dependency, service, automation or abstraction. It passes only if all three hold:

- the pain has already happened two or more times (for a guard, hook or new rule: three times);
- it breaks loudly and is easy to fix or rip out;
- there is no simpler way to get 80% of the benefit.

All three hold: build it. Any one fails: by default, do not. If you still think it is needed, the argument must be a named pain and what breaks without it; "I want it" is not an argument. Choosing complexity means you maintain it: you support it, fix it and remove it when it stops paying off. Ask yourself first: will I maintain this every week? No means do not build.

Mark it in the artifact itself with three lines:

```
COMPLEXITY: <the pain it treats>
WITHOUT IT: <what breaks>
CHEAPER ALTERNATIVE: <the option you rejected>
```

Removing a complexity together with the data it collected is a deletion: get the owner's approval first.

### 4. MVP = a paper pipe with a counter

No connector between B and C? Build a paper pipe: an MVP from ready parts in about an hour, with no new services. Paper soaks through when the water rises, and that is fine while there is little water. An expensive MVP is a defect.

The pipe is born with three things:

1. **A counter.** Reuse an existing usage log if you have one. Otherwise append one JSON line per use to a log file: `ts`, `node`, `outcome` (`ok` or `fail`).
2. **A threshold.** Write `upgrade_at: N` in the pipe's header (skill frontmatter or script docstring). N = three times the expected load for 30 days (expected once a day: 90). A second threshold is built in: 3 failures in 30 days, whichever comes first.
3. **A shout.** A small check reads the counters and prints `UPGRADE ME: <kind> <name> <count>/<threshold> failures <k>` with a non-zero exit when a threshold is crossed. Run it on a schedule that reports somewhere a human reads, and by hand before changing any pipe.

When a pipe shouts, replace paper with plastic in the same pass, or make it the first task of the next session. Build the plastic through steps 1 to 3 as if the paper pipe did not exist: question 2 looks for ready plastic (a standard, someone else's tool), not for this same pipe. The counter has already proven the pain, so the "two times" condition of step 3 is met. The plastic pipe also gets a counter and a new threshold.

Exception: where a duplicate or a loss is unacceptable (a message to a customer, a payment, a publication), one confirmed duplicate or loss means repair now; do not wait for the count.

After replacing paper with plastic, read the new result at C and check that a repeat run is safe. If that fails, return to the previous working path.

A counter at zero for 30 days is the opposite signal: nobody uses the pipe, it is a candidate for removal.

## Checks before you say done

- The design is explained in three sentences.
- The owner can find where to fix it without you.
- The part can be ripped out with one command.
- No copies: one source of truth, everything else links to it.
- The consumer C has read the new result: one correctly received example is visible at C, not only exit code 0 at B.

Any check fails: simplify, do not hand it over. "We already have it" counts only after you compare one example end to end: input, output, account, permissions, limits and behaviour on a repeat run.

## Failure modes

- **Building because it is interesting.** No answer to question 1 or 2 was recorded. Fix: answer the five questions in writing before the first file.
- **The cheap path that lies.** A script that exits 0 while writing nothing. Fix: reliability first; read the result at C.
- **Tool server by fashion.** A protocol server kept loaded for a job that a CLI or file does for zero tokens.
- **Complexity with no owner.** A new service or table added without the three-line mark; nobody maintains it and it rots.
- **Paper that never becomes plastic.** No counter or threshold, so nobody notices the pipe is soaking through.
- **Done at B, not at C.** The producer is green, the consumer never received a valid example.
- **"Already exists" taken on faith.** The existing thing has a different account, limit or repeat behaviour; check one example end to end.

## Examples

| Task | Simple path | Why |
|---|---|---|
| Is a lead hot or cold | a manual label | a scoring formula can be wrong silently, a label cannot |
| Sync notes between two machines | one rail, such as a git remote | instead of two rails and an open port on every machine |
| Report with numbers | a shared spreadsheet | instead of your own web dashboard with a server |
| Video, audio, downloads | ffmpeg, yt-dlp, rclone | a ready binary is simpler than your own code |
| Heavy mechanical work (extraction, drafts) | a model on a subscription you already pay for | your main model costs more |
| Is a pipe soaking through | one check over existing counters and an `upgrade_at:` line | no new counter per pipe |
