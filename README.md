# pstack-herdr

This is a fork of the pstack plugin from [cursor/plugins](https://github.com/cursor/plugins) (author Lauren Tan, MIT), adapted so herdr can dispatch pstack roles to other agent programs. The upstream plugin README is [pstack/README.md](pstack/README.md). `pstack/LICENSE` is unchanged. The license for this repository, including the fork's modifications, is [LICENSE](LICENSE).

## What this fork changes

- Role values `hc:<program>` and `hc:<program>:<model>` dispatch through herdr's `hc` instead of a Cursor Task. Codex uses `hc run --bg --name <name> --cwd <git-repo> codex exec -m <model> ...`. Outside a trusted git repo, Codex requires `--skip-git-repo-check` on `codex exec`.
- `setup-pstack` detects installed agents with `command -v` for `claude`, `codex`, `grok`, and `pi`, plus `hc` and `herdr`. `herdr` is listed only when it is found. The role table is written from what was found. If `hc` is missing, the table stays the upstream no-`hc` table.
- When `hc`, Claude, Codex, Grok, and Pi are all found, code roles use `hc:codex`, research roles use `hc:grok`, swarm workers use `hc:pi`, and the panels are Claude plus `hc:codex` and `hc:grok`. Pi is not on a panel.
- When only `hc` and Codex are found, the Codex-only table uses `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol`, and `hc:codex:gpt-6-luna`. Judgment stays on Astra unless the orchestrator is already Astra, in which case those roles are `inherit-parent`. Luna is only for a task that names the exact files, the exact change, and a verification command.
- `harness/pstack/claude-code/` is the generated Claude Code tree. `harness/pstack/codex/` is the generated Codex tree, with `.codex-plugin/plugin.json` and a repo marketplace at `.agents/plugins/marketplace.json`.

## Install

### Claude Code

The commands written in [pstack/README.md](pstack/README.md) are the upstream marketplace commands:

```bash
/plugin marketplace add cursor/plugins
/plugin install pstack@cursor-plugins
```

Those install pstack from [cursor/plugins](https://github.com/cursor/plugins), not from this fork. This repo's Claude Code tree is `harness/pstack/claude-code`, and `.claude-plugin/marketplace.json` points at it. Registering this private repository as a Claude Code marketplace was not verified.

### Codex

From a shell whose git can already read this private repo:

```bash
codex plugin marketplace add THonour99/pstack-herdr
```

Then:

```bash
codex
/plugins
```

Install `pstack`, start a new session, and run setup-pstack. Documented marketplace commands also include `codex plugin marketplace list`, `codex plugin marketplace upgrade`, and `codex plugin marketplace remove`. The pages do not document a shell `codex plugin install`. The Codex IDE extension does not load plugins.

Whether `codex plugin marketplace add` succeeds for this private repo was not verified. Git still needs read access. One third-party report says that on Windows `codex plugin marketplace add owner/repo` failed inside Codex's git staging clone, and `git clone` plus `codex plugin marketplace add .` worked. That report was not rechecked here.

If the marketplace install is unavailable, copy or symlink `harness/pstack/codex/skills` into `$HOME/.agents/skills` or the repo's `.agents/skills`.

Unverified items are listed in [pstack/docs/harness/codex.md](pstack/docs/harness/codex.md) and [pstack/docs/harness/claude-code.md](pstack/docs/harness/claude-code.md).
