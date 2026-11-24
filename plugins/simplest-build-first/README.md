# simplest-build-first

Stops coding agents from over-building. Before the agent writes a new service, database or tool integration, it has to show why one command, one file or an existing tool would not do.

The skill gives the agent five steps:

1. **Five questions before building**, stopping at the first "yes": can I not build it, do we already have it, is there a mature standard, is a file or one command enough, and only then one file with one job.
2. **A token-cost ladder** for choosing between working paths: ready result, CLI or script, a paid subscription model, your own model on a chunk, and only last a screenshot-driven browser. Reliability comes before price.
3. **A three-condition test for added complexity**: the pain happened at least twice, it breaks loudly and is easy to remove, and nothing simpler gets 80% of the benefit. Accepted complexity is marked in the artifact and owned by the one who added it.
4. **MVP as a paper pipe**: built from ready parts in about an hour, born with a usage counter, an `upgrade_at: N` threshold and a check that shouts when the pipe needs replacing.
5. **Five checks before done**, the last one being that the consumer actually received a valid result.

## Install

Claude Code:

```
/plugin marketplace add tonydzi/agent-plugins
/plugin install simplest-build-first@agent-plugins
```

Codex:

```
codex plugin marketplace add tonydzi/agent-plugins
codex plugin add simplest-build-first@agent-plugins
```

Other agents (Cursor, Gemini CLI and others): copy `skills/simplest-build-first/SKILL.md` into the agent's rules or skills folder.

## License

MIT
