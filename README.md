English | [简体中文](README.zh-CN.md)

# pstack-herdr

This is a fork of the pstack plugin from [cursor/plugins](https://github.com/cursor/plugins). Lauren Tan wrote pstack and released it under the MIT license. This fork keeps that plugin and sends roles through herdr. The upstream plugin README is [pstack/README.md](pstack/README.md). The guide is [pstack/docs/guide/README.md](pstack/docs/guide/README.md).

## Contents

- [What this fork changes](#what-this-fork-changes)
- [How dispatch works](#how-dispatch-works)
- [What you need installed](#what-you-need-installed)
- [Install](#install)
- [Quick start](#quick-start)
- [Default role tables](#default-role-tables)
- [Skills](#skills)
- [Change a role](#change-a-role)
- [When something fails](#when-something-fails)
- [Upstream and license](#upstream-and-license)
- [Not verified](#not-verified)

## What this fork changes

Upstream pstack sends a role to a Cursor Task with a model slug such as `grok-4.7-xhigh-fast` or `claude-opus-5-5-xhigh`. This fork adds `hc:<program>` and `hc:<program>:<model>`. Those values open a herdr window with `hc` instead of a Task.

`setup-pstack` runs `command -v` for `claude`, `codex`, `grok`, and `pi`, and also for `hc` and `herdr`. It writes the role table from the commands it finds. `herdr` is not required. The `# agents:` line lists `herdr` only when `command -v` finds it. If `hc` is missing, every role stays on the upstream table with no `hc:` values.

When `hc`, Claude, Codex, Grok, and Pi are all installed, code roles use `hc:codex`, research roles use `hc:grok`, and swarm workers use `hc:pi`. Panels are Claude plus `hc:codex` and `hc:grok`. Pi is not on a panel.

When only `hc` and Codex are installed, the table uses `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol`, and `hc:codex:gpt-6-luna`.

`harness/pstack/claude-code/` is the generated Claude Code tree. `harness/pstack/codex/` is the generated Codex tree. The Codex tree has `.codex-plugin/plugin.json`, and the repo marketplace is `.agents/plugins/marketplace.json`.

## How dispatch works

The rules live in [pstack/skills/setup-pstack/SKILL.md](pstack/skills/setup-pstack/SKILL.md) and in the Subagents section of [pstack/skills/poteto-mode/SKILL.md](pstack/skills/poteto-mode/SKILL.md).

A value `hc:<program>` does not spawn a Task. The parent reads `hc --skill`, then opens a named window.

```bash
hc run --bg --name <short-name> <program> "<brief>. When done run: hc send $HC_TERMINAL_ID <result>"
```

The parent waits with `hc wait <short-name>`, reads with `hc read`, and can follow up with `hc send`. Then the parent checks the result. On the four-agent table the parent is Claude. On the Codex-only table the parent is the Codex session. Day-to-day acceptance is that parent running the verification from the brief. The parent does not trust the child window's own report.

`hc:<program>:<model>` adds a model argument. For Codex the verified form is `codex exec -m <model>`.

```bash
hc run --bg --name <short-name> --cwd <git-repo> codex exec -m gpt-6.1-sol "<brief>. When done run: hc send $HC_TERMINAL_ID <result>"
```

A bare `hc:codex` is the same command without `-m`. For Pi, `hc:pi:<model>` passes `pi --model <model>`. Verified Pi flags are `--model`, `--thinking`, and `-p`. A bare `hc:pi` does not pass `--model`. Other Pi flags were not verified, and neither was a value for `--thinking`. A model flag for `claude` or `grok` was not verified. Do not invent one. Run that program with no model argument and say the model was not applied.

The model that wrote the code does not review it, and a review role does not reuse the window that wrote the code. On the four-agent table, code from `hc:codex` is reviewed by Claude, by Grok, or by both. Code from Claude is reviewed by Codex. On the Codex-only table, a `gpt-6.1-sol` writer is reviewed by `gpt-6-astra`, and a `gpt-6-astra` writer is reviewed by `gpt-6.1-sol`. A panel of one vendor agrees more easily than a panel of several vendors. Formal review is [interrogate](pstack/skills/interrogate/SKILL.md), using the `interrogate reviewers` line.

`hc:pi` is DeepSeek. It is for an edit that names the exact files, the exact change, and a concrete verification command. If the task cannot be specified that precisely, do not send it to `hc:pi`. Use `hc:codex` when Codex is installed, otherwise `inherit-parent`, and say so. Pi is not used for exploration, investigation, reflect, or any review panel.

If `hc` is missing, say so and use `inherit-parent` for that role. If one program is missing, say which fallback you used. Never do the work silently.

- Missing `codex`. Code roles stay on `grok-4.7-xhigh-fast`. Drop `hc:codex` from panels.
- Missing `grok`. `how explorer`, `why investigators`, and `reflect tooling` become `inherit-parent`. Drop `hc:grok` from panels.
- Missing `pi`. `swarm workers` becomes `hc:codex` when Codex was found, otherwise `grok-4.7-xhigh-fast`.
- Missing `claude`, with Codex found and the orchestrator not already `gpt-6-astra`. Each `claude-opus-5-5-xhigh` entry becomes `hc:codex:gpt-6-astra`. If the orchestrator is already `gpt-6-astra`, use `inherit-parent`.

`gpt-6.1-terra` is a Codex model for writing, visual understanding, and computer use. Do not put it on a code role.

## What you need installed

`hc` is the launcher. Without it, setup keeps the upstream role table.

`herdr` itself does not have to be on `PATH`. Detection still runs `command -v herdr` and records it only when the command resolves.

The agent commands setup looks for are `claude`, `codex`, `grok`, and `pi`. Pi is the DeepSeek program. The command name `pi` is written in one list in [setup-pstack](pstack/skills/setup-pstack/SKILL.md).

On Windows, detection uses `command -v` in Git Bash. `hc`, `codex`, and `pi` can be npm shims, a `.ps1`, a `.cmd`, and an extensionless file. `hc run` launches those shims. No `.cmd` or `.exe` workaround is required. That was checked on Codex CLI 0.167.0 with `hc run --bg --name x codex exec -m gpt-6-luna "<task>"` and `hc run --bg --name x pi ...`.

Codex exits with `Not inside a trusted directory and --skip-git-repo-check was not specified` outside a trusted git repo. Pass `--cwd` on `hc run` pointed at the repo you are editing, or add `--skip-git-repo-check` to `codex exec` and say so. Prefer `--cwd`.

On Codex CLI 0.167.0 the defaults are reasoning effort `xhigh`, approval `never`, and sandbox `workspace-write`. A dispatched Codex edits files without prompting. The brief must name the files in scope and what it must not change.

## Install

### Claude Code

The commands in [pstack/README.md](pstack/README.md) are the upstream marketplace commands.

```bash
/plugin marketplace add cursor/plugins
/plugin install pstack@cursor-plugins
```

Those install pstack from [cursor/plugins](https://github.com/cursor/plugins), not from this fork. This repo's Claude Code tree is `harness/pstack/claude-code`. `.claude-plugin/marketplace.json` points at `./harness/pstack/claude-code`. Registering this private repository as a Claude Code marketplace was not verified.

[pstack/README.md](pstack/README.md) says Claude Code prefixes plugin commands, so `/poteto-mode` is `/pstack:poteto-mode` and setup is `/pstack:setup-pstack`. That command writes `~/.claude/rules/pstack-models.md`.

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

Install `pstack`, start a new session, and run setup. Documented marketplace commands also include `codex plugin marketplace list`, `codex plugin marketplace upgrade`, and `codex plugin marketplace remove`. Those pages do not document a shell `codex plugin install`. The Codex IDE extension does not load plugins.

Whether `codex plugin marketplace add` succeeds for this private repo was not verified. Git still needs read access. One third-party report says that on Windows `codex plugin marketplace add owner/repo` failed inside Codex's git staging clone, and `git clone` plus `codex plugin marketplace add .` worked. That report was not rechecked here.

If the marketplace install is unavailable, copy or symlink `harness/pstack/codex/skills` into `$HOME/.agents/skills` or the repo's `.agents/skills`.

The name you type to start `setup-pstack` inside Codex was not verified. Once the skill is running, it writes the role block into the global instructions file Codex reads. That file is `$CODEX_HOME/AGENTS.override.md` when the variable is set and the override file is non-empty. Otherwise it is `$CODEX_HOME/AGENTS.md`, or `~/.codex/AGENTS.md` when `CODEX_HOME` is unset. The skill replaces only the block between `<!-- pstack-models:start -->` and `<!-- pstack-models:end -->`.

## Quick start

1. Install the tree for the host you are using. See [Install](#install).
2. Run setup before the first real task. In Claude Code, [pstack/README.md](pstack/README.md) says the command is `/pstack:setup-pstack`. In Codex, the invocation name was not verified. See [Not verified](#not-verified).
3. Setup detects `hc` and the agent CLIs, asks for a budget, shows the role table, and writes the rule after you accept it. The four budget labels are `unlimited`, `large`, `medium`, and `small`. With no existing rule, `large` matches the skill defaults. `inherit-parent` and `auto` mean the role runs on the parent model. An `hc:` value does not change when you pick a budget.
4. Start the work with poteto-mode. In Claude Code the plugin README says that command is `/pstack:poteto-mode`.

Example, four agents installed, Claude Code as the parent. You describe a bug and invoke poteto-mode. The [bug-fix playbook](pstack/skills/poteto-mode/playbooks/bug-fix.md) reproduces the bug, then delegates the fix to the configured `bug-fix` role. On the four-agent table that role is `hc:codex`, so the parent opens a herdr window for `codex`, sends the brief, waits, and reads the result. The playbook's `how` pass uses `how explorer`, which is `hc:grok`. If the fix crosses a function boundary, `architect` runs first, and `architect runners` are `claude-opus-5-5-xhigh`, `hc:codex`, and `hc:grok`. The parent then runs the verification itself. The Codex window does not review its own diff. Review is Claude, Grok, or both. A contested design goes to `interrogate`, whose reviewers are the same three entries. `swarm workers` is `hc:pi` only when that brief names the exact files, the exact change, and a verification command. Otherwise the parent says Pi does not fit and uses `hc:codex`.

A feature follows the [feature playbook](pstack/skills/poteto-mode/playbooks/feature.md). The code-writing delegate uses the `feature, refactoring` role, which is also `hc:codex` on this table. `how explorer` is still `hc:grok`.

Example, only `hc` and Codex installed. The parent is the Codex session. A bug-fix or feature delegate is `hc:codex:gpt-6.1-sol`. `how explorer`, `why investigators`, and `reflect tooling` are `hc:codex:gpt-6.1-sol` too. Judgment, prose, the hardest tasks, explainers, and synthesizers are `hc:codex:gpt-6-astra`, or `inherit-parent` when the orchestrator is already `gpt-6-astra`. `swarm workers` is `hc:codex:gpt-6-luna` only for a fully specified edit. Otherwise that role is `hc:codex:gpt-6.1-sol`. The parent still runs the verification. A Sol writer is reviewed by Astra. An Astra writer is reviewed by Sol.

This page does not show sample tool output. The role names above are the lines in [setup-pstack](pstack/skills/setup-pstack/SKILL.md).

## Default role tables

Four agents, with `hc`. No model suffix.

| Role | Value |
| --- | --- |
| `feature, refactoring` | `hc:codex` |
| `bug-fix` | `hc:codex` |
| `perf-issue` | `hc:codex` |
| `hillclimb` | `hc:codex` |
| `judgment and prose` | `claude-opus-5-5-xhigh` |
| `hardest tasks` | `claude-opus-5-5-xhigh` |
| `how explorer` | `hc:grok` |
| `how explainer` | `claude-opus-5-5-xhigh` |
| `why investigators` | `hc:grok` |
| `why synthesizer` | `claude-opus-5-5-xhigh` |
| `reflect tooling` | `hc:grok` |
| `reflect judgment, divergent, synthesizer` | `claude-opus-5-5-xhigh` |
| `arena runners` | `claude-opus-5-5-xhigh`, `hc:codex`, `hc:grok` |
| `arena cross-judge pool` | `claude-opus-5-5-xhigh`, `hc:codex`, `hc:grok` |
| `swarm workers` | `hc:pi` |
| `architect runners` | `claude-opus-5-5-xhigh`, `hc:codex`, `hc:grok` |
| `interrogate reviewers` | `claude-opus-5-5-xhigh`, `hc:codex`, `hc:grok` |

Codex only, with `hc`. If the orchestrator is already `gpt-6-astra`, the five judgment lines are `inherit-parent` instead of `hc:codex:gpt-6-astra`. `swarm workers` is `hc:codex:gpt-6-luna` only for a fully specified edit, and `hc:codex:gpt-6.1-sol` otherwise.

| Role | Value |
| --- | --- |
| `feature, refactoring` | `hc:codex:gpt-6.1-sol` |
| `bug-fix` | `hc:codex:gpt-6.1-sol` |
| `perf-issue` | `hc:codex:gpt-6.1-sol` |
| `hillclimb` | `hc:codex:gpt-6.1-sol` |
| `judgment and prose` | `hc:codex:gpt-6-astra` |
| `hardest tasks` | `hc:codex:gpt-6-astra` |
| `how explorer` | `hc:codex:gpt-6.1-sol` |
| `how explainer` | `hc:codex:gpt-6-astra` |
| `why investigators` | `hc:codex:gpt-6.1-sol` |
| `why synthesizer` | `hc:codex:gpt-6-astra` |
| `reflect tooling` | `hc:codex:gpt-6.1-sol` |
| `reflect judgment, divergent, synthesizer` | `hc:codex:gpt-6-astra` |
| `arena runners` | `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol` |
| `arena cross-judge pool` | `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol` |
| `swarm workers` | `hc:codex:gpt-6-luna` |
| `architect runners` | `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol` |
| `interrogate reviewers` | `hc:codex:gpt-6-astra`, `hc:codex:gpt-6.1-sol` |

Astra is the most capable of these Codex models. Sol is near Astra and cheaper. Luna is the efficient model for a clear, repeatable edit. Any other mix of installed programs starts from the four-agent table and applies the fallbacks in [How dispatch works](#how-dispatch-works). Setup says which table it picked.

If `hc` is missing, the table is the upstream one. Code roles, research roles, and swarm workers are `grok-4.7-xhigh-fast`. Judgment and synthesis stay on `claude-opus-5-5-xhigh`. Panels are `claude-opus-5-5-xhigh` and `grok-4.7-xhigh-fast`.

## Skills

Each line is the skill's own description, shortened. The files are under [pstack/skills](pstack/skills).

- [poteto-mode](pstack/skills/poteto-mode/SKILL.md). Concise, detailed responses, deliberate subagents, plain prose, simple code, and verified work. In Claude Code the plugin README says the command is `/pstack:poteto-mode`.
- [poteto-help](pstack/skills/poteto-help/SKILL.md). Helps with setup, `/poteto-mode`, and picking a skill, playbook, or principle. The command in the skill is `/poteto-help`.
- [swarm](pstack/skills/swarm/SKILL.md). Fans out N parallel workers, drains them, and returns one report. Workers use the `swarm workers` role.
- [arena](pstack/skills/arena/SKILL.md). Spawns N candidates on the same task, picks a base, and grafts the strongest parts of the others into it. Runners come from `arena runners`.
- [interrogate](pstack/skills/interrogate/SKILL.md). Adversaries try to break a change. Only a finding a run proves survives. Reviewers come from `interrogate reviewers`.
- [architect](pstack/skills/architect/SKILL.md). Sketches types, signatures, and module structure before code. Runners come from `architect runners`.
- [tdd](pstack/skills/tdd/SKILL.md). Use it when you ask for TDD or a failing test, or when the bug has a cheap local test. Skip it when the test would be expensive or unclear.
- [unslop](pstack/skills/unslop/SKILL.md). Cuts AI tells from writing.
- [no-comments](pstack/skills/no-comments/SKILL.md). Spawns Comment Sicko, fixes accepted findings, and offers encodings for claimed constraints.

`how`, `why`, and `reflect` are the research skills behind `how explorer`, `why investigators`, and `reflect tooling`. Their files are [how](pstack/skills/how/SKILL.md), [why](pstack/skills/why/SKILL.md), and [reflect](pstack/skills/reflect/SKILL.md).

## Change a role

Run setup again. It shows every role and asks you to accept the table or change a role. The choices are the detected models, each valid `hc:<program>` value, `inherit-parent`, and `auto`. `inherit-parent` and `auto` run that role on the parent model. For Cursor, omit Task `model`. On an `hc:` dispatch, do the role in the parent session and do not open another window.

You can also edit the rule file. Delete a role line to fall back to the skill default. In Cursor the file is `~/.cursor/rules/pstack-models.mdc`. In Claude Code it is `~/.claude/rules/pstack-models.md`. In Codex it is the marked block in the AGENTS file from [Install](#install). A new session picks up the change. Setup says so when it finishes.

On a later run, setup keeps a role you set to something other than a default this skill writes. A role that still matches an older herdr default is not treated as your edit. In particular, `swarm workers` left on `hc:codex` becomes `hc:pi` when Pi is found, and setup says that it changed the line.

## When something fails

`command -v hc` finds nothing. Setup keeps the upstream table and should say that `hc` is missing. Do not write an `hc:` value.

Codex prints `Not inside a trusted directory and --skip-git-repo-check was not specified`. Point `hc run --cwd` at the git repo, or add `--skip-git-repo-check` to `codex exec` and say so.

One agent CLI is missing. Use the fallback for that program in [How dispatch works](#how-dispatch-works), and say which one. Do not leave the role on an `hc:` value for a program that was not found.

Pi was asked to decide the edit itself. Do not dispatch that task to `hc:pi`. Send it to `hc:codex` or `inherit-parent` and say so.

The child finished and the parent has not run the verification. The task is not accepted yet. The parent runs the verification against the brief.

## Upstream and license

pstack comes from [cursor/plugins](https://github.com/cursor/plugins). [pstack/LICENSE](pstack/LICENSE) is unchanged and says `Copyright (c) 2026 Lauren Tan`. The license for this repository, including the fork's modifications, is [LICENSE](LICENSE). It keeps that line and adds `Copyright (c) 2026 THonour99`. Both are the MIT license.

This fork's history is the cursor/plugins history plus the commits on this repo. A repeatable command for merging later upstream commits was not verified.

## Not verified

- Registering this private repository as a Claude Code marketplace was not verified. The `/plugin marketplace add cursor/plugins` commands install upstream pstack, not this fork.
- Whether `codex plugin marketplace add THonour99/pstack-herdr` succeeds for this private repo was not verified. Git still needs read access.
- One third-party report says that on Windows `codex plugin marketplace add owner/repo` failed inside Codex's git staging clone, and `git clone` plus `codex plugin marketplace add .` worked. That report was not rechecked here.
- The documented Codex commands are `codex plugin marketplace list`, `upgrade`, and `remove`. Those pages do not document a shell `codex plugin install`. The Codex IDE extension does not load plugins.
- The name you type to start `setup-pstack` inside Codex was not verified.
- A repeatable command for merging later commits from [cursor/plugins](https://github.com/cursor/plugins) was not verified.
- A model flag for the `claude` and `grok` CLIs was not verified. Other Pi flags, and a value for `--thinking`, were not verified.
- More unverified items are listed in [pstack/docs/harness/codex.md](pstack/docs/harness/codex.md) and [pstack/docs/harness/claude-code.md](pstack/docs/harness/claude-code.md).
