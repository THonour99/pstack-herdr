# pstack in Codex

pstack skills use Cursor names. In Codex, read this file once per session, then map each Cursor name with the tables below. `<pstack>` means the folder that holds `skills/` and `docs/`. If a Cursor field has no row, drop it and say so. Never pass a flag this file does not mark as verified.

Sources checked on 2026-11-10:

- [Models](https://learn.chatgpt.com/docs/models) for `gpt-6-astra`, `gpt-6.1-sol`, `gpt-6-luna`, and `gpt-6.1-terra`, and for `codex -m` / `codex exec -m`.
- [Developer settings](https://learn.chatgpt.com/docs/developer-settings) for `~/.codex/config.toml`, `-c` / `--config`, and `model_reasoning_effort`.
- [AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md) for the global instructions file.
- [Build skills](https://learn.chatgpt.com/docs/build-skills) for skill directories.
- [Plugins](https://learn.chatgpt.com/docs/plugins) for the Codex CLI plugins browser.

## Tools

| Cursor | Codex |
| --- | --- |
| `Task` | No verified Codex Task tool. Dispatch with `hc run` as in Models. |
| `AskQuestion` | No verified equivalent. Ask in the session. |
| Open a todolist | No verified equivalent. Keep the list in your reply and update it at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | `Read` `<pstack>/skills/<name>/SKILL.md` and follow it. After the skill is installed under `.agents/skills`, Codex can also select it by its `name` and `description`. |
| `Grep`, `Glob` | Use the shell. Codex docs do not name a separate search tool here. |

## Subagent fields

These are Cursor Task fields. No Codex equivalent was verified. Drop the field and say so.

| Cursor | Codex |
| --- | --- |
| `subagent_type: generalPurpose` | Drop. Use `hc run`. |
| `poteto-agent`, `comment-sicko` | Drop. They are Cursor subagent types. Follow the skill text with `hc run`. |
| `readonly: true` | Drop. Say the worker was not launched readonly. |
| `readonly: false` | Drop. |
| `environment: "cloud"` | Drop. |
| `environment: "local"` | Drop. |
| `cloud_base_branch` | Drop. |
| `run_in_background: true` | Drop. `hc run --bg` is the herdr background flag the parent already uses. |
| `resume` | Drop. Open a fresh `hc run` and paste the earlier result into the brief. |
| `file_attachments` | List the paths in the brief. |

## Models

| Cursor | Codex |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh` | When `claude` is on PATH, `hc run --bg --name <short-name> claude ...`. A Claude CLI flag for that slug was not verified. Do not invent one. Say the slug was not passed. When `claude` is missing, use the Codex-only fallback in `setup-pstack`. |
| A `grok-*` slug | When `grok` is on PATH, `hc run` of `grok` with no model flag. A Grok model flag was not verified. Do not invent one. When `grok` is missing, `inherit-parent` and say so. |
| A `gpt-*` slug | `hc:codex:<model>` with that slug when it is one of `gpt-6-astra`, `gpt-6.1-sol`, `gpt-6-luna`, or `gpt-6.1-terra`. Otherwise ask. |
| `hc:<program>`, such as `hc:codex`, `hc:grok`, or `hc:pi` | Do not spawn a subagent. `hc run --bg --name <short-name> <program> "<brief>. When done run: hc send $HC_TERMINAL_ID <result>"`, then `hc wait` and `hc read`. Verify the result yourself. `hc:pi` only gets a task that names the exact files, the exact change, and a concrete verification command. |
| `hc:<program>:<model>`, such as `hc:codex:gpt-6.1-sol` | Same dispatch. For `codex`, pass the model with the verified `-m` flag: `hc run --bg --name <short-name> codex -m gpt-6.1-sol "<brief>. When done run: hc send $HC_TERMINAL_ID <result>"`. `codex exec -m gpt-6.1-sol "<task>"` is the verified non-interactive form. For `claude`, `grok`, or `pi`, do not invent a model flag. Run the program with no model argument and say so. |
| `auto` or `inherit-parent` | Do the role in this session. Do not open another window. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | The `pstack-models` block in the global Codex instructions file. See setup-pstack below. |
| A role value `<model> <effort>`, such as `opus high` | No verified map from a Cursor effort word onto a Codex config string, except the spellings in Reasoning effort. |

`gpt-6-astra` is the most capable Codex model. `gpt-6.1-sol` is near-Astra and cheaper. `gpt-6-luna` is the efficient model for focused, high-volume work. `gpt-6.1-terra` is for writing, visual understanding, and computer use. Do not use Terra for a code role. The CLI needs 0.159.0 or later for Terra (`codex -m gpt-6.1-terra`).

`hc:codex:gpt-6-luna` is only for a task that names the exact files, the exact change, and a concrete verification command. Otherwise use `hc:codex:gpt-6.1-sol` and say so. The parent still runs the verification.

The writer never reviews its own code. On the four-agent table, code from `hc:codex` is reviewed by Claude and/or Grok, and code from Claude is reviewed by Codex. On the Codex-only table, a `gpt-6.1-sol` writer is reviewed by `gpt-6-astra`, and a `gpt-6-astra` writer by `gpt-6.1-sol`. A one-vendor panel agrees more easily than a cross-vendor panel.

## Reasoning effort

Verified from Developer settings: personal defaults live in `~/.codex/config.toml`, a trusted project can add `.codex/config.toml`, and one run can override a key with `-c` or `--config`. The documented example is `codex --config model_reasoning_effort='"medium"'`. `/model` in the CLI changes `model` and `model_reasoning_effort`.

Verified from the models page: Astra's Light setting is `low` in configuration. The CLI calls that tier Low. Medium is the documented `--config` example. The same page names High, Extra High, Max, and Ultra as picker tiers. Their `model_reasoning_effort` strings were not in a config example. Do not guess them. Do not translate a pstack budget of `xhigh` or `max` into a Codex flag.

Pass `-c model_reasoning_effort='"<effort>"'` only when `<effort>` is `low` or `medium`. Put it on the `codex` argv, before the task string.

## setup-pstack

Follow `setup-pstack` with these changes:

- Detect. Run `command -v` in Git Bash for `hc`, `herdr`, and each command in the agent CLI list in `setup-pstack`. Do not copy that list. `herdr` is not required. List it only when it resolves. `hc:<program>` and `hc:<program>:<model>` are valid only when `hc` and that command were found. On Windows, a command found only as a `.ps1` shim may need `hc run` to launch the `.cmd` or `.exe` shim. That shim behavior was not verified.
- Defaults. If `hc` is missing, keep the no-`hc` role lines. If `hc`, `claude`, `codex`, `grok`, and `pi` are all found, keep the four-agent table with no `:<model>` suffix. If `hc` and `codex` are found and `claude`, `grok`, and `pi` are all missing, use the Codex-only table. Any other mix uses the substitutions in `setup-pstack`. Say which table and which fallback you used.
- Write. Codex home is `$CODEX_HOME` when set, otherwise `~/.codex`. If `AGENTS.override.md` there exists and is non-empty, edit it. Otherwise edit `AGENTS.md`. Replace only the block between `<!-- pstack-models:start -->` and `<!-- pstack-models:end -->`. Do not wipe the rest of the file. Do not write the role lines into `~/.codex/skills`.

## Paths

| Cursor | Codex |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | Repo skills: `.agents/skills`, scanned from the working directory up to the repo root. User skills: `$HOME/.agents/skills`. Admin skills: `/etc/codex/skills`. Each skill is a directory whose `SKILL.md` has `name` and `description`. Codex reads that metadata first and loads `SKILL.md` when it selects the skill. |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | Codex CLI opens a plugins browser with its plugins command and installs from a configured marketplace. The IDE extension does not support plugins. Installing this generated tree as a local plugin was not verified. Copy or symlink `harness/pstack/codex/skills` into `$HOME/.agents/skills` or the repo `.agents/skills`. |
| `.cursor/rules/*.mdc`, always-applied rules | Global instructions in Codex home (`~/.codex` unless `CODEX_HOME` is set): `AGENTS.override.md` if that file exists and is non-empty, otherwise `AGENTS.md`. Project instructions are `AGENTS.override.md`, then `AGENTS.md`, then names in `project_doc_fallback_filenames`, from the repo root down to the working directory. One file per directory. Closer directories are later in the prompt. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | `<pstack>/<path>` when that file exists. Read the local copy. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | No Codex transcript path was verified. Do not invent one. |

## Cursor-only features

Codex has no Cursor Automations, Bugbot, or Cursor cloud agents. When a step needs one, put one line in your reply, `Skipped in Codex: <step>. <reason>.`, then go on. Do not stop, ask, or retry.

## Unverified

- Whether `hc run` on Windows must be pointed at a `.cmd` or `.exe` shim when `command -v` finds a `.ps1`.
- Whether Codex still loads skills from `~/.codex/skills`. The verified user path is `$HOME/.agents/skills`.
- The `model_reasoning_effort` spelling of High, Extra High, Max, and Ultra. Verified spellings are `low` and `medium`.
- Model flags for the `claude`, `grok`, and `pi` CLIs.
- A Codex path for session transcripts.
- Installing `harness/pstack/codex` itself as a marketplace plugin.
- A probe that proves the current process is Codex.
