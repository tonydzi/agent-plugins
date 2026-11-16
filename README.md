# agent-plugins

One catalog for my agent plugins. Add it once, then install any plugin from it by name.
Each plugin lives in its own repository under `plugin/`; this repository only lists them,
so a plugin's code and history stay in one place.

## Install

Claude Code:

```
/plugin marketplace add tonydzi/agent-plugins
/plugin install break-it-first@agent-plugins
```

Codex:

```
codex plugin marketplace add tonydzi/agent-plugins
codex plugin add break-it-first@agent-plugins
```

Replace `break-it-first` with any name from the table.

## Plugins

| Plugin | What it does | Source |
| :- | :- | :- |
| `break-it-first` | After an agent builds something, it runs it live, breaks it on purpose and asks rival models to attack it before saying done. | [tonydzi/break-it-first](https://github.com/tonydzi/break-it-first) |
| `secondop-panel` | Second opinion from reviewer models of different labs, quorum counted by model family. | [tonydzi/secondop-panel](https://github.com/tonydzi/secondop-panel) |
| `agent-leash` | LEASH-8 scorecard: 24 statements across 8 control domains, a band and the three cheapest missing controls for an agent setup. | [tonydzi/agent-leash](https://github.com/tonydzi/agent-leash) |
| `rules-as-files` | Keep rules for people and agents as one markdown file per rule, with an index line in CLAUDE.md or AGENTS.md. | [tonydzi/claude-bible](https://github.com/tonydzi/claude-bible) |
| `claw-retro` | End-of-session retrospective: what was built, what to keep, where each rule lives, and a ready /compact block. | [tonydzi/claw-retro](https://github.com/tonydzi/claw-retro) |
| `compact-canon` | Prints a filled 7-header block to paste after /compact so decisions, paths and values survive. | [tonydzi/compact-canon](https://github.com/tonydzi/compact-canon) |
| `sqlite-graph-memory` | Recall workflow for linked markdown notes through a separately configured sqlite-graph-memory MCP server. | [tonydzi/sqlite-graph-memory](https://github.com/tonydzi/sqlite-graph-memory) |
| `charm-os` | Stop hook that records each turn in a local SQLite file, plus decision memo and trace eval skills. Needs python3. | [tonydzi/charm-os](https://github.com/tonydzi/charm-os) |

All plugins except `charm-os` are instructions only: they ship markdown skills and run no code.
`charm-os` ships a Stop hook that runs locally and sends nothing over the network.

## Coming next

- `claude-memory-tidy`: keeps an agent memory index short and closes outdated records by fields. Waiting for path cleanup in its tests and scripts.
- `lambda-cloud-mcp`: lets an agent check Lambda Cloud GPU prices and availability. Waiting for version 0.1.1 on PyPI.

## How it works

`.claude-plugin/marketplace.json` (Claude Code) and `.agents/plugins/marketplace.json` (Codex)
point to each plugin with a `git-subdir` source: the `plugin` folder of the plugin's own
repository on its `main` branch. Installing a plugin fetches only that folder.

## License

[MIT](LICENSE.md) for this catalog. Each plugin carries its own license in its repository.
