# five-whys-root-cause

Stop fixing the same bug a fourth time. This skill makes a coding agent prove the root cause of a recurring defect before it is allowed to design a fix.

Most "five whys" output is five plausible sentences. This one is a gated investigation where every link carries evidence and a falsifier.

## What the agent does

1. **Gate.** Runs the full method only on three dated cases of the same defect class, or on a named carve-out (security, data loss, money, a red critical-path system, a monitoring instrument that lied). A first or second case is logged and left alone.
2. **Series.** Builds a table of every case plus at least one control case with similar conditions and no defect.
3. **Refute first.** Tests the recorded cause directly against the code before building on it, and checks whether a fix already exists with no caller.
4. **One chain.** Five linked whys; each has a claim, evidence across the series, a falsifier, and a status of `PROVEN` or `HYPOTHESIS`. Out of evidence means `BLOCKED: evidence gap`, not a decorative fifth why.
5. **Proof.** A counterfactual: the defect appears with the condition and disappears without it.
6. **Minimal fix.** Designed only after the proof, with a simplicity test.
7. **Defect-return test.** Red on the broken variant, green with the fix, red again on the same named case once the guard is removed.
8. **Report.** One fixed contract: gate, chain, cause, evidence, invariant, fix, test, rollout.

## When it helps

- "It broke again" incidents and repeated review ping-pong over the same class of bug.
- Postmortems where the stated cause was never tested.
- Teams whose fixes pass on the author's machine and fail elsewhere.

It is the wrong tool for a first-time bug with an obvious cause.

## Install

Claude Code:

```text
/plugin marketplace add tonydzi/agent-plugins
/plugin install five-whys-root-cause@agent-plugins
```

Any other agent (Codex, Cursor, Gemini CLI): copy `skills/five-whys-root-cause/SKILL.md` into the directory your agent loads skills or rules from. The skill is plain markdown and uses only common tools (`git`, `grep`, and optionally the GitHub CLI `gh`).

## Use

Ask the agent: `five whys: <neutral description of the recurring defect>`, or invoke the skill by name.

## Contents

- `skills/five-whys-root-cause/SKILL.md`: the method, checks and failure modes.
- `.claude-plugin/plugin.json`: plugin manifest.

The plugin ships no scripts and no telemetry. The only network use is the optional `gh issue list` search the agent may run in your own repository.

## License

MIT
