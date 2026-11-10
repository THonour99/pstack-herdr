# pstack-herdr

Private herdr-adapted fork of [cursor/plugins](https://github.com/cursor/plugins), containing only the pstack plugin.

`pstack/` is the Cursor plugin. `harness/pstack/claude-code/` is the generated Claude Code copy. `harness/pstack/codex/` is the generated Codex plugin. Codex installs it with `codex plugin marketplace add THonour99/pstack-herdr`, then the plugins browser. The repo is private, so git needs read access. After editing pstack sources, run `node scripts/generate-pstack-harness-trees.mjs` from the repository root and commit the result.
