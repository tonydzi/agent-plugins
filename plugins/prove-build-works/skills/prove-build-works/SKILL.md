---
name: prove-build-works
description: "Acceptance gate to run right after building or changing a script, hook, skill, scheduled job, pipeline or CLI. Run it live, break it on purpose, show a test red on broken code, get a review from a different vendor's model, re-check by a second method, then give PASS, PARTIAL or FAIL with evidence. Use before saying done, fixed or it works."
---

# prove-build-works

Agents say "done" when the work is two-thirds done: the code exists, one happy-path run was green, and nobody tried to break it. This gate treats "done" as a claim. A claim needs evidence: a live run, a deliberate break, a visible result and a second pass by a different method.

It checks one thing: the artifact this task just built or changed, while the context is still hot. It is not a session retrospective and not a system health check.

## When to use

- Right after you build or change something that must do something: a script, CLI, hook, gate, linter, skill, scheduled job or data pipeline.
- Before you write "done", "fixed", "it works" or a green check mark.
- Before you write "impossible" or "the tool does not support this" (step 2).

Skip it for conversational answers and trivial text edits. If the task built nothing, say "nothing to test" and stop.

## Steps

Every step ends with evidence: the command, its output and its exit code. "I checked" is not evidence.

1. **Scope from facts, not from memory.**
   - List what this task created or changed: `git status --short` and `git diff --stat`, plus any file outside git that you touched.
   - Take the acceptance criteria as they were written before the work started. If none were recorded, write `BASELINE_MISSING`. Do not swap in a later list of convenient criteria and do not claim full acceptance.
   - Check that the thing was not already built: `grep -ril "<2-4 task words>"` over the script, tool and hook directories, not only over the notes. Report how many directories and files you searched. A hit means you read it before you keep your own version.

2. **About to say "impossible"? Ask the tool first.** Go through this order and do not skip rungs:
   1. `<tool> --help`, `-h` and `help <subcommand>`: the full flag list, not the one you remember.
   2. Each subcommand's own `--help`.
   3. The tool's config file and environment variables.
   4. Documentation for the same version.
   5. Only then logs, reverse engineering and experiments.

   If you did not finish the order, the only allowed wording is "I did not find how to do this, I looked in: ...". An expensive investigation does not replace the cheap rungs.
   The reverse trap: `--help` proves a flag exists, not that it works on your data. A flag your verdict depends on must be called on real data in the same pass, or you write "flag exists, not verified on our data".
   Run `<tool> --help; echo $?`. A non-zero code or a traceback is a defect to fix, not a missing answer.

3. **Run it live.** Run the artifact on real data and show the actual output.
   - Script: run it. Use read-only or dry-run mode if it has side effects.
   - Skill or procedure: execute its steps by hand here.
   - Hook or scheduled job: trigger it the way its runner does and read the output it leaves.
   - Use the same interpreter and binary the consumer will use. Print `which <binary>` or `sys.executable` and compare it with the scheduler entry or the caller's command line.
   - "Works in theory" does not count.

4. **Break it on purpose.**
   - Feed it empty input, malformed input, a missing dependency and a missing key. It must fail with a clear error, not silently pretend to work.
   - Search the changed files for hardcoded machine paths (a home directory with a user name, a drive letter) and for facts read from a stale copy instead of the live source.
   - If you fixed a choice (engine, vendor, model, route), call the artifact with no choice at all and prove the default is the one you claim. A selection bug has two doors: "named wrong" and "not named".
   - If you built a metric or a score, write down its theoretical range and add a case that would exceed it. For a ratio, write down what one row of the numerator and one row of the denominator physically mean. Different units give a number that lies silently.
   - Find every place where an honest measurement falls back to a worse one. A fallback may feed a report. It must not feed an action such as kill, send, write or commit.
   - If you touched shell code, find every place an exit code is read after a pipe. `$?` there belongs to the last stage: `python3 -c "import sys; sys.exit(3)" 2>&1 | tail -1; echo $?` prints 0. Use `set -o pipefail` or `${PIPESTATUS[0]}`, or do not pipe the command whose code you read.
   - If a gate or filter decides by `"word" in text` or by `grep` over JSON, YAML or code, check that it judges structure. A short word matches inside longer words and matches keys as well as values. Structural parsing is required where the source is structured and the verdict blocks, skips or publishes something. Plain grep over flat human text stays fine.

5. **Show the test red.** Every fix closes with a test that failed on the broken code.
   - Break a copy, never the live file. Record four things: the red command, its non-zero exit code, a checksum showing the original is unchanged, and the green command with exit 0.
   - A refusal to start is not red. A usage error or an import error proves nothing about the behavior under test.
   - Strictness canary: call the test with a junk flag. It must exit non-zero. A test that swallows unknown arguments can report green from the wrong mode.
   - Every case must go red, not just the file. One honest failing case hides neighbors that never fail. The red branch must break the object under test, not delete the failure.
   - Check that each assertion looks at the value the break moves. Otherwise the check is vacuous from birth and a red run will not show it.
   - Before you trust a tool to guard a defect class, run it on a file you know is broken. If it stays silent, it cannot see the class.
   - Fail-closed gate: disable its guards one at a time and run the suite. A mutant that stays green is a guard nobody protects. Run mutant series with `PYTHONDONTWRITEBYTECODE=1`, because bytecode caches are keyed by mtime and size and a quick mutation can replay the previous mutant. Each mutant must kill exactly its own check. Finish with a control run on the restored original.
   - Checker driven by a pattern (regex, stop list, glob, SQL LIKE): write an edge table of 10 to 20 rows of "input, expected yes or no", run it and report misses as a number. Cover letter case, Unicode dashes, a line break inside a word, plural, possessive, prefix words and legitimate homonyms. Test false positives first: a false alarm makes someone "fix" correct text.

6. **Get a second opinion from a different vendor's model.** Your own check is blind to your own blind spots.
   - Subagents of the same model are not a second opinion. Use another vendor's CLI or web chat, read-only.
   - Send the live code of the changed parts and the list of what you declared fixed. Do not send your paraphrase. The task for the reviewer is "break these fixes".
   - Read the answers, not the summary line. If the reviewer says no code or diff was provided, the subject never arrived and the review counts as skipped.
   - A finding is a hypothesis. Verify each against the source and report two columns: accepted, and refuted with evidence. The same finding from two or more models independently: fix it.
   - A reviewer who approves must run at least one check and show its exit code. An opinion without a run is a hypothesis, not acceptance.
   - Skipping is allowed only explicitly, with the reason and every channel you tried. An explicit skip caps the verdict at PARTIAL, unless no executable artifact changed.

7. **Check visibility.** The most common silent bug is that nobody can see whether the thing ran.
   - If the artifact has its own deterministic test or gate, run that and take its verdict. Do not re-derive it with ad-hoc grep.
   - Name the log line, counter or output file that proves a run happened. An exit status with no log is not debuggable and fails this step.
   - Take one red line of the output and ask whether a person can tell from it what to fix. A red flag printed with the wrong reason is a lying indicator.
   - An empty set is not green. A check over zero items prints "nothing to judge", not PASS.
   - An instrument states its coverage: "inspected N of M" and "cannot judge K of M". If it has a cap, the cut happens after sorting and the cap is printed.
   - Then apply every row of this table whose trigger matches. A missed row caps the verdict at PARTIAL.

   | Trigger | Check before the verdict |
   | :- | :- |
   | It walks a growing set (files, threads, rows) | Does run cost grow with total volume or with the number of changes? If with volume, cache the verdict by a cheap change marker from the source (`updated_at`, ETag, hash, mtime). |
   | It is public (README, page, package, post) | Fetch it anonymously with no token or cookie and match a specific string from the artifact. Status 200 with an empty body is the usual silent failure. |
   | It writes to a target picked by config (`default`, `current`, first in list) | Prove the target is the one in use. The liveness sign must not include a file the tool itself writes. Print any mismatch between config and fact. |
   | It relies on a join, dedup or idempotency key | Count the share of records that carry the key. A key empty in most records is a gate that is always open and never red. Name the code that reads the key and the code that fills it. |
   | It moves items into an in-progress state (claim, lease, processing) | Who returns the item if the holder dies? From which event is the hold age measured? Does the health counter see the in-progress bucket? |
   | It scores or filters a mixed stream | Prove each item is of the expected type by a positive sign. Count true against false hits for every dictionary word on live data. |
   | It links your record to an external object | Use a structural key (domain, id in URL, a field), not a name found in free text. An ambiguous match returns nothing. Name the channel the tool does not look at. |
   | It is a hook, gate or watchdog | Three proofs, each a command with an exit code: it is registered in the runner's config; an event on stdin through the exact registered command gives the expected exit code; the runner's own log shows it fired. A file with a green test and no registration is not built. |
   | It is a PASS/FAIL judge over text or objects | It has a labeled example set with a reason per label and both classes present. Report the numbers before the verdict. A rise in false positives against the frozen baseline is a regression even at equal accuracy. |

8. **Test, doc and usage counter.** Every live part leaves this gate with all three, or the verdict is capped at PARTIAL.
   - A new part gets its test and its doc in the same pass. The doc lives in the file itself: purpose, input and output, who calls it, what breaks without it.
   - Changed behavior means the doc is updated in the same session.
   - A test nobody runs does not exist. Name the schedule or suite that runs it.
   - The test names the mutations it must catch.
   - The test writes only to a temp directory. A check that writes into live data is a defect.
   - A CLI answers `--help` with exit 0 and rejects an unknown flag with a non-zero exit.
   - The part counts its real uses in an append-only log, and tests do not write into that log.

9. **Unattended loops name their backpressure.** If the artifact runs without a human in the loop (scheduled job, self-restarting agent, agent in a shell loop), it must have at least one machine pressure:
   1. a deterministic judge that each iteration must pass and that lives outside the thing it judges;
   2. a hard cap on iterations or time;
   3. a budget brake on tokens or calls;
   4. a kill switch that stops it without a code change;
   5. a "not worse" threshold that rolls the iteration back.

   Break the named pressure on purpose: one iteration must be rejected. "The prompt asks it to be careful" and "a human reviews later" are not pressure. A loop with no cap on iterations or time is FAIL, not PARTIAL.

10. **Root cause, fix, rerun.** Any failure in steps 3 to 9 on this task's artifact: find the root cause, fix the class and not the symptom, and go back to step 3. Before you declare "we never had a guard for this", open the guard's own log and find one successful run. A guard that only ever logged busy, skip or blocked was locked, not absent.
    Move the fix into a separate session with a written seed (what was done, what was rejected and why, acceptance criteria, non-goals) if any signal fires: 2 or more failed attempts; the root spans more than 2 files or an adjacent system; reproduction takes over 10 minutes; 70% or more of the context is used. The artifact is then PARTIAL.

11. **Second pass by a different method.** Steps 3 to 9 green is the first pass. Until the second pass is shown, the most you may write is "first run green, re-checking".
    1. Start from the requirement, not from the code. List what was asked, in the requester's words, and put evidence against every item.
    2. Change the method. Rerunning the same command is not a second pass. Read the changed lines from disk instead of from memory of the edit. Ask the consumer instead of the producer. Use an instrument instead of your eyes.
    3. Look at exactly the places you changed. Is the change on disk? Did it reach the consumer? Is the old behavior gone, not living next to the new one?
    4. Re-check your own instrument. A measurement tool is also a claim.
    5. State what you did not check: `checked: ... | not checked: ... (why)`.
    6. Critical changes get a third pass by outside eyes (step 6): money, anything irreversible, a rollout to many machines, outbound messages, publication, secrets.

12. **Verdict with evidence.** Give a short table of what was checked, how (live output) and the result. Then one line:
    - `PASS`: ran live, broke it, result visible, second opinion received, second pass shown. Only this means done.
    - `PARTIAL`: works, with a named gap.
    - `FAIL`: does not work or cannot be seen. Give the root cause and what you are fixing.

    Verdict ceilings. Any of these caps the verdict at PARTIAL even if everything else is green: second opinion skipped; test, doc or counter missing; a matching row of the step 7 table not checked; backpressure not named and not proven; second pass not shown; "impossible" stated without the search order of step 2.
    Self-serving check. If the conclusion is convenient for you (it closes the work, blames something external, lets you skip an unpleasant task), add one independent piece of evidence before you state it.
    PASS confirms delivery, not effect. If the artifact promised an effect (fewer tokens, faster, cheaper), add a line with the metric, the instrument, the baseline before the change and the date you will measure.

## Checks before you say done

- [ ] The scope came from `git status` or an explicit file list, and the criteria came from before the work (or `BASELINE_MISSING` is stated).
- [ ] Live output is shown, produced with the consumer's interpreter or binary.
- [ ] At least one deliberate break is shown with its exit code.
- [ ] Every fix has a test shown red on a broken copy and green on the original, and the original's checksum is unchanged.
- [ ] The junk-flag canary exits non-zero.
- [ ] A different vendor's model reviewed the live code, its answer was read, and findings are sorted into accepted and refuted. Or the skip is explicit and the verdict is PARTIAL.
- [ ] A log line, counter or output proves the run happened, and coverage is stated as N of M.
- [ ] Every matching row of the step 7 table is answered.
- [ ] Test and doc exist for every new part, and a schedule or suite runs the test.
- [ ] Any unattended loop has a named pressure that rejected one iteration when broken.
- [ ] The second pass used a different method and the line `checked: ... | not checked: ...` is present.
- [ ] The verdict line is PASS, PARTIAL or FAIL and each row of the table has evidence.

## Failure modes

- **Green by pipe.** The exit code was read after `| tail` or `| grep`. The check was green because the last pipe stage succeeded.
- **Red that never ran.** `test --flagA || test --flagB`: the first call swallowed an unknown flag, ran the green suite and returned 0, so the red command never ran.
- **Vacuous assertion.** The break changed one value and the assertion watched another. All cases stay green on broken code.
- **Tool blind to the class.** A syntax check was chosen for a defect that only appears at run time. It is green on every broken file.
- **Review of a paraphrase.** Reviewers attacked your summary, not the code. In the source measurements for this method a panel had 25% precision on a paraphrase and 10 of 10 on live code.
- **Review that never received the subject.** The summary says "N reviewers answered" while each answer says no code was provided.
- **Same-vendor panel.** Many subagents of one model found real bugs and shared the same blind spots. No other vendor was called.
- **Expensive proof of "impossible".** A long log investigation concluded "not supported" while `--help` listed the flag.
- **Flag found, not called.** `--help` showed a fitting flag, the plan was built on it, and on real data the flag expected a different input.
- **Short census reported green.** The census regex matched 3 commands of 39. The gate judged those 3 and reported clean.
- **Green suite, wrong metric.** A ranking metric returned 1.63 with a mathematical maximum of 1.0 while 78 tests passed. No test had duplicate items.
- **Guards nobody guards.** In one fail-closed gate 5 of 8 mutants survived with all 14 tests green.
- **Built and never wired.** The hook file and its test exist, the runner's config has no line for it, and it has never fired.
- **Same check twice.** The second pass reran the first command and repeated the same blindness.
- **Convenient verdict.** The conclusion closed the task and was accepted on one piece of evidence.
